---
pm-task: true
projectId: "paperwiki-small-object-detection"
parentId:
id: "t-unc-sod-6pqpf7y57o"
title: "Unc-SOD: An Uncertainty Learning Framework for Small Object Detection"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-07-01"
due:
progress: 0
assignees: []
tags: []
customFields:
  "7l8l795xmtcjvlf6": 2026
  "1frf59rymtcjvske": "IEEE TIP"
subtaskIds: []
dependencies: []
year: 2026
venue: "IEEE TIP"
jcr_quartile: Q1
task: [small-object-detection]
direction: [improvement]
paper_tags: [paper, small-object-detection, uncertainty, two-stage-detector, rpn]
source: "Projects/_pdf/Small_Object_Detection/2026_TIP_Unc-SOD.pdf"
source_type: personal
createdAt: "2026-08-18T11:00:00.000Z"
updatedAt: "2026-08-31T00:00:00.000Z"
---

#paper #small-object-detection #uncertainty #two-stage-detector #rpn

> [!quote] 원제
> **Unc-SOD: An Uncertainty Learning Framework for Small Object Detection**
> Xiang Yuan, Gong Cheng, Jiacheng Cheng, Ruixiang Yao, Junwei Han — Northwestern Polytechnical University, IEEE TIP 2026
> https://doi.org/10.1109/TIP.2026.3654892

# 한 줄 요약
<mark style="background: #FFF3A3A6;">Two-stage small object detector의 RPN에 uncertainty branch를 추가해 인스턴스별 위치 불확실성을 동적 positive-sampling 기준으로 쓰고, 두 단계에서 쓰이는 pyramid feature 간 불일치(hierarchy-level uncertainty)를 Perception-and-Interaction 스킴으로 보정하는 프레임워크.</mark>

> [!info] 내 메모
> 

# 정리

## 기존 방법의 한계
- **Sample scarcity(sampling 규칙의 경직성)**:
  RPN은 고정 IoU 임계값(≥0.7)으로 positive/negative를 나누는데, 작은 객체는 이 기준을 만족하는 prior가 극히 적다. Fig. 1에서 IoU 0.8 이상 prior조차 학습이 진행될수록 오히려 target에서 멀어지는 역설이 관찰된다 — 데이터 자체의 구조 왜곡(부분 가림·모션 블러 등)에서 오는 data-level uncertainty가 근본 원인이다.
- **Hierarchy-level uncertainty(두 스테이지 간 feature 불일치)**:
  작은 객체는 최하위 pyramid 레벨(P2)에서만 RoI Align되도록 설계되어 있지만, 실제 Faster R-CNN의 proposal 95% 이상은 상위 레벨({P3, P4, P5})에서 나온다(Fig. 3) — 두 스테이지가 각각 다른 레벨의 feature를 쓰는 셈이라 대응이 어긋난다.

## 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Positive-sampling 기준 조정 및 uncertainty 모델링**
- 데이터 증강: Copy-and-paste(Kisantal et al.), DS-GAN[39] — 근본적 기준 변경 없음.
- Assignment 기준 조정: RFLA[28](Gaussian 수용영역 유사도), CFINet[16](coarse-to-fine) — 여전히 모든 인스턴스에 동일 기준.
- <mark style="background: #FFF3A3A6;">Box uncertainty 모델링: He et al.[23](KL divergence), Gaussian YOLOv3[24], GFL[25] — 극단적 변형/작은 인스턴스는 못 다룸, uncertainty를 NMS 후처리에만 사용.</mark>
- **타겟/해결**: <mark style="background: #FFF3A3A6;">Sample scarcity(①)를 다루지만, "instance-level uncertainty를 sampling 기준으로 쓰자"는 결합 시도는 없었다 — uncertainty를 모델링해도 후처리에만 쓰거나, sampling 기준은 여전히 모든 인스턴스에 동일하게 고정돼 있다.</mark>

