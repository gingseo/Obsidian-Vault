---
pm-task: true
projectId: "paperwiki-object-detection"
parentId:
id: "t-cross-dino-4037l83x0k"
title: "Cross-DINO: Cross the Deep MLP and Transformer for Small Object Detection"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-09-15"
due:
progress: 0
assignees: []
tags: []
customFields:
  "nh3oelhxmtcnb377": 2025
  "gx1mmrf0mtcnb37a": "TMM"
subtaskIds: []
dependencies: []
year: 2025
venue: "IEEE Transactions on Multimedia (TMM)"
jcr_quartile: Q1
task: [object-detection]
direction: [improvement]
paper_tags: [paper, small-object-detection, detr, dino, mlp, feature-blurring, soft-label]
source: "Projects/_pdf/Object_Detection/DETR/작은객체/2025_TMM_Cross-DINO.pdf"
source_type: personal
createdAt: "2026-09-15T12:05:33.000Z"
updatedAt: "2026-09-15T12:05:33.000Z"
---

#paper #small-object-detection #detr #dino #mlp #feature-blurring #soft-label

> [!quote] 원제
> **Cross-DINO: Cross the Deep MLP and Transformer for Small Object Detection**
> Guiping Cao, Wenjian Huang, Xiangyuan Lan, Jianguo Zhang, Dongmei Jiang, Yaowei Wang — Southern University of Science and Technology / Pengcheng Laboratory / Harbin Institute of Technology, IEEE Transactions on Multimedia (Vol. 27) 2025
> https://doi.org/10.1109/TMM.2025.3599074

# 한 줄 요약
<mark style="background: #FFF3A3A6;">DINO를 기반으로, 초기 feature를 short/long-range 정보로 함께 강화하는 deep MLP 백본과 backbone feature·encoder feature를 두 단계로 재결합하는 Cross Coding Twice Module(CCTM), 객체 크기·클래스를 함께 반영하는 Category-Size 소프트 라벨 기반 Boost Loss를 추가해 DETR 계열의 소형 객체 탐지 정확도(특히 클래스 예측 점수)를 개선한 모델.</mark>

> [!info] 내 메모
> 

# 정리

## 기존 방법의 한계
- **초기 feature 표현의 한계(짧은 범위 정보만 집계)**:
  대부분 DETR류는 ResNet을 백본으로 써서 작은 커널의 convolution으로 지역(short-range) 정보만 집계한다. 이 초기 feature는 이후 encoder·decoder 전체 성능에 캐스케이딩되는데, 소형 객체를 인식하는 데 필요한 문맥(context) 정보가 부족하다.
- **Transformer encoder에서의 object missing과 feature blurring**:
  Encoder의 다중 attention layer가 저해상도 feature에서 소형 객체에 제대로 attend하지 못해 객체를 놓치고, layer를 거듭할수록 부정확한 attention map이 feature를 점점 흐리게(blur) 만든다(Fig. 6에서 시각적으로 확인).
- **클래스 예측 점수의 크기 편향(box·class 예측의 분리)**:
  기존 방법들은 박스 회귀와 클래스 분류 loss를 독립적으로 계산해 객체 크기와 클래스 예측 신뢰도 사이의 관계를 무시한다. 그 결과 소형 객체는 대형 객체보다 일관되게 낮은 클래스 예측 점수를 받는다(저자의 DINO-4scale 통계 분석, Fig. 3).

## 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Small Object Detection을 위한 feature/샘플 보강**
- Feature representation 개선([16], [44], [52]), context 정보 통합([65], [68]), 클래스 불균형 보정([68]), 학습 샘플 증대([1], [26], [53], [75]).
- Normalized Wasserstein Distance([58])로 IoU의 위치 민감도 문제를 완화, RFLA([61])는 Gaussian receptive field 기반 label assignment로 anchor/anchor-free 방식의 스케일-샘플 불균형을 개선.
- CFINet([66])은 coarse-to-fine 2-stage 구조와 feature imitation learning으로 SODA 벤치마크를 새로 제시.
- SR-TOD([3])는 self-reconstruction 기반으로 tiny object를 눈에 띄게 만드는 방법으로, 기존 detector와 결합 가능.
- **타겟/해결**: 위 "초기 feature 표현의 한계(①)"를 부분적으로 다루지만 대부분 CNN 기반 detector에 대한 처방이며, DETR류의 encoder 내부 구조(attention 기반 blurring)나 클래스 예측 점수 편향(③)은 다루지 않는다.

**갈래 2 — DETR-like 구조 개선**
- 수렴 가속: Conditional DETR([43]), DN-DETR([28]), Stable-DINO([33]).
- Query 형식 개선: Efficient DETR([64]), DAB-DETR([32]), DINO([67]), MLP-DINO([4]).
- Encoder/decoder 재설계: Deformable-DETR([73]), CF-DETR([6]).
- DiffusionDet([9]): 노이즈 박스에서 객체 박스로 denoising하는 diffusion 기반 새 프레임워크.
- <mark style="background: #FFF3A3A6;">DQ-DETR([24]): 항공 이미지의 인스턴스 불균형을 완화하려고 query 개수를 동적으로 조정 — 이 위키의 Dynamic Query DETR 계열(갈래6_쿼리개수)과 같은 흐름.</mark>
- QueryDet([62]): 저해상도 feature 예측으로 고해상도 sparse convolution 위치를 좁히는 cascade sparse query — 고해상도 feature 자체의 품질 문제는 그대로 남음.
- **타겟/해결**: <mark style="background: #FFF3A3A6;">일반 객체 탐지에서 수렴 속도·query 효율성(구조적 문제)은 개선했지만, "DETR류 구조 자체를 SOD에 맞게 최적화"하는 연구는 부족하다 — 대부분 query 조정에 그치고, 초기 feature 표현과 encoder의 feature blurring 문제를 정면으로 다루지 않는다.</mark>

