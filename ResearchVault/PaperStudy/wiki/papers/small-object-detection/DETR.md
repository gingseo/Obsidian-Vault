---
title: "End-to-End Object Detection with Transformers"
authors: [Nicolas Carion, Francisco Massa, Gabriel Synnaeve, Nicolas Usunier, Alexander Kirillov, Sergey Zagoruyko]
year: 2020
venue: "ECCV"
jcr_quartile: Q1
task: [small-object-detection]
direction: [foundational, novel-approach]
tags: [paper, object-detection, transformer, set-prediction, bipartite-matching, end-to-end, panoptic-segmentation]
status: read
user_read: false
added: 2026-08-12
source: "raw/small-object-detection/2020_ECCV_DETR.pdf"
created: 2026-08-12
---

# 한 줄 요약
<mark style="background: #FFF3A3A6;">객체 탐지를 anchor·NMS 같은 수작업 구성요소 없이 직접적인 집합 예측(direct set prediction) 문제로 재정의해, CNN backbone + transformer encoder-decoder + 이분 매칭(bipartite matching) 기반 집합 손실만으로 end-to-end 학습되는 최초의 DETR 프레임워크.</mark>

원문 요약(Abstract/Introduction/Main Contribution/Conclusion 번역): [[DETR-source]]

# 문제 정의

### 기존 방법의 한계
- **간접적 집합 예측**:
  기존 탐지기는 대규모 proposal, anchor, window center에 대한 대리 회귀·분류 문제를 정의해 집합 예측을 간접적으로 처리한다 — 최종 성능이 후처리(NMS), anchor 집합 설계, GT-anchor 할당 휴리스틱에 크게 좌우된다.
- **End-to-end 학습의 미실현**:
  기계 번역·음성 인식 등에서는 end-to-end 구조적 예측이 큰 발전을 이끌었지만, 객체 탐지의 기존 시도들은 추가 사전 지식을 필요로 하거나 강력한 baseline 대비 경쟁력을 입증하지 못했다.
- **중복 예측 제거의 수작업 의존**:
  근접-중복 예측을 정리하려면 NMS 같은 후처리가 필수적인데, 이는 예측 요소 간 상호작용을 명시적으로 모델링하지 않는 임시방편적 해법이다.

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Anchor/proposal 기반 대리 문제 정의**
- Two-stage 탐지기(Faster R-CNN [37], Cascade R-CNN [5]): proposal에 대한 상대적 예측.
- Single-stage 탐지기(anchor 기반 [23], window center 기반 [53, 46]): anchor나 grid 중심에 대한 상대적 예측.
- 공통 한계: 최종 성능이 anchor 집합 설계와 GT 할당 휴리스틱의 정교함에 크게 의존한다는 것이 후속 연구[52]로 입증됨.

**갈래 2 — 집합 기반 손실을 쓰는 초기 시도**
- Erhan et al. [9], Liu et al.(SSD) [25], Redmon et al.(YOLO) [35]: 이분 매칭 손실을 사용하지만, 예측 간 관계를 convolution/fully-connected layer로만 모델링해 결국 수작업 NMS 후처리가 성능 향상에 여전히 기여.
- Learnable NMS [16, 4], relation network [17]: attention으로 예측 간 관계를 명시적으로 모델링해 후처리를 제거하지만, proposal box 좌표 같은 추가적인 수작업 context feature에 의존.

**갈래 3 — 순환(recurrent) 기반 end-to-end 집합 예측**
- Stewart et al. [43], Romera-Paredes & Torr [41], Ren & Zemel [36], Salvador et al. [42]: CNN 기반 encoder-decoder와 이분 매칭 손실로 박스 집합을 직접 생성 — 소규모 데이터셋에서만 검증되었고 최신 강력한 baseline과 비교되지 않음. RNN 기반 autoregressive 구조라 병렬 디코딩을 활용하지 못함.

**갭**: <mark style="background: #FFF3A3A6;">집합 기반 손실과 관계 모델링을 결합하려는 시도는 있었지만, 여전히 수작업 context feature에 의존하거나(갈래 2) autoregressive RNN 구조의 순차적 디코딩 비용에 묶여(갈래 3) 최신 강력한 baseline과 경쟁력을 입증하지 못했다. Transformer의 병렬 디코딩과 이분 매칭 손실을 결합해, 수작업 사전 지식 없이 강력한 baseline과 대등한 성능을 내는 접근은 없었다.</mark>