**갈래 2 — Architectural redesign을 통한 hierarchy 대응**
- RFPN[30]류 architectural redesign이 pyramid를 재구성.
- SR-TOD[31]가 reconstruction 부산물을 활용.
- **타겟/해결**: Hierarchy-level uncertainty(②)를 겨냥한 일부 시도는 있으나, 두 스테이지의 feature 불일치 자체를 정면으로 다룬 연구는 거의 없는 미개척 문제로 남아있다.

**갭**: <mark style="background: #FFF3A3A6;">"어떤 prior를 positive로 볼지"(①)에는 uncertainty를 sampling 기준으로 명시적으로 결합한 연구가 없었고, "어느 pyramid 레벨의 feature를 쓸지"(②)는 거의 다뤄지지 않은 미개척 문제로 남아있다.</mark>

## 이 논문이 풀고자 하는 문제
1. 위치 불확실성을 정량화해 sampling에 반영 — proposal 품질/개수를 개선하는 것.
2. 두 스테이지 간 feature hierarchy 불일치를 해소해 판별력 있는 표현을 확보하는 것.

**갭 종합**: <mark style="background: #FFF3A3A6;">"어떤 prior를 positive로 볼지"와 "어느 pyramid 레벨의 feature를 쓸지"는 서로 다른 문제처럼 보이지만, 둘 다 작은 객체 특유의 제한된 정보량이 만드는 불확실성(uncertainty)을 기존 파이프라인이 명시적으로 다루지 않는다는 공통 원인에서 나온다. Unc-SOD는 이 불확실성을 인스턴스 레벨(sampling)과 계층 레벨(feature 융합) 두 곳 모두에서 명시적으로 모델링해 동시에 대응한다.</mark>

> [!info] 내 메모
> 

# 해결 방법 요약

| | 문제 ① — Sample scarcity(sampling 규칙의 경직성) | 문제 ② — Hierarchy-level uncertainty(두 스테이지 간 feature 불일치) |
|---|---|---|
| **해결 방법** | RPN에 위치 불확실성 예측 branch를 추가해, IoU 대신 instance-level uncertainty(`u_g`)를 동적 positive 기준으로 사용 | 원래 anchor 레벨(F_o)과 크기 기반 할당 레벨(F_a) feature를 상호 보완 융합하는 Perception-and-Interaction으로 최종 표현을 만든다 |
| **예상되는 문제점** | 애매한 인스턴스가 학습 기회를 박탈당할 위험 — 밀집·중첩 상황에서 완전히 해결되지 못함(Fig. 13 실패 사례) | Uncertainty 예측 오류가 전체 파이프라인의 sampling 품질을 좌우하는 구조적 리스크가 남음 |

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">RPN에 <span style="color:#c0392b; font-weight:bold;">위치 불확실성 예측 branch</span>를 추가해 예측된 표준편차로부터 instance-level uncertainty를 계산하고, 이를 고정 IoU 임계값 대신 동적 positive-sampling 기준으로 사용하며(Uncertainty-Aware Sampling), 두 pyramid level의 feature를 상호 보완 융합하는 <span style="color:#c0392b; font-weight:bold;">Perception-and-Interaction</span> 스킴으로 판별력 있는 최종 표현을 만든다.</mark>

## 전체 파이프라인 (Fig. 5 기준)

```
입력 이미지 (3, H, W)
       │
       ▼
Backbone + FPN                                   → 다중 스케일 feature P2~P5
       │
       ▼
① Uncertainty-Aware Sampling (RPN 확장)
   - Predefined priors → 3×3 conv → 1×1 conv → cls/reg/uncertainty(σ) 3개 head
   - u_g = 가중집계(prior별 정규화 uncertainty)                 → 인스턴스별 스칼라 u_g
   - Positive 기준: IoU(x_g, x_p) ≥ u_g  (기존: IoU ≥ 0.7 고정)
       │
       ▼
   Proposal Networks                              → RoI 후보
       │
       ▼
② Perception-and-Interaction (Detection Head)
   - F_o: 원래 anchor 레벨 RoI Align 출력             → (H, W, 256)
   - F_a: 크기 기반 할당 레벨 RoI Align 출력            → (H, W, 256)
   - Analytic Perception(F_a → kernel → F_o에 적용)   → F_ap
   - Holistic Perception(GAP(F_o) → F_a에 결합)       → F_hp
   - Cross Interaction(Q=F_hp, K/V=F_ap)              → F_ci
   - F_pi = φ_re(F_ci) + F_a                          → (H, W, 256)
       │
       ▼
Cls / Reg Head                                     → 최종 클래스·박스 예측
```

