---
pm-task: true
projectId: "paperwiki-small-object-detection"
parentId:
id: "t-bafnet-9wfeflzzfm"
title: "Boundary-Aware Feature Fusion With Dual-Stream Attention for Remote Sensing Small Object Detection"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-08-24"
due:
progress: 0
assignees: []
tags: []
customFields:
  "7l8l795xmtcjvlf6": 2025
  "1frf59rymtcjvske": "IEEE TGRS"
subtaskIds: []
dependencies: []
year: 2025
venue: "IEEE Transactions on Geoscience and Remote Sensing (TGRS)"
jcr_quartile: Q1
task: [small-object-detection]
direction: [novel-approach, improvement]
paper_tags: [paper, small-object-detection, remote-sensing, attention-mechanism, feature-fusion, boundary-supervision, auxiliary-task]
source: "/Users/GyeongSeo/Workspace/논문_pdf/Small_Object_Detection/2025_TGRS_BAFNet.pdf"
createdAt: "2026-08-24T03:28:00.000Z"
updatedAt: "2026-08-28T17:30:00.000Z"
---

Project: [[논문_Small_Object_Detection|Small Object Detection]]
#paper #small-object-detection #remote-sensing #attention-mechanism #feature-fusion #boundary-supervision #auxiliary-task

> [!quote] 원제
> **Boundary-Aware Feature Fusion With Dual-Stream Attention for Remote Sensing Small Object Detection**
> Jingnan Song, Mingliang Zhou, Jun Luo, Huayan Pu, Yong Feng, Xuekai Wei, Weijia Jia — Chongqing University 외, IEEE TGRS 2025

# 한 줄 요약
<mark style="background: #FFF3A3A6;">최고레벨 feature에서 전경(FPAM)·배경(BPAM) attention map을 동시에 생성해 저레벨 feature와 결합하는 Dual-Stream Attention Module(DSAM)과, 예측한 경계를 Laplacian pyramid 기반 GT로 supervision해 cross-scale fusion 중 손실되는 경계 정보를 보존하는 Boundary-Aware Branch를 결합한 BAFNet — AI-TOD/VisDrone/DIOR/LEVIR-Ship 네 원격탐사 벤치마크에서 SOTA를 달성한 DetectoRS 기반 탐지기.</mark>

> [!info] 내 메모
> 

# 정리

| | 문제 ① — 배경과의 혼동 | 문제 ② — Cross-scale fusion 중 경계 정보 손실 |
|---|---|---|
| **문제 정의** | 원격탐사 영상의 소형 객체는 픽셀 비율이 낮고 외형이 흐릿해, 복잡하고 다양한 지형·텍스처 배경과 구별하기 어렵다. | 고레벨 semantic feature(문맥 정보 풍부)와 저레벨 spatial feature(공간 디테일 풍부)를 직접 융합하면 boundary-blurring이나 over-erosion이 발생해 세밀한 feature 정보가 손실된다. |
| **풀고자 하는 문제** | 고레벨 semantic feature로부터 전경과 배경 정보를 동시에, 상호보완적으로 추출해 저레벨 spatial feature와 결합하는 것 | Cross-scale feature fusion 과정에서 손실되기 쉬운 객체 경계 디테일을 명시적으로 보존하는 것 |
| **선행 연구 접근** | - FPN[1]/PANet[2]/BiFPN[3]: feature 융합 구조 개선<br>- MHN[26]/HRNet[27]/ASFF[28]: 레벨 간 semantic 불일치 해소, 다중 스케일 표현 강화<br>- 문맥 관계 모델링([6]~[11]): 객체-주변 문맥 의존성 캡처<br>- **갭**: 융합 구조·문맥 모델링을 개선할 뿐, "전경-배경을 동시에 상호보완적으로" 모델링하는 이중 스트림 설계는 없음. | - DetectoRS[31]: recursive feature pyramid + switchable atrous convolution<br>- FSANet[32]: feature-aware alignment + spatial-aware guidance head로 반복 정제<br>- **갭**: feature 정렬·반복 정제는 다루지만, 경계 정보를 명시적 supervision 신호로 활용하지는 않음. |
| **해결 방법** | 최고레벨 feature `P4`로 전경 attention(FPAM)을 만들고, 그 여집합(`1-FPAM`)으로 배경 attention(BPAM)을 동시에 얻어 저레벨 feature `P0`에 각각 적용 — 배경 억제를 명시적 별도 신호로 다룬다. | Laplacian pyramid로 만든 멀티스케일 GT 경계 맵으로 boundary head를 supervision해, DSAM으로 강화된 feature가 경계 정보를 보존하도록 학습을 유도한다. |
| **예상되는 문제점** | BPAM이 FPAM의 단순 여집합(`E-FPAM`)이라 별도로 학습되지 않고 FPAM 품질에 전적으로 종속된다 — FPAM이 부정확하면 BPAM도 자동으로 부정확해지는 구조적 종속성. | Boundary GT를 Laplacian pyramid로 만드는 과정 자체가 추가 연산 오버헤드이며, 이 비용에 대한 정량 보고가 논문에 없다. |