**갭**: <mark style="background: #FFF3A3A6;">SOD 전용 기법들은 CNN 기반 detector 위주이고, DETR류 개선 연구들은 일반 탐지 성능·수렴 속도·query 형식에 집중해왔다. "DETR류의 백본-encoder-decoder 캐스케이드 구조 안에서 초기 feature 품질, encoder의 feature blurring, 클래스 예측의 크기 편향을 함께 다루는" 시도는 없었다.</mark>

## 이 논문이 풀고자 하는 문제
1. DETR류 모델의 초기 feature 표현을 short-range와 long-range 문맥 정보를 모두 포함하도록 강화하는 것.
2. Transformer encoder를 거치며 소형 객체의 세부 정보가 blur·missing되는 문제를 backbone feature의 fine-grained 정보로 보완하는 것.
3. 객체 크기와 클래스 예측 점수 사이의 관계를 명시적으로 모델링해, 소형 객체의 낮은 클래스 예측 점수를 끌어올리는 것.

**갭 종합**: <mark style="background: #FFF3A3A6;">DETR류의 백본→encoder→decoder라는 캐스케이드 구조 자체가 SOD에 불리하게 작동한다는 것이 이 논문의 통찰이다 — 초기 feature가 부족하면 이후 모든 단계가 그 한계를 물려받고, encoder의 attention은 오히려 디테일을 지우며, box·class 예측의 독립적 처리는 크기 편향을 방치한다. Cross-DINO는 이 세 지점 각각에 대응하는 모듈(MLP 백본, CCTM, Boost Loss)로 갭을 메운다.</mark>

> [!info] 내 메모
> 

# 해결 방법 요약

| | 문제 ① — 초기 feature의 short-range 편중 | 문제 ② — encoder의 feature blurring/missing | 문제 ③ — 클래스 예측의 크기 편향 |
|---|---|---|---|
| **해결 방법** | Deep MLP(Strip-MLP 기반 CLAP-Strip-MLP)를 백본으로 써서 short+long range 정보를 함께 집계 | CCTM으로 backbone feature $B$의 fine-grained 정보를 encoder feature $E$에 두 단계로 재주입 | Category-Size(CS) soft label을 만들어 Boost Loss로 객체 크기에 따라 클래스 예측 손실을 재가중 |
| **예상되는 문제점** | MLP 계열은 원래 입력 크기가 고정된 가중치를 쓰는데, object detection은 입력 크기가 가변적이므로 별도의 CLAP(Cropping with Adaptive over-LaPping) 기법이 필요 — 추가 전처리 복잡도 | Cross-gating을 두 단계(1차 element-wise, 2차 GRN+MLP)로 반복해 연산·파라미터가 늘어날 수 있음(정량 비교는 본문에 없음) | 매우 작은 객체(예: 8×8 px, 1024×1024 이미지)는 정규화된 $cs_i$ 값이 0에 가까워져($\hat{cs}=0.0078$) $(1-\hat{cs})^\beta$ 항의 효과가 약해지는 한계를 저자도 인지, $\beta$로 보정 |

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">DINO 위에 (1) short-range와 long-range 정보를 함께 집계하는 <span style="color:#c0392b; font-weight:bold;">Deep MLP 백본</span>, (2) backbone feature와 encoder feature를 두 단계로 교차 인코딩하는 <span style="color:#c0392b; font-weight:bold;">Cross Coding Twice Module(CCTM)</span>, (3) 객체의 Category와 Size를 결합한 소프트 라벨 <span style="color:#c0392b; font-weight:bold;">Category-Size(CS)</span> 기반 <span style="color:#c0392b; font-weight:bold;">Boost Loss</span>를 추가한다.</mark>

## 전체 파이프라인 (Fig. 4 기준)

```
입력 이미지
       │
       ▼
① Deep MLP Backbone (CLAP-Strip-MLP)     → Backbone Feature B  (B, C, L)
       │
       ▼
Flatten → Transformer Encoder             → Encoder Feature E  (B, C, L)
       │
       ▼
② CCTM (B, E를 두 단계로 교차 인코딩)      → Cross Feature E_cf (B, C, L), 동일 shape 유지
       │                                    (Boost Loss는 E와 모든 decoder 예측에 적용)
       ▼
E_cf → Flatten, Top-K 선택 → Content/Positional Queries
       │
       ▼
Transformer Decoder (다중 layer, iterative box refinement)
       │
       ▼
FFN(classes), FFN(boxes)                  → 예측 (클래스, 박스)
       │
       ▼ (학습 시)
Boost Loss (Category-Size soft label 기반, encoder feature E와 모든 decoder 예측에 적용)
```

> [!info] 내 메모
> 

### ① Deep MLP 백본 — CLAP-Strip-MLP

- **역할**:
  ResNet 계열 백본은 작은 커널 convolution으로 지역(short-range) 정보만 집계해, 소형 객체 인식에 필요한 문맥 정보가 부족하다. Deep MLP 백본은 short-range와 long-range 정보를 동시에 집계해 더 풍부한 초기 feature 표현을 제공한다.