> [!info] 내 메모
> 

### ① Uncertainty-Aware Sampling

- **역할**:
  고정 IoU 임계값(≥0.7)은 쉬운 인스턴스와 애매한 인스턴스를 구분하지 않고 동일한 엄격한 기준을 적용한다. 이 branch는 각 positive prior의 위치 불확실성(표준편차)을 예측해 인스턴스별 동적 positive 기준을 만들어, 확신도 높은 인스턴스는 기준을 완화하고 애매한 인스턴스는 엄격하게 유지한다.
- **구현**:
  RPN에 3×3 conv + 1×1 conv로 구성된 auxiliary branch를 추가해, 예측 박스 각 변(top/left/bottom/right)의 표준편차 `σ_t, σ_l, σ_b, σ_r`을 출력한다(box 표현을 Gaussian 분포로 모델링, He et al.[23] 확장). 정규화된 uncertainty를 변의 길이로 가중합해 prior별 `u_norm`을 얻고, 한 인스턴스의 모든 positive prior에 대해 IoU 기반 비선형(sigmoid-like) 가중합으로 집계해 instance-level uncertainty `u_g`를 산출한다. 고정 임계값 대신 `IoU(x_g, x_p) ≥ u_g`를 positive 기준으로 사용 — 인스턴스마다 동적으로 바뀐다.
- **입출력 shape**:
  Predefined priors → uncertainty branch → 각 prior당 표준편차 4개 값 `(4,)` → 인스턴스당 스칼라 `u_g`.

```python
# 논문 Eq.(2)-(8) 기반 의사코드
L_unc = (x_g - x_p)**2 / (2 * sigma**2) - 0.5 * log(sigma**2)   # uncertainty branch 학습 loss

u_norm = (d_t/d)*sigma_t + (d_l/d)*sigma_l + (d_b/d)*sigma_b + (d_r/d)*sigma_r   # d = 각 변 거리 합
# Gamma(z) = 1 / (1 + exp(-2z))  : IoU 낮은 prior일수록 가중치 증가하는 비선형 변환
u_g = sum(Gamma(delta_i) * u_norm_i for i in positive_priors) / sum(Gamma(delta_i) for i in positive_priors)

# Positive 기준(동적)
is_positive = IoU(x_g, x_p) >= u_g          # 기존: IoU(x_g, x_p) >= 0.7 (고정)

L_RPN = L_cls + L_reg + alpha * L_unc        # alpha=1.0이 실험적 최적
```

<mark style="background: #FFF9D6A6;">고정 IoU 기준은 쉬운 인스턴스에도 똑같이 엄격했다. Instance-level uncertainty를 기준으로 쓰면 확신도 높은 인스턴스는 기준이 완화되고 애매한 인스턴스는 엄격해져, "모두에게 동일 기준"이라는 근본 원인이 사라진다 — Ablation(Table VII)에서 Uncertainty-aware Sampling 단독 추가만으로 SODA-D AP가 28.9%→29.9%로 상승한 것이 이를 뒷받침한다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> 애매한 인스턴스가 학습 기회를 박탈당할 위험이 완전히 해결되지 않았고, 밀집·중첩 상황에서는 여전히 실패 사례가 존재한다(Fig. 13). 또한 uncertainty branch의 예측 정확도에 sampling 품질 전체가 의존하는 구조적 리스크가 있다.

> [!info] 내 메모
> 

### ② Perception-and-Interaction