### 이 논문이 풀고자 하는 문제
1. Anchor 생성·NMS 같은 수작업 구성요소 없이 객체 탐지를 직접적인 집합 예측으로 푸는 것
2. 예측 간 전역적 관계(pairwise interaction)를 모델링해 중복 예측을 후처리 없이 억제하는 것
3. 병렬 디코딩으로 autoregressive 구조의 순차적 추론 비용을 피하는 것

# 제안 방법

<mark style="background: #FFF3A3A6;">Transformer의 self-attention이 시퀀스 내 모든 요소 쌍의 상호작용을 명시적으로 모델링한다는 성질을 활용해, 고정된 소수의 학습된 object query가 이미지 전체 컨텍스트와 서로의 관계를 병렬로 추론하도록 하고, 예측과 GT 사이의 이분 매칭에 기반한 집합 손실로 end-to-end 학습한다.</mark>

### ① 이분 매칭 기반 집합 예측 손실
- N(이미지당 전형적 객체 수보다 충분히 큰 고정값)개의 예측을 한 번의 디코더 패스로 추론하고, GT도 크기 N으로 "object 없음(∅)" 패딩.
- 헝가리안 알고리즘(Hungarian algorithm)으로 예측-GT 간 최소 비용 순열(permutation)을 찾아 일대일 매칭을 계산.
- 매칭 비용은 클래스 확률과 박스 유사도(L1 + generalized IoU)의 조합.

> [!example]- 구현 디테일
> ```
> σ̂ = argmin_σ Σ_i L_match(y_i, ŷ_σ(i))
> L_match = -1{c_i≠∅}·p̂_σ(i)(c_i) + 1{c_i≠∅}·L_box(b_i, b̂_σ(i))
> L_Hungarian = Σ_i [-log p̂_σ̂(i)(c_i) + 1{c_i≠∅}·L_box(b_i, b̂_σ̂(i))]
> L_box = λ_iou·L_iou + λ_L1·||b_i - b̂_σ(i)||_1   (λ_iou=2, λ_L1=5)
> ```
> ∅ 클래스의 log-확률 항은 클래스 불균형 보정을 위해 10배 다운웨이트. 박스 손실에 GIoU를 추가한 이유는 순수 L1이 박스 크기에 따라 상대 오차가 같아도 절대 스케일이 달라지는 문제를 갖기 때문(스케일 불변성 확보).

<mark style="background: #FFF9D6A6;">이분 매칭은 각 GT 요소가 정확히 하나의 예측과만 대응하도록 강제해 예측 순서에 대한 순열 불변성(permutation-invariance)을 보장한다 — 이는 "문제 정의"에서 지적한 근접-중복 예측 문제를 후처리(NMS) 없이 손실 함수 설계만으로 해결하는 핵심 장치다.</mark>

### ② Transformer Encoder-Decoder 아키텍처
- CNN backbone(ResNet)이 저해상도 activation map을 추출하면, 1×1 conv로 채널을 축소하고 공간 차원을 flatten해 시퀀스로 변환, 고정된 sine 위치 인코딩을 더해 encoder에 입력.
- Encoder는 표준 multi-head self-attention + FFN 스택 — 이미지 전역 컨텍스트로 인스턴스를 서로 분리하는 역할(Fig. 3에서 attention map이 이미 개별 인스턴스를 분리).
- Decoder는 N개의 학습된 위치 임베딩("object query")을 입력받아, self-attention과 encoder-decoder cross-attention을 거쳐 모든 객체를 병렬로(한 번에) 디코딩 — Vaswani et al.의 원조 transformer가 쓰는 autoregressive 순차 디코딩과 대조.
- 각 디코더 출력 임베딩은 공유 FFN을 거쳐 클래스와 박스 좌표를 독립적으로 예측.

> [!example]- 구현 디테일
> ```
> f ∈ R^{C×H×W}, C=2048, H,W = H0/32, W0/32   (backbone 출력)
> z0 = Conv1x1(f) ∈ R^{d×H×W} → flatten → d×HW 시퀀스
> ```
> Auxiliary decoding loss: 모든 디코더 레이어 뒤에 예측 FFN + Hungarian loss를 추가로 붙여 학습 — 각 클래스의 정확한 객체 수를 출력하는 데 도움. 예측 FFN은 레이어 간 파라미터 공유.

<mark style="background: #FFF9D6A6;">병렬 디코딩은 self-attention의 전역 계산 능력을 유지하면서도 autoregressive 구조의 순차적 추론 비용을 제거한다 — "문제 정의"의 세 번째 문제(순차적 디코딩 비용)를 해소하는 동시에, self-attention이 모든 예측 쌍의 상호작용을 명시적으로 모델링하므로 별도의 수작업 context feature 없이도 중복 억제가 가능해진다("문제 정의"의 첫 번째·두 번째 문제 해소).</mark>