- **구현**:
  Strip-MLP([5])라는 효율적인 token interaction 모델을 DINO 프레임워크에 통합. Strip-MLP는 attention이 아니라 MLP 레이어로 짧고 긴 공간 범위의 정보를 동시에 집계한다. MLP 모델의 가중치는 입력 이미지 크기에 결합되어 고정 크기만 처리 가능하다는 한계가 있어, 논문은 **CLAP(Cropping with Adaptive over-LaPping)** 기법을 제안 — 이미지를 미니 패치 단위로 적응적으로 크롭·패딩하고 모든 패치에 동일한 가중치를 공유 적용해, 임의 크기 이미지를 처리할 수 있게 한다(세부 방법은 부록 A에 수록, 본 PDF에는 포함되지 않음).
- **입출력 shape**:
  입력 이미지 → Backbone Feature $B \in \mathbb{R}^{B \times C \times L}$ ($B$: 배치 크기, $C$: 채널 수, $L$: 토큰 길이). DINO와 동일하게 4-scale feature map 사용(공정 비교를 위해).

> [!example]- 구현 디테일
> CLAP-Strip-MLP는 Strip-MLP를 범용 vision backbone으로 확장한 버전으로, 논문은 이를 통해 "다운스트림 dense prediction task에서 중요한 역할"을 한다고 서술한다. ResNet50 대비 CLAP-Strip-T의 채널 수가 훨씬 적다(ResNet50: [512, 1024, 2048] vs CLAP-Strip-T: [160, 320, 640]) — Table I에서 파라미터·GFLOPs가 낮으면서도 높은 AP를 달성하는 이유 중 하나로 제시된다.

<mark style="background: #FFF9D6A6;">초기 feature가 short-range 정보에만 의존하면 소형 객체 주변 문맥을 놓치기 쉽다. Deep MLP는 attention보다 가벼운 연산으로 long-range 의존성까지 포착해, 이후 encoder·decoder가 물려받는 초기 정보의 품질 자체를 높인다 — DINO-CLAP-Strip-T가 DINO(ResNet50)보다 파라미터·GFLOPs가 적으면서도 AP가 더 높은 것(Table I, 49.0→51.7 AP)이 이를 뒷받침한다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> AI-TOD처럼 평균 객체 크기가 12.8px로 극도로 작은 데이터셋에서는 CLAP-Strip-T 적용 시 오히려 AP가 22.6%로 하락한다(DINO-R50 23.9% 대비). 저자는 이를 CLAP-Strip-T의 채널 수가 ResNet50보다 현저히 적어(31.25%) 매우 작은 객체를 구분하기에 정보가 불충분하기 때문이라고 설명한다 — MLP 백본의 이득이 데이터셋의 객체 크기 분포에 따라 반전될 수 있다는 한계.

> [!info] 내 메모
> 

### ② Cross Coding Twice Module (CCTM)

- **역할**:
  Transformer encoder는 attention 연산을 통해 feature를 정제하지만, 다중 layer를 거치며 오히려 feature를 blur시키고 소형 객체를 놓치는 경향이 있다(Fig. 6). Backbone feature $B$는 fine-grained 정보를 담고 있지만 encoder feature $E$에는 없다. CCTM은 이 두 feature를 교차 인코딩해 $E$가 $B$의 fine-grained 디테일을 점진적으로 흡수하게 한다.
- **구현**:
  두 단계로 구성. **1단계**는 cross-attention에서 영감을 받은 gating 방식으로 $B$, $E$ 각각에 Dense→LayerNorm→GELU를 적용해 gating map $B'$, $E'$을 만들고, $E$를 $B$ 기반 gating으로 재조합한 1차 cross feature $E^1_{cross}$를 만든다. **2단계**는 GRN(Global Response Normalization)과 MLP 레이어로 채널 방향 대비·선택성을 높이고, $B'$·$E'$의 곱(crossing gating map)으로 $B$의 유용한 fine-grained 정보를 다시 한 번 선택적으로 반영해 최종 Cross Feature $E_{cf}$를 만든다.
- **입출력 shape**:
  $B, E \in \mathbb{R}^{B \times C \times L}$ (동일 shape 공유) → $E_{cf} \in \mathbb{R}^{B \times C \times L}$ (입력과 동일 shape 유지).

```python
# 논문 Eq.(1)-(3) 기반. 1단계(taking E^1_cross as example)
E_prime = sigmoid(GELU(LayerNorm(FC(E))))              # gating map E'
E1_cross = E + B * (1 - E_prime)                        # 1차 cross coding (B'도 대칭적으로 계산)

# 2단계 — GRN·MLP로 채널 재가중 후 crossing gating map(B'E')으로 재선택
# B'_cross, E'_cross는 1단계 출력에 GRN+MLP를 적용해 얻음 (도식 Fig.5 기준)
E_cf = 2 * E * (B_prime * E_prime) + B * (1 - B_prime * E_prime)   # 논문 Eq.(3)
# E_cf, B'E'는 E1_cross와 동일한 feature dimension 공유
```