**갭 종합**: <mark style="background: #FFF3A3A6;">선행 연구들은 feature fusion 구조 개선, 문맥 모델링, label assignment를 각각 따로 발전시켜 왔지만, "경계(boundary) 정보의 명시적 보존"이라는 관점을 정면으로 다룬 연구는 드물었다. 전경과 배경을 상호보완적으로 모델링하는 이중 스트림 attention과, 경계를 별도 supervision 신호로 보완하는 방법을 결합한 시도가 BAFNet의 통찰이다.</mark>

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">최고레벨 feature `P4`로부터 <span style="color:#c0392b; font-weight:bold;">전경 우선 attention map(FPAM)과 그 여집합인 배경 우선 attention map(BPAM)을 동시에 생성</span>해 양방향 문맥으로 저레벨 feature `P0`를 강화하고(DSAM), 동시에 <span style="color:#c0392b; font-weight:bold;">Laplacian pyramid로 만든 멀티스케일 경계 GT로 boundary head를 supervision</span>해 이 강화 과정에서 경계 정보가 손실되지 않도록 보조한다(Boundary-Aware Branch).</mark>

## 전체 파이프라인 (Fig. 1 기준)

```
입력 이미지
       │
       ▼
① Backbone (ResNet-50) + Neck (RFP)         → P0, P1, P2, P3, P4 (다중스케일 feature)
       │
       ▼ (P0, P4만 사용)
② DSAM (Dual-Stream Attention Module)        → S0 (강화된 저레벨 feature, P0와 동일 채널·해상도)
       │
       ├──────────────────────────────┐
       ▼                              ▼ (학습 시에만)
③ Detection Head (S0, P1~P4 입력)      ④ Boundary-Aware Branch (S0 입력)
       │                              │
       ▼                              ▼
출력: (cls, box)                     boundary loss L_b
```

> [!info] 내 메모
> 

### ① Backbone + Neck
- **역할**: 원본 이미지에서 다중 스케일 feature map `P0~P4`를 추출한다. `P0`는 가장 저레벨(고해상도, 공간 디테일 풍부), `P4`는 가장 고레벨(저해상도, semantic 정보 풍부).
- **구현**: ImageNet 사전학습 ResNet-50 backbone + RFP(Recursive Feature Pyramid, DetectoRS[31]에서 제안된 neck) — RFP는 feature pyramid를 반복적으로 backbone에 되먹임해 정제하는 구조.
- **입출력 shape**: 입력 이미지 `(3, H, W)` → `P0, P1, P2, P3, P4` (해상도가 단계마다 절반씩 줄고 채널은 늘어나는 표준 FPN 계열 피라미드).

> [!info] 내 메모
> 