# 실험 결과

### 핵심 결과
| 벤치마크 | 지표 | Before(Faster R-CNN-FPN+, 강화된 baseline) | After(DETR, 동일 파라미터 수) |
|---|---|---|---|
| COCO val | AP / APS / APL | 42.0 / 26.6 / 53.4 | 42.0 / 20.5 / 61.1 |
| COCO val (R101 backbone) | AP | 44.0 | 43.5 |

> [!note]- 세부 결과 및 Ablation
> #### Encoder 레이어 수 영향
> | Encoder 레이어 | AP | APS | APL |
> |---|---|---|---|
> | 0 | 36.7 | 16.8 | 54.2 |
> | 6(baseline) | 40.6 | 19.9 | 60.2 |
> | 12 | 41.6 | 19.8 | 61.9 |
> - Encoder 제거 시 AP 3.9 하락, 특히 대형 객체에서 6.0 하락 — 전역 scene reasoning이 인스턴스 분리에 중요함을 시사.
>
> #### Decoder 레이어 수 / NMS 필요성
> - 첫 번째~마지막 디코더 레이어 사이 AP/AP50이 +8.2/+9.5만큼 꾸준히 개선.
> - 첫 레이어 출력에는 NMS가 AP를 개선(레이어 간 통신 부재로 중복 예측 발생)하지만, 레이어가 깊어질수록 NMS 효과가 감소하고 마지막 레이어에서는 오히려 AP를 소폭 깎음(NMS가 true positive를 잘못 제거) — self-attention이 중복 억제를 대체함을 검증.
>
> #### Loss 구성 요소
> | class | L1 | GIoU | AP | APS | APM | APL |
> |---|---|---|---|---|---|---|
> | ✓ | ✓ | | 35.8 | 13.7 | 39.8 | 57.9 |
> | ✓ | | ✓ | 39.9 | 19.9 | 43.2 | 57.9 |
> | ✓ | ✓ | ✓(baseline) | 40.6 | 19.9 | 44.3 | 60.2 |
> - GIoU 단독으로도 성능 대부분을 설명(baseline 대비 -0.7 AP), L1 단독은 성능이 크게 떨어짐.
>
> #### Panoptic Segmentation (COCO val)
> | 모델 | PQ | PQth | PQst |
> |---|---|---|---|
> | PanopticFPN++(R101) | 44.1 | 51.0 | 33.6 |
> | DETR-R101 | 45.1 | 50.5 | 37.0 |
> - Stuff 클래스에서 특히 우세(전역 attention의 이점으로 해석), things 클래스는 mask AP에서 최대 8점 열세에도 경쟁력 있는 PQth 달성.
>
> #### 일반화·분석
> - 학습 데이터에 13마리 이상의 기린이 있는 이미지가 없음에도, 합성 이미지에서 24마리를 문제없이 탐지 — object query가 클래스별로 특화되지 않음을 시사.
> - Decoder slot(query)별로 특정 영역·박스 크기에 특화되는 경향 관찰(Fig. 7).
> - 인스턴스 수가 학습 분포를 크게 벗어나면(50개 초과) 탐지 성능이 급격히 저하 — 100개 모두 존재 시 평균 30개만 탐지.

# Discussion

### 이 아이디어의 잠재적 부작용
- 매우 긴 학습 스케줄(300~500 epoch)과 특수한 학습 설정(Adam, dropout, random crop 등)이 필요 → <mark style="background: #FF5582A6;">저자들도 "표준 탐지기와 여러 면에서 학습 설정이 다르다"고 명시하며, Faster R-CNN처럼 SGD와 최소 증강만으로 학습되는 구조 대비 진입장벽이 있음을 인정.</mark>
- Encoder self-attention의 연산 비용이 O(d(HW)²)로 이미지 해상도에 이차적으로 증가 → <mark style="background: #FF5582A6;">DC5(dilated C5) 변형은 해상도를 2배 높이는 대신 encoder self-attention 비용이 16배, 전체 연산이 2배 증가한다고 논문이 직접 명시.</mark>