<mark style="background: #FFF9D6A6;">Encoder feature만으로는 fine-grained 디테일이 이미 blur된 상태이므로, backbone feature를 gating을 통해 "선택적으로" 재주입하면 encoder가 가진 semantic 정제 능력은 유지하면서 backbone의 디테일도 함께 살릴 수 있다 — 두 번의 gating(1차: 결합, 2차: GRN 기반 채널 재선택)으로 무분별한 합산이 아니라 적응적 선택이 이뤄진다는 점이 핵심. Ablation(Table VIII)에서 gating을 아예 빼고 MLP만 쓴 경우(AP 49.1)보다 한 번만 gating한 경우(49.4), 두 번 gating한 CCTM(49.8)이 순서대로 개선되어 "두 단계 gating" 설계 자체의 기여가 확인된다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> CCTM은 두 단계의 gating·GRN·MLP 연산을 encoder와 decoder 사이에 추가로 삽입하는 구조라 연산 오버헤드가 있을 것으로 예상되지만, 본문에는 CCTM 단독의 FLOPs/latency 증분이 별도로 보고되지 않는다(Table I의 전체 모델 GFLOPs만 확인 가능).

> [!info] 내 메모
> 

### ③ Boost Loss — Category-Size(CS) 소프트 라벨

- **역할**:
  기존 방법들은 박스 회귀와 클래스 분류 loss를 독립적으로 계산해, 객체 크기와 클래스 예측 신뢰도 사이의 관계를 무시한다. 저자의 통계 분석(Fig. 3, DINO-4scale on COCO val2017)에 따르면 대형 객체의 평균 클래스 예측 점수가 소형 객체보다 일관되게 높다. Boost Loss는 객체 크기를 클래스 확률 예측 과정에 명시적으로 결합해 소형 객체의 클래스 예측 점수를 강화한다.
- **구현**:
  Category-Size(CS)라는 새로운 소프트 라벨을 정의 — 객체의 크기 비율(이미지 대비 박스 높이·너비의 기하평균)과 클래스 라벨을 곱한 값. GT의 CS와 예측의 $\hat{CS}$를 각각 구성하고, 이를 정답 비교 없이 스케일링 팩터로 활용해 분류 확률에 곱한다. 작은 $\hat{cs}$ 값(=작은 객체)일수록 loss 가중치를 키워 모델이 소형 객체에 더 집중하게 한다. Positive 객체에만 적용(negative에 적용하면 negative 객체 수가 압도적으로 많아 노이즈만 증가).
- **입출력 shape**:
  예측 클래스 확률 $p_i$, GT/예측 박스 크기·클래스 → 스칼라 loss (encoder feature $E$의 예측과 모든 decoder layer의 예측에 각각 적용).

```python
# 논문 Eq.(4)-(5) 기반. i는 객체(박스) 인덱스
cs_i = sqrt((h_i / H) * (w_i / W)) * y_i      # GT Category-Size (y_i: 클래스 라벨)
# 예측에 대해서도 동일하게 hat_cs_i 구성

L_Boost = -(1/N) * sum_i [
    alpha * (1 - hat_cs_i ** beta) ** gamma * cs_i ** beta * log(p_i)     # positive
    + (1 - alpha) * p_i ** gamma * (1 - y_i) * log(1 - p_i)                # negative
]
# alpha, beta, gamma는 하이퍼파라미터. 기본값: beta=1.0, alpha=0.25, gamma=2.0
# (SODA-D/VisDrone은 beta=0.1, AI-TOD는 SR-TOD/RFLA 세팅을 따라 batch size 등 조정)
```

<mark style="background: #FFF9D6A6;">클래스 확률 예측에 객체 크기 정보를 직접 결합하면, 소형 객체의 loss 가중치가 자동으로 커져 모델이 학습 중 소형 객체의 클래스 예측에 더 집중하게 된다 — Fig. 3에서 Cross-DINO가 DINO보다 소형 객체 클래스 예측 점수를 일관되게 높이는 것으로 확인되며, Table VI/VII 전 backbone·데이터셋 ablation에서 Boost Loss 단독 적용만으로도 AP 개선이 관찰된다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> 매우 작은 객체(예: 1024×1024 이미지에서 8×8px 객체)는 정규화된 $\hat{cs}$가 0.0078처럼 0에 극도로 가까워져 $(1-\hat{cs}^{\beta})$ 항이 1에 근접, 재가중 효과가 오히려 희석된다 — 저자도 이 한계를 명시하고 $\beta$를 데이터셋별로 조정(SODA-D/VisDrone: 0.1)해 완화를 시도한다.

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| ① Deep MLP 백본 | 입력 이미지 | $B \in (B, C, L)$ | short+long range 초기 feature 집계 | CLAP-Strip-MLP (Strip-MLP + CLAP 가변 크기 처리) |
| Transformer Encoder | $B$ (flatten) | $E \in (B, C, L)$ | feature 정제(단, blurring 위험) | 표준 multi-layer self-attention encoder |
| ② CCTM | $B, E \in (B, C, L)$ | $E_{cf} \in (B, C, L)$ | backbone의 fine-grained 정보를 encoder feature에 재주입 | 2단계 cross-gating(1차 element-wise, 2차 GRN+MLP) |
| Decoder | $E_{cf}$ (Top-K), queries | 예측(클래스, 박스) | 반복적 박스 정제 | 표준 DINO decoder (dynamic positional + static content query) |
| ③ Boost Loss | 예측 확률 $p_i$, GT/예측 박스·클래스 | 스칼라 loss | 소형 객체 클래스 예측 점수 강화 | Category-Size 소프트 라벨 기반 재가중 focal-style loss |

> [!info] 내 메모
> 