### ② Dual-Stream Attention Module (DSAM)
- **역할**: 최고레벨 feature `P4`의 풍부한 semantic 정보로 "여기가 전경(객체)인지, 배경인지"를 동시에 판단하는 attention map 두 장을 만들고, 이를 저레벨 feature `P0`에 곱해 전경·배경을 각각 강조한 feature를 얻은 뒤 dilated convolution으로 다양한 크기의 문맥을 포착해 최종 강화 feature `S0`를 만든다.
- **구현**:
  1. `P4`에 [[1x1_Convolution]] + upsampling + sigmoid를 적용해 Foreground Priority Attention Map(FPAM) `A^F`를 생성.
  2. 전체 1 행렬 `E`에서 `A^F`를 빼서 Background Priority Attention Map(BPAM) `A^B = E - A^F`를 얻음 — 별도 학습 없이 FPAM으로부터 상보적으로 유도.
  3. 저레벨 `P0`에 `A^F`, `A^B`를 각각 element-wise 곱해 전경/배경 강조 feature를 얻고, [[Dilated_Convolution]] 4개 브랜치(1×1 conv 1개 + rate 3/5/7 dilated conv 3개, 파라미터 비공유)로 각각 처리.
  4. 두 브랜치 결과를 concat 후 3×3 conv+ReLU로 `E0`, 이를 다시 `P0`와 concat해 최종 `S0`.
- **입출력 shape**: `P4` + `P0` → `S0` (P0와 동일 채널·공간 해상도).

```python
# 의사코드 (논문 수식 1-3 기반)
A_F = sigmoid(upsample(conv1x1(P4)))          # FPAM
A_B = E - A_F                                  # BPAM, E=전체 1 행렬
fg = concat([D1(P0 * A_F), D2(P0*A_F), D3(P0*A_F), D4(P0*A_F)])  # D1=1x1conv, D2~4=dilated(rate3,5,7)
bg = concat([D1(P0 * A_B), D2(P0*A_B), D3(P0*A_B), D4(P0*A_B)])
E0 = conv3x3_relu(concat([fg, bg]))
S0 = concat([E0, P0])
```

(연한 노랑 하이라이트) <mark style="background: #FFF9D6A6;">고레벨 feature의 풍부한 semantic이 전경-배경 구별에 유리하다는 관찰을, 전경 attention과 그 여집합(배경 attention)을 동시에 저레벨 spatial feature에 적용함으로써 활용한다 — "정리" 표의 첫 번째 문제(배경과의 혼동)를, 객체만 강조하는 단일 스트림이 아니라 배경 억제까지 명시적으로 모델링하는 이중 스트림으로 해결한다는 것이 핵심 차별점이다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> BPAM이 FPAM의 단순한 여집합(`E-A^F`)으로 정의되어, 별도로 학습되지 않고 FPAM의 품질에 전적으로 종속된다 — FPAM이 부정확하면 BPAM도 자동으로 부정확해지는 구조적 종속성이 있으나, 논문은 이 상호의존성이 실패하는 경우를 별도로 분석하지 않는다.

> [!info] 내 메모
> 

### ③ Detection Head
- **역할**: DSAM으로 강화된 `S0`와 나머지 피라미드 레벨(`P1~P4`)을 함께 입력받아 최종 (클래스, 박스)를 예측한다.
- **구현**: DetectoRS 계열 detection head(논문은 head 자체를 새로 제안하지 않고 DetectoRS의 head를 그대로 사용).
- **입출력 shape**: `S0, P1, P2, P3, P4` → `(cls, box)`.

> [!info] 내 메모
> 

### ④ Boundary-Aware Branch (학습 시에만)
- **역할**: 강화된 저레벨 feature `S0`로부터 경계(boundary)를 예측하고, 입력 이미지에서 Laplacian pyramid로 만든 GT 경계 맵과 비교해 경계 정보가 fusion 과정에서 손실되지 않도록 추가 학습 신호를 준다.
- **구현**:
  1. 입력 이미지를 그레이스케일 `g_d`로 변환, stride 1/2/4의 Laplacian convolution(2D conv + 학습 가능한 1×1 conv)으로 멀티스케일 경계 feature `r1, r2, r4` 추출.
  2. `r2, r4`를 업샘플링 후 `r1`과 concat해 경계 feature map `g_p` 구성, conv로 예측 경계 맵 `f_b` 생성 후 threshold 0.1로 이진 GT 변환.
  3. `S0`를 별도 boundary head(BHead)에 입력해 경계를 예측하고, GT 경계 맵과 weighted binary cross-entropy loss(경계 픽셀에 높은 가중치)로 학습.
