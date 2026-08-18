---
title: "Unc-SOD: An Uncertainty Learning Framework for Small Object Detection"
authors: [Xiang Yuan, Gong Cheng, Jiacheng Cheng, Ruixiang Yao, Junwei Han]
year: 2026
venue: "IEEE TIP"
jcr_quartile: null
task: [small-object-detection]
direction: [improvement]
tags: [paper, small-object-detection, uncertainty, two-stage-detector, rpn]
status: in-progress
added: 2026-07-01
source: "PaperStudy/Raw/Small_Object_Detection/2026_TIP_Unc-SOD.pdf"
created: 2026-08-04
---

#paper #small-object-detection #uncertainty #two-stage-detector #rpn

# 한 줄 요약
<mark style="background: #FFF3A3A6;">Two-stage small object detector의 RPN에 uncertainty branch를 추가해 sample scarcity 문제를 동적으로 완화하고, 두 단계에서 쓰이는 feature 간 불일치(hierarchy-level uncertainty)를 Perception-and-Interaction으로 보정하는 프레임워크.</mark>


# 문제 정의

### 기존 방법의 한계
- **Sample scarcity**:
  고정 IoU 임계값(≥0.7)으로 positive/negative를 나눠, 작은 객체는 기준을 만족하는 prior가 극히 적음. IoU 0.8 이상 prior조차 학습이 진행될수록 오히려 타겟에서 멀어지는 역설 관찰(Fig. 1).
- **Data-level uncertainty**:
  부분 가림·모션 블러로 구조가 왜곡되어 GT 라벨링 자체가 모호함.
- **Hierarchy-level uncertainty**:
  작은 객체는 최하위 피라미드(P2)에서만 RoI Align하지만, 실제 proposal의 95% 이상은 상위 레벨(P3~P5)에서 나옴 — 두 스테이지가 쓰는 feature가 불일치.

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Sample scarcity 대응**
- 데이터 증강: Copy-and-paste(Kisantal et al.), DS-GAN [39] — 근본적 기준 변경 없음.
- Assignment 기준 조정: RFLA [28](Gaussian 수용영역 유사도), CFINet [16](coarse-to-fine) — 여전히 모든 인스턴스에 동일 기준.

**갈래 2 — Box uncertainty 모델링**
- Gaussian/일반 분포로 박스 표현: He et al. [23](KL divergence), Gaussian YOLOv3 [24], GFL [25] — 극단적 변형/작은 인스턴스는 못 다룸, uncertainty를 NMS 후처리에만 사용.

**갭**: <mark style="background: #FFF3A3A6;">두 갈래가 독립 발전해왔고, "instance-level uncertainty를 sampling 기준으로 쓰자"는 결합 시도가 없었다.</mark> hierarchy-level uncertainty도 거의 다뤄지지 않은 미개척 문제.

### 이 논문이 풀고자 하는 문제
1. 위치 불확실성을 정량화해 sampling에 반영 → proposal 품질/개수 개선
2. 두 스테이지 간 feature hierarchy 불일치 해소 → 판별력 있는 표현 확보

# 제안 방법

<mark style="background: #FFF3A3A6;">RPN에 위치 불확실성 예측 branch를 추가해 동적 positive-sampling 기준으로 쓰고, 원래 pyramid feature와 할당 pyramid feature를 상호 보완적으로 융합한다.</mark>

### ① Uncertainty-Aware Sampling
- 박스 각 변을 Gaussian 분포로 모델링(He et al. [23] 확장)해 표준편차 σ 예측.
- Positive prior들의 uncertainty를 IoU 기반 비선형 가중합으로 집계해 instance-level uncertainty `u_g` 산출.
- 고정 임계값 대신 `IoU ≥ u_g`를 positive 기준으로 사용 — 인스턴스마다 동적으로 바뀜.
- IoU 낮은 prior일수록 가중치 ↑ ("겹침 적은데 확신도 양호 = 애매한 인스턴스라는 신호").

> [!example]- 구현 디테일
> ```
> L_unc = (x_g - x_p)² / 2σ² − 0.5·log σ²
> ```
> Positive prior 집계는 Eq. 5~6, sigmoid-like Γ 함수 사용.
>
> ```
> L_RPN = L_cls + L_reg + α·L_unc     (α=1.0이 실험적 최적)
> ```
> 튜닝 대상은 `T_pos=0.60`, kernel size `K=3` 정도로 거의 없음.

<mark style="background: #FFF9D6A6;">고정 IoU 기준은 쉬운 인스턴스에도 똑같이 엄격했다. Instance-level uncertainty를 기준으로 쓰면 확신도 높은 인스턴스는 기준이 완화되고 애매한 인스턴스는 엄격해져, "모두에게 동일 기준"이라는 근본 원인이 사라진다.</mark>

### ② Perception-and-Interaction
- **F_o**(원래 anchor 레벨): 구조 정보 풍부, 잡음에 취약.
- **F_a**(크기 기반 할당 레벨): compact, 노이즈에 강함.
- 처리 순서: Analytic Perception(F_a로 kernel 생성 → F_o 구조 복원) → Holistic Perception(F_o GAP → F_a와 결합) → Cross Interaction(F_hp를 query, F_ap를 key/value) → 최종 표현 F_pi.

<mark style="background: #FFF9D6A6;">기존엔 할당 레벨(F_a)만 최종 판단에 쓰여 제안 근거였던 F_o 정보가 버려졌다. 두 레벨을 상호 보완 융합하면 "제안 근거"와 "판단 근거"의 불일치가 해소된다.</mark>

# 실험 결과

### 핵심 결과