# 실험 결과

### 설정
COCO2017[30], WiderPerson[69], VisDrone2019[72], AI-TOD[59], SODA-D[10] 5개 벤치마크. Ablation은 COCO2017·VisDrone2019에서 수행. 백본은 ResNet50/Swin-T/CLAP-Strip-T 세 종류. AdamW, weight decay $1\times10^{-4}$, 8×V100(COCO/WiderPerson), 12 epoch 기본(1× 스케줄, 별도 명시 시 24/36epoch), decoder query 900개+denoising query 100개(DINO 세팅 계승). $\beta=1.0, \alpha=0.25, \gamma=2.0$ 기본(SODA-D/VisDrone은 $\beta=0.1$).

### 핵심 결과 — Table I (COCO val2017, ResNet50/12epoch 기준)
**표를 보는 법**: DINO(ResNet50, 47M, 279G, 12epoch)가 Cross-DINO의 직접 baseline이다 — 같은 backbone·epoch 행끼리 비교하면 제안 모듈들의 순수 기여를 바로 확인할 수 있다.

| 모델 | Backbone | Epochs | AP | AP$_S$ | Params | GFLOPs |
|---|---|---|---|---|---|---|
| DINO[68] | ResNet50 | 12 | 49.0 | 32.0 | 47M | 279G |
| **Cross-DINO (ours)** | ResNet50 | 12 | **50.1 (+1.1)** | **33.1 (+1.1)** | 48M | 288G |