- **입출력 shape**: `S0` → boundary 예측 `bd` (입력 이미지와 같은 공간 비율의 이진 맵), loss는 스칼라.

```python
# 의사코드 (논문 수식 4-6, Algorithm 1 기반)
g_d = grayscale(input_image)
r1, r2, r4 = laplacian_pyramid(g_d, strides=[1, 2, 4])   # 2D conv + 학습되는 1x1 conv
g_p = concat([r1, upsample(r2), upsample(r4)])
f_b = conv(g_p)                                            # 예측 경계 맵
gt_boundary = (f_b > 0.1)                                  # 이진화

bd = boundary_head(S0)                                     # S0 -> 경계 예측
L_b = weighted_bce(bd, gt_boundary)                        # 경계 픽셀에 높은 가중치
L_total = alpha * L_b + L_cls + L_reg                      # alpha=0.5(ablation 최적)
```

(연한 노랑 하이라이트) <mark style="background: #FFF9D6A6;">"정리" 표의 두 번째 문제(cross-scale fusion 중 경계 정보 손실)를, 경계를 별도의 auxiliary supervision 신호로 명시화해 fusion 과정 자체가 이 정보를 보존하도록 학습을 유도함으로써 해결한다 — DSAM만으로는 얻지 못하는 fine-grained localization 정밀도(Table V에서 boundary branch 추가 시 AP_s가 21.0→22.4로 개선)를 이 supervision이 별도로 제공한다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> Boundary GT를 Laplacian pyramid로 생성하는 과정 자체가 추가 연산이며, 이 오버헤드(학습/추론 시간)에 대한 정량 보고가 이 논문에 없다. 또한 boundary branch는 학습에만 관여하므로(추론 시 제거) 추론 속도에는 영향이 없지만, 학습 비용은 늘어난다.

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| ① Backbone+Neck | (3, H, W) | P0~P4 (피라미드) | 다중스케일 feature 추출 | ResNet-50 + RFP(DetectoRS neck) |
| ② DSAM | P4, P0 | S0 (P0와 동일 shape) | 전경·배경 강조 + 경계 보존용 강화 | 1×1conv+sigmoid(FPAM/BPAM) + Dilated Conv 4branch |
| ③ Detection Head | S0, P1~P4 | (cls, box) | 최종 객체 예측 | DetectoRS head |
| ④ Boundary-Aware Branch | S0 (학습시만) | boundary 예측 + L_b | 경계 정보 보존 supervision | Laplacian pyramid GT + weighted BCE |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table I (AI-TOD test)
**표를 보는 법**: 각 행이 하나의 모델, `AP_vt`가 극소형(very tiny) 객체 성능이라 — 이 논문이 특히 강조하는 지표다.

| 벤치마크 | 지표 | Before(CAF²ENet-S, 경쟁 SOTA) | After(BAFNet) |
|---|---|---|---|
| AI-TOD | AP / AP_vt | 30.2 / 12.8 | 30.5 / 16.6 (+3.8%p) |
| VisDrone val | AP / AP_s | 29.2(CMDNet) / 17.5(DINO) | 30.8 / 22.4 (+4.9%p) |
| DIOR | mAP | 76.2(SDPNet, 2위) | 78.1 (+1.9%p) |
| LEVIR-Ship | AP50 | 83.3(ORFENet) | 84.2 |