- **역할**:
  Faster R-CNN류는 proposal 크기에 따라 RoI Align할 pyramid 레벨을 결정하는데(size-dependent mapping), 소형 객체는 대부분 최하위 P2로 매핑되도록 설계돼 있음에도 실제 proposal의 95% 이상은 상위 레벨에서 나온다(Fig. 3). 이 모듈은 "제안 근거가 된 원래 레벨(F_o)"과 "판단에 쓰이는 할당된 레벨(F_a)"이라는 서로 다른 두 표현을 상호 보완적으로 융합해 이 hierarchy 불일치를 해소한다.
- **구현**:
  - **F_o**(원래 anchor 레벨): 구조 정보가 풍부하지만 왜곡·잡음에 취약.
  - **F_a**(크기 기반 할당 레벨): compact하고 노이즈에 강함.
  - 두 feature를 128채널로 압축(F̃_o, F̃_a)한 뒤, Analytic Perception(F̃_a로 position-dependent kernel `W_ap` 생성 → F̃_o에 convolution 적용해 구조를 복원)과 Holistic Perception(F̃_o를 GAP → F̃_a와 결합, 높은 압축 정보로 노이즈·변형에 대한 견고성 부여)을 각각 계산.
  - Cross Interaction(self-attention 형태, `F_hp`를 query, `F_ap`를 key/value로)으로 두 표현을 재교정한 뒤, `φ_re`로 채널을 256으로 복원하고 `F_a`와 addition해 최종 표현 `F_pi`를 얻는다.
- **입출력 shape**:
  `F_o, F_a ∈ (H, W, 256)` → 각각 128채널로 압축 → `F_pi ∈ (H, W, 256)`.

```python
# 논문 Eq.(10)-(15) 기반 의사코드
F_ap = W_ap conv F_o_tilde        # Analytic Perception: F_a_tilde로 생성한 position-dependent kernel을 F_o_tilde에 적용
W_ap = Norm(phi_ap(F_a_tilde))    # 1x1 conv로 커널 생성 후 정규화

F_hp = GAP(F_o_tilde) dot F_a_tilde   # Holistic Perception

F_ci = softmax(F_q @ F_k.T / sqrt(d)) @ F_v   # Cross Interaction
F_q, F_k, F_v = phi_q(F_hp), phi_k(F_ap), phi_v(F_ap)

F_pi = phi_re(F_ci) + F_a          # 최종 표현
```

<mark style="background: #FFF9D6A6;">기존엔 할당 레벨(F_a)만 최종 판단에 쓰여 제안 근거였던 F_o의 구조 정보가 버려졌다. 두 레벨을 상호 보완 융합하면 "제안 근거"와 "판단 근거"의 불일치가 해소된다 — Ablation(Table VII)에서 Perception-and-Interaction 단독 추가로 SODA-D AP가 28.9%→30.4%, 특히 도전적인 AP_eS·AP_rS에서 두드러진 개선을 보인다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> 밀집 가림 상황에서 NMS 이후에도 중복 예측이 남는 문제는 이 모듈이 해결하지 못한다 — feature 표현을 개선할 뿐 인스턴스 간 공간적 근접성 자체를 다루지 않기 때문이다.

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| ① Uncertainty-Aware Sampling | Predefined priors | 인스턴스당 스칼라 u_g | 동적 positive 기준으로 sample scarcity 완화 | RPN 확장(3×3+1×1 conv), Gaussian box 표현 + 비선형 IoU 가중합 |
| ② Perception-and-Interaction | F_o, F_a ∈ (H,W,256) | F_pi ∈ (H,W,256) | 두 스테이지 간 feature hierarchy 불일치 해소 | Analytic/Holistic Perception + Cross Interaction(self-attention 형태) |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table I (SODA-D), Table II (SODA-A)
**표를 보는 법**: Baseline은 두 표 모두 Faster RCNN(SODA-A는 Rotated Faster RCNN) 계열이다 — AP, AP_eS(극소형)까지 함께 보면 소형 객체 특화 개선 여부를 알 수 있다.

| 벤치마크 | 지표 | Before(Baseline) | After(Unc-SOD) |
|---|---|---|---|
| SODA-D | AP | 28.9% | 31.0% |
| SODA-A | AP | 32.5% | 34.8% (SOTA) |