> [!note]- 세부 결과 및 Ablation
> #### 다양한 backbone·epoch에서의 개선 (Table I, 발췌)
> | 모델 | Backbone | Epochs | AP | AP$_S$ | Params | GFLOPs |
> |---|---|---|---|---|---|---|
> | DINO | ResNet50 | 24 | 50.4 | 33.3 | 47M | 279G |
> | Cross-DINO | ResNet50 | 24 | 51.4 (+1.0) | 34.1 (+0.8) | 48M | 288G |
> | DINO | Swin-T | 12 | 51.3 | 32.5 | 48M | 280G |
> | Cross-DINO | Swin-T | 12 | 52.1 (+0.8) | 36.9 (+2.4) | 49M | 302G |
> | H-Deformable-DETR | Swin-T | 36 | 53.2 | 35.9 | 66.8M | - |
> | Cross-DINO | Swin-T | 36 | 54.6 (+1.4) | 37.5 (+1.6) | 49M | 302G |
> | DINO | CLAP-Strip-T | 12 | 49.0 | 33.0 | 44M | 263G |
> | DINO-CLAP-Strip(ours) | CLAP-Strip-T | 12 | 51.7 (+1.7) | 35.0 (+3.4) | 44M | 277G |
> | **Cross-DINO (ours)** | CLAP-Strip-T | 12 | **52.6 (+3.6)** | **36.4 (+4.4)** | 45M | 277G |
> | Cross-DINO | CLAP-Strip-T | 36 | 54.6 | 37.5 | 45M | 277G |
>
> Abstract에서 강조하는 "36.4% AP$_S$, 45M 파라미터, DINO 대비 +4.4%p AP$_S$, 12epoch"가 CLAP-Strip-T backbone 행에 해당. DINO-CLAP-Strip(ResNet50→CLAP-Strip-T 교체만)만으로도 +2.7% AP(49.0→51.7) 개선되어 백본 교체 자체의 기여가 크다는 것을 보여준다.
>
> #### WiderPerson (Table II)
> | 모델 | Epochs | AP | Recall | mMR |
> |---|---|---|---|---|
> | DINO-ResNet50 | 24 | 92.75 | 99.08 | 40.08 |
> | Cross-DINO-ResNet50 (ours) | 24 | 93.29 (+0.54) | 99.64 | 39.94 |
> | DINO-Swin-T | 24 | 93.07 | 99.42 | 38.78 |
> | Cross-DINO-Swin-T (ours) | 24 | 93.93 (+0.86) | 99.65 | 37.88 |
> | DINO-CLAP-Strip (ours) | 24 | 93.19 | 99.42 | 38.21 |
> | Cross-DINO-CLAP-Strip (ours) | 24 | 93.92 (+0.73) | 99.65 | 36.91 |
>
> mMR(낮을수록 좋음) 기준 Cross-DINO-CLAP-Strip이 DINO 대비 -3.17%p 개선.
>
> #### VisDrone2019 (Table III)
> | 모델 | Eps | AP | AP$_{50}$ | AP$_{75}$ | AP$_{vt}$ | AP$_t$ | AP$_s$ |
> |---|---|---|---|---|---|---|---|
> | DINO-R50 | 12 | 31.7 | 54.1 | 31.9 | 5.9 | 14.0 | 27.4 |
> | Cross-DINO-R50 (ours) | 12 | 33.1 | 55.1 | 33.7 | 6.6 | 14.8 | 28.5 |
> | DINO-CLAP-Strip-T | 12 | 33.6 | 56.8 | 34.5 | 6.2 | 16.3 | 29.6 |
> | Cross-DINO-CLAP-Strip-T (ours) | 12 | 35.4 | 59.8 | 35.9 | 8.2 | 16.3 | 31.5 |
> | △ Improvement | - | 3.7 | 5.7 | 4.0 | 2.3 | 2.2 | 4.1 |
>
> DINO-R50 대비 ResNet50 백본에서 AP +1.4%p(31.7→33.1), CLAP-Strip-T 백본 도입 시 추가로 AP +1.9%p, AP$_{vt}$ +0.6%p 개선. DINO-R50 baseline 대비 전체로는 AP +3.7%p.
>
> #### AI-TOD (Table IV)
> | 모델 | Eps | AP | AP$_{50}$ | AP$_{75}$ | AP$_{vt}$ | AP$_t$ | AP$_s$ |
> |---|---|---|---|---|---|---|---|
> | DINO-R50 | 12 | 23.9 | 57.8 | 15.6 | 10.3 | 24.5 | 30.5 |
> | Cross-DINO-R50 (ours) | 12 | 23.7 | 57.5 | 15.4 | 9.7 | 21.8 | 27.4 |
> | Cross-DINO-CLAP-Strip-T | 12 | 22.6 | 54.7 | 13.4 | 9.5 | 23.7 | 28.7 |
> | Cross-DINO-CLAP-Strip-T-1200 | 12 | 25.1 | 58.9 | 17.5 | 13.2 | 25.4 | 31.2 |
>
> 평균 객체 크기 12.8px인 AI-TOD에서는 ResNet50 기준 Cross-DINO가 DINO보다 오히려 소폭 낮다(23.9→23.7 AP) — 저자는 이를 "AP 전체 개선폭이 작은 것"은 (1) 객체가 극도로 작아 SOD 자체가 더 어렵고 (2) Boost loss가 이미 작은 크기끼리는 가중치 차이가 작아 변별력이 떨어지기 때문이라고 설명한다. 입력 해상도를 1200×1200로 키운 "-1200" 버전에서만 개선(+1.2%p AP, +4.5%p AP$_{vt}$)이 확인된다.
>
> #### SODA-D (Table V)
> | 모델 | Eps | AP | AP$_{0.5}$ | AP$_{0.75}$ | AP$_{eS}$ | AP$_{rS}$ | AP$_{gS}$ | AP$_N$ |
> |---|---|---|---|---|---|---|---|---|
> | DINO | 12 | 27.7 | 55.5 | 23.8 | 11.7 | 27.7 | 33.9 | 42.9 |
> | Cross-DINO (ours) | 12 | 32.0 | 63.0 | 27.9 | 15.0 | 28.4 | 38.3 | 47.9 |
> | △ Improvement | - | 4.3 | 7.5 | 4.1 | 3.3 | 4.7 | 4.4 | 5.0 |
>
> CFINet(SOTA, AP 30.7) 대비도 +1.3%p AP, +0.3%p AP$_{eS}$, +3.3%p AP$_N$로 개선. DINO 대비 AP+4.3%p, AP$_{eS}$+3.3%p, AP$_N$+5.0%p.
>
> #### Ablation — CCTM·Boost Loss × 3 backbone (Table VI, COCO)
> | CCTM | Boost | Backbone | AP | AP$_{50}$ | AP$_{75}$ | AP$_S$ | AP$_M$ | AP$_L$ |
> |---|---|---|---|---|---|---|---|---|
> | | | ResNet50 | 49.0 | 66.6 | 53.5 | 32.0 | 52.3 | 63.0 |
> | ✓ | | ResNet50 | 49.8 | 67.6 | 54.5 | 32.4 | 53.0 | 64.3 |
> | | ✓ | ResNet50 | 49.6 | 67.0 | 54.0 | 32.2 | 53.2 | 63.5 |
> | ✓ | ✓ | ResNet50 | 50.0 | 67.8 | 54.7 | 32.5 | 53.5 | 65.4 |
> | | | Swin-T | 51.3 | 69.0 | 56.0 | 34.5 | 54.4 | 66.0 |
> | ✓ | | Swin-T | 52.0 | 69.8 | 56.9 | 35.4 | 55.1 | 66.8 |
> | | ✓ | Swin-T | 52.0 | 70.1 | 56.7 | 35.4 | 54.8 | 66.1 |
> | ✓ | ✓ | Swin-T | 52.1 | 70.3 | 56.7 | 36.9 | 55.4 | 67.0 |
> | | | CLAP-Strip-T | 51.7 | 69.6 | 56.8 | 35.5 | 55.1 | 66.0 |
> | ✓ | | CLAP-Strip-T | 52.4 | 70.6 | 57.5 | 35.7 | 56.4 | 67.2 |
> | | ✓ | CLAP-Strip-T | 52.6 | 70.8 | 57.5 | 36.2 | 56.4 | 68.0 |
> | ✓ | ✓ | CLAP-Strip-T | 52.6 | 70.8 | 57.6 | 36.4 | 56.6 | 68.2 |
>
> 세 backbone 모두에서 CCTM·Boost Loss 각각 단독으로도 개선, 함께 적용 시 대체로 가장 높은 AP$_S$ 달성.
>
> #### Ablation — CLAP-Strip-T·CCTM·Boost (Table VII, VisDrone2019, DINO-ResNet50 기준)
> | CLAP-Strip-T | CCTM | Boost | AP | AP$_{50}$ | AP$_{75}$ | AP$_{vt}$ | AP$_t$ | AP$_s$ |
> |---|---|---|---|---|---|---|---|---|
> | | | | 31.7 | 54.1 | 31.9 | 5.9 | 14.0 | 27.4 |
> | ✓ | | | 33.6 | 56.1 | 34.2 | 6.5 | 16.1 | 29.8 |
> | ✓ | ✓ | | 33.8 | 56.1 | 34.6 | 8.4 | 15.9 | 29.5 |
> | ✓ | ✓ | ✓ | 35.4 | 59.8 | 35.9 | 8.2 | 16.2 | 31.5 |
>
> CLAP-Strip-T 백본 교체 단독으로 AP +1.9%p(가장 큰 단일 기여), CCTM 추가 시 AP$_{vt}$가 6.5→8.4로 크게 개선, Boost Loss까지 더하면 AP 전체가 33.8→35.4로 재차 상승.
>
> #### CCTM 내부 구성요소 ablation (Table VIII, COCO, DINO-ResNet50 기준)
> | Ablation | AP | AP$_{50}$ | AP$_{75}$ | AP$_S$ | AP$_M$ | AP$_L$ |
> |---|---|---|---|---|---|---|
> | Baseline | 49.0 | 66.6 | 53.5 | 32.0 | 52.3 | 63.0 |
> | MLP only (w/o gating) | 49.1 | 66.9 | 53.3 | 32.3 | 52.3 | 62.9 |
> | Once-gating only | 49.4 | 66.8 | 53.9 | 32.1 | 52.9 | 63.5 |
> | CCTM (twice-gating) | 49.8 | 67.6 | 54.5 | 32.4 | 53.0 | 64.3 |
>
> Gating을 아예 빼면(MLP only) 거의 개선이 없고(AP +0.1%p), 1회 gating(once-gating)만으로도 유의미한 개선(+0.4%p), 2회 gating(CCTM)이 가장 우수해 "두 단계 교차 gating" 설계가 단순 MLP 결합보다 실질적으로 기여함을 보여준다.

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- **극소형 객체(AI-TOD 등)에서의 역전 현상**:
  <mark style="background: #FF5582A6;">평균 객체 크기가 12.8px인 AI-TOD에서 ResNet50 기준 Cross-DINO가 DINO보다 AP가 오히려 낮다(23.9→23.7). 저자는 Boost Loss의 크기 기반 재가중이 이미 다들 "매우 작은" 크기끼리는 변별력을 잃기 때문이라고 설명하지만, 이는 Boost Loss가 "소형 객체를 더 강조한다"는 설계 의도가 극단적인 크기 영역에서는 반대로 무력화될 수 있음을 시사한다.</mark>