> [!note]- 세부 결과 및 Ablation
> #### Table V — 메인 ablation (DSAM·BAB 유무, VisDrone val)
> **보는 법**: DSAM/BAB 체크 유무에 따른 4개 행 비교 — 개별(+2.3/+2.7)과 결합(+3.8) 개선폭을 대조.
> DSAM·BAB 개별 기여가 비슷한 수준(+2.3/+2.7)이며, 결합 시 단순 합보다 약간 더 큰 개선(+3.8) — 두 모듈이 상호보완적으로 작동함을 시사. AP_s 개선폭(+4.7)이 AP_l 개선폭(-1.3, 오히려 하락)보다 훨씬 커 소형 객체 특화 효과가 뚜렷.
>
> #### Table VI — DSAM feature layer 비교
> **보는 법**: P3/P4 중 어느 레이어로 FPAM·BPAM을 만드는지 비교 — P4 단독이 가장 높음(AP 30.8 vs P3+P4 조합보다 우수).
> 가장 높은 semantic 레벨(P4)이 전경/배경 attention 생성에 가장 유효함을 확인.
>
> #### Table VII — Boundary aggregation 방법 비교 (Canny vs Laplacian)
> **보는 법**: 같은 위치에 Canny 대신 Laplacian pyramid를 썼을 때 전 지표가 개선되는지 확인.
> Canny 29.8 AP < Laplacian(채택) 30.8 AP — 멀티스케일 경계 추출 능력의 차이로 저자가 해석.
>
> #### Table VIII — α(boundary loss 가중치) 하이퍼파라미터
> **보는 법**: α=0(BAB 없음)부터 1.0까지 6개 값 중 최고점을 확인.
> α=0(29.3) → 0.5(채택, 30.8, 최고) → 1.0(30.3) — 낮으면 boundary loss 기여가 약하고, 높으면 detection loss를 압도해 오히려 성능 저하.
>
> #### Fig. 5 — Heatmap 정성 비교 (VisDrone)
> **보는 법**: 원본/baseline/BAFNet 세 열의 attention heatmap을 비교 — BAFNet 열이 전경 객체에 더 선명하게 집중하는지 눈으로 확인.
>
> #### Fig. 6 — DIOR 정성 비교
> **보는 법**: 노란/빨간 점선 박스가 각각 소형 객체 탐지·오탐 감소 사례를 표시 — 다른 방법들이 놓친 위치를 BAFNet이 잡아내는지 확인.

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- BPAM이 FPAM의 단순한 여집합으로 정의되어, FPAM이 실제로 배경을 잘못 강조하는 경우(예: 배경이 전경과 유사한 텍스처)에 BPAM도 함께 오염되는 구조적 종속성이 있으나, 논문은 이런 실패 사례를 별도로 분석하지 않는다.
- Table V에서 AP_l(대형 객체)이 DSAM+BAB 결합 시 오히려 하락(44.5→43.2) — 저자는 이 하락을 별도로 논의하지 않으며, 소형 객체 특화 설계가 대형 객체 성능을 일부 희생시킬 가능성에 대한 명시적 분석이 없다.

### 한계
- <mark style="background: #FF5582A6;">AI-TOD에서 AP50(59.8)이 CAF²ENet-S(63.7)보다 낮음 — 특정 IoU 임계값(0.5)에서는 경쟁 SOTA에 못 미치며, 이 논문의 우위는 주로 엄격한 지표(AP75, AP_vt)에서 나타난다.</mark>
- Backbone이 ResNet-50으로 고정되어 있고, 최신 Transformer 기반 backbone(Swin-T 등)과의 결합 실험은 이 논문에서 수행되지 않음.
- Conclusion에서 저자가 직접 명시: 향후 "노이즈 처리"와 "노이즈에 더 민감한 탐지 알고리즘"을 다룰 계획이라고만 언급할 뿐, 현재 버전의 노이즈 강건성에 대한 정량 분석은 이 논문에 없다.