> [!note]- 세부 결과 및 Ablation
> #### 설정
> - **데이터셋**: SODA-D(교통 시나리오, 9클래스, 278,433 인스턴스), SODA-A(위성 이미지, 9클래스, 872,069 인스턴스, oriented box), COCO, TT100K(교통 표지판), VisDrone — 800×800 패치로 crop 후 SODA-D/A는 1200×1200으로 리사이즈, TT100K는 1024×1024, COCO/VisDrone은 표준 설정
> - **지표**: AP는 IoU 0.5~0.95(10개 threshold) 평균, AP_eS(0,144)/AP_rS(144,400)/AP_gS(400,1024)/AP_N(1024,2000) — SODA 데이터셋 전용 크기 구간
> - ResNet-50+FPN, SGD(momentum 0.9, weight decay 0.0001), 12 epoch(1x), lr 0.01(8/11 epoch decay), 2×는 24 epoch. RTX 3090 ×2
>
> #### SODA-D 상세 (Table I, 발췌)
> | Method | Schedule | AP | AP50 | AP75 | AP_eS | AP_rS | AP_gS | AP_N |
> |---|---|---|---|---|---|---|---|---|
> | Baseline(Faster RCNN) | 1× | 28.9 | 59.4 | 24.1 | 13.8 | 25.7 | 34.5 | 43.0 |
> | RFLA[28] | 1× | 29.7 | 60.2 | 25.2 | 13.2 | 26.9 | 44.6 | 44.6 |
> | CFINet[16] | 1× | 30.7 | 60.8 | 26.7 | 14.7 | 26.4 | 43.6 | 44.6 |
> | SR-TOD[31] | 1× | 29.3 | 60.0 | 24.5 | 13.8 | 26.0 | 43.4 | 44.7 |
> | BAFNet[71] | 1× | 30.1 | **61.0** | 25.5 | 14.6 | 26.9 | 44.7 | 51.0 |
> | Unc-SOD(ours) | 1× | 31.0 | 60.27 | 27.1 | 14.9 | 27.6 | 36.9 | 45.8 |
> | Unc-SOD(ours) | 2× | **32.5** | 62.5 | 28.7 | 16.4 | 28.6 | 36.6 | 49.7 |
>
> 1× 스케줄만으로 대부분 SOD-특화 경쟁자(RFLA, CFINet, SR-TOD, BAFNet)를 상회 — 추가 데이터 증강이나 긴 학습 없이 달성한 결과라는 점을 저자가 강조.
>
> #### SODA-A 상세 (Table II, 발췌)
> Oriented RCNN(AP 34.4%, AP75 28.6%로 고품질 localization은 최고)·CFINet(AP 34.4%, AP_eS 13.5%로 극소형 특화)이 강한 경쟁자이나, Unc-SOD가 AP 34.8%로 최고 — 특히 AP_eS 13.8%로 극소형 객체에서도 균형 잡힌 성능을 보인다.
>
> #### 추가 벤치마크
> | 벤치마크 | 지표 | Before | After | 비고 |
> |---|---|---|---|---|
> | COCO val | APS | 21.0% | 23.3% | EFPN[18]·CFINet[16] 상회, generic detector 대비도 우위 |
> | TT100K | APS | 39.7% | 41.9% | 2위 BAFNet[71](40.9%) 대비 +1.0%p |
> | VisDrone val | APS | 17.4% | 20.0% | HawkNet[83](19.9%)·ClusDet[82](17.6%) 상회, 두 경쟁자보다 짧은 스케줄(15 epoch) |
>
> #### Ablation — 컴포넌트별 기여 (Table VI, SODA-D/COCO)
> | 구성 | SODA-D AP | SODA-D AP_eS | COCO AP | COCO APS |
> |---|---|---|---|---|
> | Baseline | 28.9 | 13.8 | 37.2 | 21.0 |
> | + Uncertainty-aware Sampling(US) | 29.9 | 14.3 | 38.2 | 22.4 |
> | + Perception-and-Interaction(PI) | 30.4 | 14.6 | 38.2 | 22.7 |
> | + US + PI | **31.0** | **14.9** | **38.6** | **23.3** |
>
> COCO에서도 같은 경향(21.0%→22.4%→22.7%→23.3%) — 데이터셋 특정 현상이 아님.
>
> #### Ablation — Uncertainty branch/sampling 세부 (Table VII)
> Baseline(28.9 AP) → uncertainty branch만 추가(sampling 미반영, 29.2 AP, 미미) → sampling까지 반영(29.9 AP) — uncertainty를 예측하는 것 자체보다 "sampling 기준으로 실제로 쓰는가"가 핵심.
>
> #### 하이퍼파라미터 탐색
> - T_pos(Eq.3의 positive IoU 임계값, Table VIII): 0.50→29.4, **0.60→29.9(채택)**, 0.70→29.7, 0.80→29.5 — 너무 낮으면 저품질 prior가 섞이고 너무 높으면 샘플이 부족해 sweet spot 형태.
> - α(uncertainty loss 가중치, Table IX): 0.1→30.6, 0.5→30.8, **1.0→31.0(채택)**, 2.0→30.5.
> - Γ(·) 변환(Table X): Uniform weighting 29.6 < IoU weighting(선형) 29.4(오히려 vanilla보다 낮음, 고-IoU prior 과대평가) < **Sigmoid-like Mapping(Eq.6, 채택) 29.9**.
> - Analytic/Holistic Perception 요소별(Table XI): Interaction만 29.9→30.3, Analytic+Interaction 30.6, Holistic+Interaction 30.8, 전부 결합 **31.0**(addition term 제거 시 30.8로 −0.2%p).
> - Interaction 입력 조합(Table XII): Query=F_hp, Key/Value=F_ap가 최적(31.0) — 다른 조합(F_ap를 query로 등)은 29.8~30.5로 열세.
> - F_o의 소스 레벨(Table XIV): Original(현재 설계, 31.0)이 P2~P5 고정 레벨(30.5~30.8)보다 우수 — P2 고정 시에도 0.2%p 차이로 근소, P2가 이미 소형 객체 정보를 풍부히 담고 있기 때문.
> - Perception-and-Interaction 대안 비교(Table XIII): Addition/Concatenation with Convs(30.1~30.3) < Cross-attention(30.4) < **Perception-and-Interaction(31.0)** — 단순 융합·attention만으로는 부족.
> - K(Analytic Perception 커널 크기, Table XV): K=1→30.6, **K=3→31.0(채택)**, K=5→30.9.
>
> #### 일반화 검증 — 다른 detector 이식 (Table XVI, SODA-D)
> | Detector | AP | + Unc-SOD |
> |---|---|---|
> | Cascade RCNN[11] | 31.2 | 32.6 (+1.4) |
> | DetectoRS[88] | 31.3 | 32.8 (+1.5) |
>
> Cascade R-CNN·DetectoRS 두 강한 baseline에도 일관된 개선 — 특정 detector에 국한되지 않음.
>
> #### 정성 결과
> - Fig. 11(SODA-D): baseline 대비 극단적 크기 인스턴스 탐지 개선 + false positive 억제를 동시에 보여줌.
> - Fig. 12(TT100K): confidence 0.3 이상 예측만 시각화, 소형 표지판까지 정확히 탐지.
> - Fig. 13(실패 사례): 밀집 가림 상황에서 GT 박스 간 근접성 때문에 redundant prediction이 NMS 이후에도 잔존.

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- 애매한 인스턴스가 학습 기회를 박탈당할 위험 → <mark style="background: #FF5582A6;">완전히 해결 못함, 밀집·중첩 상황 실패 사례 존재(Fig. 13).</mark>
- Uncertainty 예측 오류가 좋은 후보를 걸러낼 위험 → 비선형 가중으로 부분 완화했으나 <mark style="background: #FF5582A6;">uncertainty branch 정확도에 전체가 의존하는 구조적 리스크는 남음.</mark>