- **MLP 백본의 채널 수 트레이드오프**:
  CLAP-Strip-T는 ResNet50보다 채널 수가 훨씬 적어(31.25%) 파라미터·GFLOPs 효율은 좋지만, AI-TOD처럼 극도로 작은 객체가 많은 데이터셋에서는 오히려 정보 부족으로 성능이 하락한다(Table IV, 22.6% AP) — 경량화와 극소형 객체 인식 사이의 트레이드오프가 명확히 존재한다.

### 한계
- <mark style="background: #FF5582A6;">CLAP 기법 자체의 세부 구현(미니 패치 크롭·오버래핑 방식)이 본문이 아니라 supplementary material(부록 A)에만 있어, 본 PDF만으로는 정확히 어떻게 임의 크기 이미지를 처리하는지 완전히 검증할 수 없다.</mark>
- <mark style="background: #FF5582A6;">AI-TOD 결과(Table IV)에서 보듯, 극소형 객체가 지배적인 데이터셋에서는 제안 방법의 이득이 명확하지 않거나 오히려 역전되는 사례가 있다 — 저자도 이를 인지하고 원인 두 가지(데이터셋 난이도, Boost Loss의 변별력 저하)를 제시하지만 해결책은 입력 해상도를 키우는 것(1200×1200) 외에 제시하지 않는다.</mark>
- CCTM·Boost Loss 각 모듈의 단독 연산 비용(latency, 추가 FLOPs)이 전체 모델 수치로만 보고되고 모듈별로 분리되어 있지 않아, 어떤 모듈이 연산 대비 이득이 가장 큰지 정량적으로 판단하기 어렵다.
- $\alpha, \beta, \gamma$ 하이퍼파라미터가 데이터셋마다 다르게 설정되는데(SODA-D/VisDrone은 $\beta=0.1$, 기본은 $\beta=1.0$), 이 값을 어떻게 탐색했는지(grid search 범위 등) 구체적인 근거는 제시되지 않는다.