### 생각할 점
- <mark style="background: #A6E3A1A6;">FPAM/BPAM의 "attention map과 그 여집합"이라는 설계는, 이 위키의 다른 attention 강화 논문들이 대체로 전경 강조 attention만 만드는 것과 달리, 배경 억제를 명시적 별도 신호로 다룬다는 점에서 독특하다 — 별도 파라미터 없이 "1에서 빼기"만으로 상보적 신호를 얻는 경량 설계가 인상적이다.</mark>
- <mark style="background: #A6E3A1A6;">Boundary-Aware Branch는 학습에만 관여하는 auxiliary head라는 점에서 [[ORFENet]]의 ORB, [[FFSSTDNet]]의 FSR과 같은 "학습 시에만 존재하는 auxiliary branch" 패턴을 공유하지만, 재구성 대상이 이미지나 마스크가 아니라 "경계"라는 점에서 세 번째 변형이다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">Table V의 DSAM(+2.3)·BAB(+2.7) 개별 기여가 비슷한 수준이라는 점은, 두 모듈이 서로 다른 종류의 정보(전경/배경 문맥 vs 경계 디테일)를 다루기 때문일 가능성이 있으며, "정보 소스가 이질적일수록 개별 기여가 더 균형 있게 나뉜다"는 가설을 세워볼 수 있다.</mark>
- <mark style="background: #A6E3A1A6;">[[ORFENet]]의 ORB(foreground/background 이진 마스크 재구성)와 이 논문의 Boundary-Aware Branch(경계 예측)는 목적이 유사하지만 supervision target이 다르다(영역 vs 경계) — 두 auxiliary task를 동시에 결합하면 상호보완적으로 더 큰 개선이 가능할지 검토할 가치가 있다.</mark>

> [!info] 내 메모
> 

# 관련 개념
- [[Dual_Stream_Foreground_Background_Attention]] — 이 논문의 DSAM 핵심 기여. 고레벨 semantic feature로부터 전경 attention과 그 여집합인 배경 attention을 동시에 생성해 저레벨 feature를 상호보완적으로 강화하는 기법.
- [[1x1_Convolution]] — FPAM 생성, DSAM의 dilated conv 브랜치 중 하나로 사용.
- [[Dilated_Convolution]] — DSAM에서 전경·배경 강조 feature로부터 다양한 크기의 문맥을 포착하는 데 사용(rate 3/5/7).

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]] — feature 강화 계열에 새로 추가. attention(전경+배경 이중 스트림) + auxiliary boundary supervision 결합이라는 조합은 이 위키에서 처음.

# 읽어볼 만한 논문
- 참고문헌 기반: S. Qiao, L.-C. Chen, A. Yuille, "DetectoRS: Detecting objects with recursive feature pyramid and switchable atrous convolution" (CVPR 2021) [31] — 이 논문의 detector 및 neck(RFP) 뼈대. BAFNet의 모듈이 어디에 삽입되는지 이해하려면 필수.
- 참고문헌 기반: C. Xu, J. Wang, W. Yang, H. Yu, L. Yu, G.-S. Xia, "RFLA: Gaussian receptive field based label assignment for tiny object detection" (ECCV 2022) [13] — 이 논문이 채택한 label assignment 전략. 이 위키에서 이미 여러 논문이 인용하는 핵심 선행 연구로 우선순위가 매우 높음(아직 위키에 없음, #pending:rfla).
- 참고문헌 기반: J. Wu, Z. Pan, B. Lei, Y. Hu, "FSANet: Feature-and-spatial-aligned network for tiny object detection in remote sensing images" (IEEE Trans. Geosci. Remote Sens., 2022) [32] — AI-TOD·VisDrone 비교표에서 반복 등장하는 강력한 경쟁 방법. [[ORFENet]] 노트에서도 이미 1-stage SOTA로 언급된 논문.
- 자유 추천(검증 필요): 이미지 segmentation에서 경계 인식(boundary-aware) supervision을 auxiliary loss로 쓰는 연구 — 검색 키워드: `boundary-aware auxiliary loss semantic segmentation edge supervision`. Boundary-Aware Branch의 설계가 segmentation 분야의 유사 기법에서 얼마나 영향을 받았는지 배경 이해에 도움될 것으로 예상.