| 벤치마크 | 지표 | Before | After |
|---|---|---|---|
| SODA-D | AP | 28.9% | 31.0% |
| SODA-A | AP | 32.5% | 34.8% (SOTA) |

> [!note]- 세부 결과 및 Ablation
> #### 추가 벤치마크
> | 벤치마크 | 지표 | Before | After | 비고 |
> |---|---|---|---|---|
> | SODA-D | APeS(극소형) | 13.8% | 14.9% | |
> | COCO val | APS | 21.0% | 23.3% | EFPN [18]·CFINet [16] 상회 |
> | TT100K | APS | 39.7% | 41.9% | 2위 BAFNet [71] 대비 +1.0%p |
> | VisDrone | APS | 17.4% | 20.0% | HawkNet [83]·ClusDet [82] 상회 |
>
> #### Ablation (TABLE VI, SODA-D)
> | 구성 | AP | APeS |
> |---|---|---|
> | Baseline | 28.9% | 13.8% |
> | + Uncertainty-aware Sampling | 29.9% | 14.3% |
> | + Perception-and-Interaction | 30.4% | 14.6% |
> | + 둘 다 | 31.0% | 14.9% |
>
> COCO에서도 같은 경향 재현(21.0%→22.4%→22.7%→23.3%) — 데이터셋 특정 현상 아님.
>
> #### 세부 발견
> - Uncertainty branch 단독(sampling 미반영)은 개선 미미 — sampling에 쓰는 방식 자체가 핵심.
> - Γ 함수: 선형 가중은 비효과적(29.4%), sigmoid 비선형이 최적(29.9%).
> - T_pos=0.60, α=1.0이 최적 지점.
> - Query=F_hp, Key/Value=F_ap 조합이 최고 성능.
> - Cascade R-CNN [11], DetectoRS [88]에 이식해도 +1.4%p, +1.5%p로 일반화 확인.

# Discussion

### 이 아이디어의 잠재적 부작용
- 애매한 인스턴스가 학습 기회를 박탈당할 위험 → <mark style="background: #FF5582A6;">완전히 해결 못함, 밀집·중첩 상황 실패 사례 존재(Fig. 13).</mark>
- Uncertainty 예측 오류가 좋은 후보를 걸러낼 위험 → 비선형 가중으로 부분 완화했으나 <mark style="background: #FF5582A6;">uncertainty branch 정확도에 전체가 의존하는 구조적 리스크는 남음.</mark>

### 한계
- <mark style="background: #FF5582A6;">Aleatoric uncertainty만 다룸, epistemic uncertainty는 미다룸</mark> — 저자가 향후 과제로 명시.
- <mark style="background: #FF5582A6;">밀집 가림 상황에서 NMS 이후에도 중복 예측 남음.</mark>
- 소수 하이퍼파라미터가 이 논문의 5개 벤치마크에 맞춰진 값 — 다른 도메인 재검증 필요.

### 생각할 점
- <mark style="background: #A6E3A1A6;">Instance-level uncertainty는 SOD를 넘어 세그멘테이션 경계, 이상 탐지 정상/비정상 경계 등 "라벨 자체가 애매한" 다른 태스크에도 이식 가능해 보임.</mark>
- Hierarchy-level uncertainty를 사후 융합이 아니라 애초에 일관된 pyramid level을 쓰도록 구조를 바꾸는 대안도 가능(연산 비용 트레이드오프 예상).

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">[[Gaussian_Box_Uncertainty_Modeling]]의 instance-level uncertainty를 [[Self_Reconstruction_Difference_Map]]·[[Frequency_Domain_Feature_Enhancement]] 같은 feature 강화 계열과 결합 가능 — 불확실성 큰 영역에 feature 강화를 더 강하게 적용.</mark> 두 축은 [[Small_Object_Detection_Approaches]]에서 직교적 개선으로 분류됨.
- Epistemic uncertainty 결합은 ensemble/MC dropout이 흔한 방식 — 비용 대비 이득 검증 필요.

# 관련 개념
- [[Gaussian_Box_Uncertainty_Modeling]] — 박스 좌표를 Gaussian 분포로 모델링. He et al. [23]의 KL loss 방식을 확장.
- [[Perception_And_Interaction]] — 두 pyramid level의 feature를 상호 보완 융합하는 핵심 모듈.

# 관련 문서
- 비교 후보: [[SR-TOD]] (동일 저자 그룹이 인용하는 SODA-D/SODA-A 비교 대상). RFLA [28] #pending:rfla, CFINet [16] #pending:cfinet 도 비교군으로 언급되지만 아직 위키에 노트 없음.
- 같은 저자 그룹: [[Detection_Oriented_Rectification]] — feature 열화/복원 관점의 다른 각도 연구.
- 비교: [[Small_Object_Detection_Approaches]] — label assignment/sampling 축으로 분류.

# 읽어볼 만한 논문
- 참고문헌 기반: C. Xu et al., "RFLA: Gaussian receptive field based label assignment for tiny object detection" [28] (ECCV 2022) — Unc-SOD의 sampling 전략과 직접 비교되는 baseline.
- 참고문헌 기반: A. Kendall and Y. Gal, "What uncertainties do we need in Bayesian deep learning for computer vision" [47] (NeurIPS 2017) — epistemic uncertainty 개념의 원조.
- 참고문헌 기반: J. U. Kim et al., "CUA loss" [49] (TCSVT 2021) — epistemic uncertainty를 손실 함수에 반영한 사례.
- 자유 추천(검증 필요): MC Dropout 기반 epistemic uncertainty 근사 연구 — 검색 키워드: `"Monte Carlo dropout" object detection epistemic uncertainty`