### 생각할 점
- <mark style="background: #A6E3A1A6;">CCTM의 "backbone feature를 encoder feature에 게이팅으로 재주입"하는 아이디어는 UAV-DETR의 SAC(Semantic Alignment and Calibration)가 서로 다른 fusion 경로의 feature를 정렬하는 것과 목적은 다르지만, "정보가 손실되는 지점에 원본에 가까운 신호를 다시 섞어준다"는 상위 전략은 유사하다 — 두 방법 모두 대상은 다르지만(CCTM: backbone↔encoder, SAC: fusion 경로 간) "어디서 정보가 손실되는지 정확히 짚고 그 지점에 직접 개입한다"는 설계 원칙을 공유한다.</mark>
- Boost Loss의 Category-Size soft label은 박스 크기와 클래스를 곱해 단일 스칼라로 압축하는데, 이 압축 과정에서 "박스의 종횡비"(가늘고 긴 객체 등) 정보는 사라진다 — 종횡비가 극단적인 객체(예: 전선, 표지판)에도 이 방법이 동일하게 유효할지는 검증되지 않았다.

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">이 위키의 "feature 강화" 계열([[2025_RemoteSensing_FANet|FANet]]의 주파수 attention, [[2024_ECCV_SR-TOD|SR-TOD]]의 reconstruction difference map, [[2025_arXiv_UAV-DETR|UAV-DETR]]의 FF 모듈)은 모두 "약한 feature 자체를 보강"하는 데 집중하는 반면, Cross-DINO는 feature 보강(CCTM)과 손실 함수 재설계(Boost Loss)를 함께 다룬다는 점에서 [[Object_Detection_Approaches]]의 "feature 강화" 축과 "label assignment/loss 재설계" 축을 한 논문 안에서 동시에 다루는 사례로 추가할 만하다 — 특히 UAV-DETR과 마찬가지로 DINO/DETR 계열이라는 점에서, "DETR 계열에서 feature 강화를 어떻게 하는가"를 비교하는 직접적인 대조군이 된다(UAV-DETR: 주파수 도메인, Cross-DINO: backbone-encoder 게이팅 재주입).</mark>
- [[2026_TIP_Unc-SOD|Unc-SOD]]의 uncertainty 기반 label assignment와 Cross-DINO의 Category-Size 기반 Boost Loss는 둘 다 "크기 정보를 학습 신호에 직접 반영한다"는 공통점이 있다 — Unc-SOD는 예측 불확실성을, Cross-DINO는 객체 크기 자체를 스케일링 팩터로 쓴다는 점에서 다르며, 두 신호를 결합한 재가중 방식도 고려할 만하다.

> [!info] 내 메모
> 

# 관련 개념
- [[Cross_Coding_Twice_Module]] — 이 논문의 핵심 기여. Backbone feature와 encoder feature를 두 단계 gating으로 교차 인코딩해 encoder의 feature blurring 문제를 보완하는 기법.

# 관련 문서
- 비교: [[Object_Detection_Approaches]] — feature 강화 축(backbone-encoder 간 fine-grained 정보 재주입)이자, [[2025_arXiv_UAV-DETR|UAV-DETR]]과 함께 이 비교 문서에서 DETR/DINO 계열 SOD 특화 논문의 두 번째 사례로 추가할 가치가 있다. UAV-DETR이 RT-DETR 기반 주파수 도메인 접근인 데 비해, Cross-DINO는 DINO 기반으로 backbone-encoder feature 재결합과 크기 인지 손실 함수를 함께 쓴다는 점에서 구별된다.
- Baseline: DINO[67] (Zhang et al., ICLR 2023) — 이 논문이 직접 확장하는 baseline 구조. 아직 위키에 노트 없음 #pending:dino
- 관련: [[2024_ECCV_DQ-DETR|DQ-DETR]] — 같은 SUSTech/PCL 저자진(Huang 등)이 참여한 선행 연구로, query 개수를 동적으로 조정해 항공 이미지 인스턴스 불균형을 완화. Cross-DINO는 query 조정이 아니라 feature 품질·손실 함수에 집중한다는 점에서 상호보완적.

# 읽어볼 만한 논문
- 참고문헌 기반: Z. Xu, C. Xu, J. Yang, and Z. Yu, "MLP-DINO: Category modeling and query graphing with deep MLP for object detection" [4] (Proc. 33rd Int. Joint Conf. Artif. Intell., 2024) — Cross-DINO 저자 일부가 참여한 직접적인 선행 연구로 보이며, deep MLP를 DINO에 결합하는 아이디어의 직계 계보. Cross-DINO와의 차이(CCTM·Boost Loss 추가 여부)를 비교하며 읽으면 이 논문의 증분 기여를 명확히 파악할 수 있다.
- 참고문헌 기반: G. Cao et al., "Strip-MLP: Efficient token interaction for vision MLP" [5] (Proc. IEEE/CVF Int. Conf. Comput. Vis., 2023) — Cross-DINO의 Deep MLP 백본이 직접 기반으로 삼는 원조 아키텍처. CLAP 기법을 이해하려면 먼저 Strip-MLP의 token interaction 방식을 알아야 한다.
- 참고문헌 기반: Y.-X. Huang, H.-I Liu, H.-H. Shuai, and W.-H. Cheng, "DQ-DETR: DETR with dynamic query for tiny object detection" [24] (Proc. Eur. Conf. Comput. Vis., 2025) — 이 위키에 이미 있는 [[2024_ECCV_DQ-DETR|DQ-DETR]]. Related Work에서 직접 비교 대상으로 인용되며, DETR류를 SOD에 최적화하는 또 다른 축(query 동적 조정)이라 Cross-DINO(feature/loss 축)와의 상호보완성을 이해하는 데 유용.
- 참고문헌 기반: C. Xu, J. Wang, Y. Yang, H. Wei, and G.-S. Xia, "RFLA: Gaussian receptive field based label assignment for tiny object detection" [61] (Proc. Eur. Conf. Comput. Vis., 2022) — 이 위키의 여러 다른 소형 객체 탐지 논문(FANet, BAFNet 등)에서도 반복 인용되는 핵심 선행 연구. 아직 위키에 없어 우선순위가 높다.
- 자유 추천(검증 필요): DINO 계열에 주파수 도메인 feature 강화를 결합한 후속 연구가 있는지 — 검색 키워드: `DINO DETR frequency domain feature enhancement small object detection`. UAV-DETR(RT-DETR 기반)과 Cross-DINO(DINO 기반)를 잇는 세 번째 DETR 계열 SOD 논문이 있는지 확인할 때 참고.