### 한계
- <mark style="background: #FF5582A6;">Aleatoric uncertainty만 다룸, epistemic uncertainty는 미다룸</mark> — 저자가 Conclusion에서 향후 과제로 명시(ensemble/MC dropout 기반 epistemic uncertainty를 결합해 overconfident false positive를 억제하는 방향).
- <mark style="background: #FF5582A6;">밀집 가림 상황에서 NMS 이후에도 중복 예측 남음(Fig. 13) — GT 박스 간 근접성이 근본 원인.</mark>
- 소수 하이퍼파라미터(T_pos=0.60, α=1.0, K=3)가 이 논문의 5개 벤치마크에 맞춰진 값 — 다른 도메인 재검증 필요.

### 생각할 점
- <mark style="background: #A6E3A1A6;">Instance-level uncertainty는 SOD를 넘어 세그멘테이션 경계, 이상 탐지 정상/비정상 경계 등 "라벨 자체가 애매한" 다른 태스크에도 이식 가능해 보임.</mark>
- Hierarchy-level uncertainty를 사후 융합이 아니라 애초에 일관된 pyramid level을 쓰도록 구조를 바꾸는 대안도 가능(연산 비용 트레이드오프 예상) — 실제로 Ablation(Table XIV)에서 P2 고정 레벨도 근소한 차이(−0.2%p)로 원본 설계에 근접했다는 점이 이 대안의 여지를 뒷받침한다.

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">[[Gaussian_Box_Uncertainty_Modeling]]의 instance-level uncertainty를 [[Self_Reconstruction_Difference_Map]]·[[Frequency_Domain_Feature_Enhancement]] 같은 feature 강화 계열과 결합 가능 — 불확실성 큰 영역에 feature 강화를 더 강하게 적용.</mark> 두 축은 [[Small_Object_Detection_Approaches]]에서 직교적 개선으로 분류됨.
- Epistemic uncertainty 결합은 ensemble/MC dropout이 흔한 방식 — 비용 대비 이득 검증 필요.