### 한계
- <mark style="background: #FF5582A6;">소형 객체(APS) 성능이 Faster R-CNN 대비 유의미하게 낮음(42.0 AP 동률 모델에서 APS -5.5~-6.1) — 저자들이 "FPN이 Faster R-CNN에 기여했던 것과 유사한 향후 개선이 필요하다"고 명시적으로 인정한 첫 한계.</mark>
- <mark style="background: #FF5582A6;">100개를 초과하는 동일 클래스 인스턴스가 밀집한 out-of-distribution 이미지에서 탐지 성능이 급격히 저하(Fig. 12) — 고정된 query 슬롯 수(N=100)라는 설계 자체의 한계.</mark>
- <mark style="background: #FF5582A6;">겹치는 객체가 많은 장면에서 panoptic mask 예측이 부정확해지는 실패 사례가 보고됨(Fig. 11a, 비행기 인스턴스 누락/부정확한 분할).</mark>

### 생각할 점
- <mark style="background: #A6E3A1A6;">이 위키의 [[UAV-DETR]]이 baseline으로 삼는 RT-DETR([[UAV-DETR]] 노트의 갈래 1)은 이 DETR의 NMS-free·anchor-free 철학을 그대로 계승하면서 실시간성을 더한 후속 계열 — DETR이 처음 지적한 "소형 객체 성능 열세"라는 한계가 UAV-DETR에서 주파수 도메인 모듈로 다시 다뤄지고 있다는 점에서 문제의식이 6년 가까이 이어지고 있다.</mark>
- <mark style="background: #A6E3A1A6;">DETR이 명시한 APS 한계는 이 위키의 small-object-detection 논문 대부분이 공유하는 출발점이다 — FPN이 Faster R-CNN의 소형 객체 성능을 끌어올렸듯, 이 위키의 feature 강화 계열 논문들([[sr-tod]], [[FANet]] 등)이 DETR 계열에도 유사하게 적용될 여지가 있는지는 아직 검증되지 않았다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">DETR의 이분 매칭이 갖는 "일대일 강제 할당"은 [[Unc-SOD]]나 [[cdatod-diff]]가 다루는 "동적/확률적 label assignment"와 근본적으로 다른 철학이다 — 결정론적 매칭 대신 불확실성 기반 매칭을 DETR류 구조에 도입하면 소형 객체의 애매한 GT 대응 문제를 완화할 여지가 있어 보인다.</mark>
- <mark style="background: #A6E3A1A6;">[[QueryDet]]의 "object query"라는 이름은 DETR의 object query와 유사하지만 메커니즘은 완전히 다르다(DETR: 학습된 슬롯이 전역 attention으로 객체를 찾음, QueryDet: 저해상도 예측이 고해상도 위치를 가리킴) — 두 "query" 개념의 차이를 명확히 구분해 둘 필요가 있다.</mark>

# 관련 개념
- (없음 — 이분 매칭 집합 손실, transformer encoder-decoder는 이 위키에서 독립 개념 문서로 만들기보다 DETR 자체의 정의로 충분히 설명되며, 아직 이를 재사용/변형하는 다른 논문이 이 위키에 없어 판단을 보류. UAV-DETR이 이 구조를 재사용하지만 RT-DETR을 경유하므로 직접적인 확장 관계는 아님)

# 관련 문서
- 비교: [[small-object-detection-approaches]]

# 읽어볼 만한 논문
- 참고문헌 기반: S. Ren, K. He, R. Girshick, J. Sun, "Faster R-CNN: Towards real-time object detection with region proposal networks" (PAMI 2015) [37] — 이 논문이 직접 비교하는 핵심 baseline. DETR의 모든 성능 비교가 이 구조를 기준으로 하므로 배경 이해에 필수.
- 참고문헌 기반: A. Vaswani, N. Shazeer, N. Parmar, J. Uszkoreit, L. Jones, A. N. Gomez, L. Kaiser, I. Polosukhin, "Attention is all you need" (NeurIPS 2017) [47] — DETR의 encoder-decoder 아키텍처가 직접 기반하는 원조 transformer 논문.
- 참고문헌 기반: A. Kirillov, K. He, R. Girshick, C. Rother, P. Dollár, "Panoptic segmentation" (CVPR 2019) [19] — DETR이 확장하는 panoptic segmentation 과제의 정의 논문. DETR의 통합 예측 방식이 이 과제를 어떻게 재정의하는지 이해하려면 필요.
- 자유 추천(검증 필요): DETR의 느린 수렴(느린 학습 스케줄) 문제를 해결한 후속 연구(Deformable DETR 등) — 이미 [[UAV-DETR]] 노트의 reading-list에 Deformable DETR이 등재되어 있으나, 그 원인이 된 DETR 자체의 수렴 문제를 다루는 분석 논문. 검색 키워드: `DETR slow convergence analysis sparse attention 2021`