> [!info] 내 메모
> 

# 관련 개념
- [[Gaussian_Box_Uncertainty_Modeling]] — 박스 좌표를 Gaussian 분포로 모델링. He et al. [23]의 KL loss 방식을 확장.
- [[Perception_And_Interaction]] — 두 pyramid level의 feature를 상호 보완 융합하는 핵심 모듈.

# 관련 문서
- 비교 후보: [[2024_ECCV_SR-TOD|SR-TOD]] (동일 저자 그룹이 인용하는 SODA-D/SODA-A 비교 대상). RFLA[28] #pending:rfla, CFINet[16] #pending:cfinet 도 비교군으로 언급되지만 아직 위키에 노트 없음.
- 같은 저자 그룹: [[2026_TPAMI_Detection_Oriented_Rectification|Detection_Oriented_Rectification]] — feature 열화/복원 관점의 다른 각도 연구.
- 비교: [[Small_Object_Detection_Approaches]] — label assignment/sampling 축으로 분류.

# 읽어볼 만한 논문
- 참고문헌 기반: C. Xu et al., "RFLA: Gaussian receptive field based label assignment for tiny object detection" [28] (ECCV 2022) — Unc-SOD의 sampling 전략과 직접 비교되는 baseline.
- 참고문헌 기반: A. Kendall and Y. Gal, "What uncertainties do we need in Bayesian deep learning for computer vision?" [47] (NeurIPS 2017) — epistemic uncertainty 개념의 원조. 저자가 Conclusion에서 향후 과제로 지목한 aleatoric+epistemic 통합 프레임워크를 이해하려면 필요.
- 참고문헌 기반: J. U. Kim et al., "CUA loss: Class uncertainty-aware gradient modulation for robust object detection" [49] (IEEE TCSVT 2021) — epistemic uncertainty를 손실 함수에 반영한 사례.
- 자유 추천(검증 필요): MC Dropout 기반 epistemic uncertainty 근사 연구 — 검색 키워드: `"Monte Carlo dropout" object detection epistemic uncertainty`. 저자가 명시한 향후 과제(aleatoric+epistemic 통합)와 직결.
