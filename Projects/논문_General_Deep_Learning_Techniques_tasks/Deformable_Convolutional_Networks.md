---
pm-task: true
projectId: "paperwiki-general-deep-learning-techniques"
parentId:
id: "t-deformable-conv-net-xt7usjieho"
title: "Deformable Convolutional Networks"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-08-20"
due:
progress: 0
assignees: []
tags: []
customFields:
  "2te7y2fdmtcnaks7": 2017
  "wys8uhhjmtcnaks9": "ICCV"
subtaskIds: []
dependencies: []
year: 2017
venue: "ICCV"
jcr_quartile: Q1
task: [general-deep-learning-techniques]
direction: [foundational]
paper_tags: [paper, deformable-convolution, geometric-transformation, object-detection, semantic-segmentation, backbone-module]
source: "/Users/GyeongSeo/Workspace/논문_pdf/Small_Object_Detection/2017_ICCV_Deformable-Convolutional-Networks.pdf"
createdAt: "2026-08-24T03:00:00.000Z"
updatedAt: "2026-08-24T03:00:00.000Z"
---

Project: [[논문_General_Deep_Learning_Techniques|General Deep Learning Techniques]]
#paper #deformable-convolution #geometric-transformation #object-detection #semantic-segmentation #backbone-module

# 한 줄 요약
<mark style="background: #FFF3A3A6;">표준 convolution/RoI pooling의 고정된 정사각 grid 샘플링 위치에 학습 가능한 2D offset을 더해, 별도 supervision 없이 객체의 크기·형태에 맞춰 수용영역 자체가 입력 내용에 따라 적응적으로 변형되는 deformable convolution과 deformable RoI pooling을 제안한 논문.</mark>

# 문제 정의

### 기존 방법의 한계
- **고정된 기하학적 구조**:
  Convolution은 항상 고정된 위치에서 입력을 샘플링하고, pooling은 고정된 비율로 공간을 축소하며, RoI pooling은 RoI를 고정된 spatial bin으로 나눈다 — 기하학적 변환(형태·크기·시점·부분 변형)을 다룰 내부 메커니즘이 CNN 구조 자체에 없다.
- **동일 레이어 내 모든 위치가 동일 수용영역**:
  같은 레이어의 모든 activation unit이 동일한 크기의 수용영역을 갖는다. 고수준 레이어일수록 위치마다 다른 스케일/형태의 semantic을 인코딩해야 하는데(예: semantic segmentation의 fine localization), 이를 위한 receptive field 크기의 위치별 적응적 결정 메커니즘이 없다.
- **박스 기반 feature 추출의 근본적 한계**:
  최신 object detection 방법들도 여전히 직사각형 bounding box 기반으로 feature를 추출한다 — non-rigid(비강체) 객체에는 명백히 최적이 아니다.
- **기존 대안들의 한계**:
  데이터 증강으로 변환 불변성을 학습하는 방법은 비싼 학습 비용과 복잡한 모델 파라미터를 요구하고, SIFT 같은 수작업 변환-불변 feature/알고리즘은 지나치게 복잡한 변환에는 설계 자체가 어렵거나 불가능하다.

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — 전역 파라메트릭 변환 학습**
- Spatial Transform Networks(STN) [26]: affine 변환처럼 전역 파라메트릭 변환을 데이터로부터 학습해 feature map을 warp — 비싼 feature warping과 어려운 변환 파라미터 학습이 필요해 semantic segmentation·object detection 같은 dense/semi-dense 예측 과제에는 사실상 적용되지 못함(STN은 소규모 이미지 분류에서만 성공 사례).

**갈래 2 — Convolution 자체의 샘플링 위치를 augment**
- Active Convolution [27]: convolution 샘플링 위치에 offset을 더하고 end-to-end로 학습 — deformable convolution과 동시대 연구지만, offset이 모든 공간 위치에서 공유되는 정적(static) 모델 파라미터라 이미지 내용에 따라 달라지지 않음.

**갈래 3 — 유효 수용영역/Atrous convolution**
- Effective Receptive Field 분석 [43]: 이론적 수용영역 중 실제로 기여하는 유효 수용영역은 Gaussian 분포를 따르며 이론값보다 훨씬 좁고, 레이어 수에 square-root로만 느리게 증가함을 규명 — 이는 atrous convolution이 널리 쓰이는 이유를 설명하지만, atrous convolution도 dilation rate가 고정된 정적 하이퍼파라미터라는 한계는 그대로 남음.

**갈래 4 — 변형 가능한 파트 모델**
- Deformable Part Models(DPM) [11]: 객체 파트 간 공간적 변형을 명시적으로 모델링 — 얕은 모델이라 표현력이 제한적이고, distance transform을 특수 pooling으로 변환해 CNN화해도[17] 컴포넌트·파트 크기 선택 같은 수작업 휴리스틱이 필요해 end-to-end가 아님.

**갭**: <mark style="background: #FFF3A3A6;">STN은 dense/semi-dense 예측에 부적합할 만큼 무겁고, Active Convolution의 offset은 이미지 내용과 무관한 정적 파라미터이며, atrous convolution의 dilation은 고정 하이퍼파라미터, DPM은 얕고 end-to-end가 아니다. "입력 내용에 따라 로컬·조밀(dense)하게, 그리고 end-to-end로" 샘플링 위치 자체를 동적으로 바꾸는 경량 메커니즘은 없었다.</mark>

### 이 논문이 풀고자 하는 문제
1. 별도의 추가 supervision 없이, convolution의 샘플링 위치를 입력 feature 내용에 따라 동적으로 변형하는 것
2. RoI pooling의 고정 spatial bin 분할도 마찬가지로 객체 형태에 맞게 적응시키는 것
3. 위 두 모듈을 기존 CNN 아키텍처(ResNet, Faster R-CNN, R-FCN, DeepLab 등)에 경량으로 이식 가능하게 만들어 end-to-end 학습되게 하는 것

# 제안 방법

<mark style="background: #FFF3A3A6;">표준 convolution/RoI pooling의 각 샘플링 위치에 2D offset을 추가하고, 이 offset 자체를 입력 feature map으로부터 별도의 convolution 레이어가 예측하도록 만든다 — offset이 이미지 내용에 따라 로컬·조밀·동적으로 결정되므로, 별도 supervision 없이 표준 역전파만으로 end-to-end 학습된다.</mark>

### ① Deformable Convolution
- 표준 3×3 convolution은 고정 grid `R = {(-1,-1), (-1,0), ..., (1,1)}`의 각 위치를 가중합.
- Deformable convolution은 같은 입력 feature map 위에 동일한 spatial resolution·dilation을 갖는 별도 convolution 레이어를 두어, 각 grid 위치마다 학습된 offset `Δp_n`을 출력(출력 채널 2N, N=|R|).
- 오프셋은 대개 분수(fractional) 값이므로 bilinear interpolation으로 그 위치의 feature 값을 계산.
- Offset을 만드는 conv 레이어와 원래 feature를 만드는 conv 레이어는 동시에 학습되며, 학습 초기 오프셋 예측 레이어는 0 가중치로 초기화(처음엔 표준 convolution과 동일하게 시작).

> [!example]- 구현 디테일
> ```
> y(p0) = Σ_{pn∈R} w(pn) · x(p0 + pn + Δpn)     # 표준: y(p0)=Σw(pn)·x(p0+pn)
> x(p) = Σ_q G(q,p) · x(q)                       # bilinear interpolation
> G(q,p) = g(qx,px)·g(qy,py),  g(a,b)=max(0,1-|a-b|)
> ```
> 추가된 conv/fc 레이어는 0으로 초기화, 학습률은 기존 레이어의 β배(기본 β=1, Faster R-CNN의 fc 레이어는 β=0.01). Deformable convolution은 마지막 몇 개 conv 레이어(커널 크기>1)에만 선택 적용 — 실험적으로 3개 레이어가 여러 과제에서 최적 trade-off.

<mark style="background: #FFF9D6A6;">동일 레이어의 모든 위치가 동일 수용영역을 갖는다는 "문제 정의"의 첫 문제를, 위치마다 다른 offset을 예측해 실질적인 수용영역·샘플링 형태를 입력 내용(객체의 크기·형태)에 맞춰 조정함으로써 직접 해소한다 — Table 2의 effective dilation 통계가 실제로 필터 크기가 객체 크기와 상관관계를 가짐을 정량적으로 뒷받침한다.</mark>

### ② Deformable RoI Pooling
- 표준 RoI pooling은 RoI를 k×k bin으로 나눠 각 bin의 평균을 취함(`y(i,j) = Σ x(p0+p)/n_ij`).
- Deformable RoI pooling은 각 bin 위치에도 offset `Δp_ij`를 추가 — RoI pooling으로 얻은 pooled feature map에 fc 레이어를 적용해 정규화된 offset을 예측한 뒤, RoI의 width·height로 element-wise 곱해 실제 offset으로 변환(오프셋 크기를 RoI 크기에 불변하게 만들기 위해 스칼라 γ=0.1로 추가 조정).
- Position-Sensitive(PS) RoI pooling에도 동일하게 확장(deformable PS RoI pooling) — 이땐 일반 feature map 대신 클래스별 score map에 offset을 적용.

> [!example]- 구현 디테일
> ```
> y(i,j) = Σ_{p∈bin(i,j)} x(p0+p+Δp_ij)/n_ij
> Δp_ij = γ · Δp̂_ij ∘ (w,h)     # γ=0.1, 정규화 offset을 RoI 크기로 스케일
> ```
> Offset 정규화(RoI 크기로 나눠 학습)가 RoI 크기에 무관하게 offset 학습이 일반화되도록 하는 데 필수적임을 부록 A에서 역전파 유도로 뒷받침.

<mark style="background: #FFF9D6A6;">"문제 정의"의 세 번째 문제(박스 기반 feature 추출이 non-rigid 객체에 최적이 아님)를, RoI의 각 spatial bin이 고정 격자에서 벗어나 실제 객체 전경(foreground) 영역으로 이동하도록 학습시켜 해소한다 — Figure 7에서 다양한 형태의 객체마다 bin이 실제 부분(part) 위치로 이동하는 정성적 근거를 보인다.</mark>

# 실험 결과

### 핵심 결과 (COCO test-dev, ResNet-101, mAP@[0.5:0.95])
| 방법 | Before(plain ConvNet) | After(Deformable ConvNet) |
|---|---|---|
| class-aware RPN | 23.2 | 25.8 (+11% 상대) |
| Faster R-CNN | 29.4 | 33.1 (+13% 상대) |
| R-FCN | 30.8 | 34.5 (+12% 상대) |

> [!note]- 세부 결과 및 Ablation
> #### Deformable convolution 레이어 수 (Table 1, VOC 2007 test, ResNet-101)
> | 레이어 수 | DeepLab mIoU@V | class-aware RPN mAP@0.5 | Faster R-CNN mAP@0.5 | R-FCN mAP@0.5 |
> |---|---|---|---|---|
> | 0(baseline) | 69.7 | 68.0 | 78.1 | 80.0 |
> | 1 | 73.9 | 73.5 | 78.6 | 80.6 |
> | 3(default) | 75.2 | 74.5 | 78.6 | 81.4 |
> | 6 | 74.8 | 74.6 | 78.7 | 81.5 |
> - 3 레이어에서 대부분 saturate, DeepLab만 3에서 최고. 이후 실험은 3 레이어를 기본값으로 사용.
>
> #### Effective dilation 통계 (Table 2, R-FCN 3-deformable-layer)
> | 레이어 | small | medium | large | background |
> |---|---|---|---|---|
> | res5c | 5.3±3.3 | 5.8±3.5 | 8.4±4.5 | 6.2±3.0 |
> | res5b | 2.5±1.3 | 3.1±1.5 | 5.1±2.5 | 3.2±1.1 |
> | res5a | 2.2±1.2 | 2.9±1.3 | 4.2±1.6 | 3.1±1.1 |
> - 필터 크기가 객체 크기와 명확히 상관관계 — deformation이 실제로 image content로부터 학습됨을 증명. Background 영역 필터 크기는 medium~large 객체 사이 — 배경 인식에도 상당히 큰 수용영역이 필요함을 시사.
>
> #### Deformable convolution vs atrous convolution (Table 3, ResNet-101)
> | 방법 | DeepLab mIoU@V/@C | class-aware RPN mAP@0.5/@0.7 | Faster R-CNN mAP@0.5/@0.7 | R-FCN mAP@0.5/@0.7 |
> |---|---|---|---|---|
> | atrous(2,2,2) 기본 | 69.7/70.4 | 68.0/44.9 | 78.1/62.1 | 80.0/61.8 |
> | atrous(8,8,8) | 73.2/72.4 | 73.2/55.1 | 77.8/61.8 | 80.3/63.2 |
> | deformable convolution | **75.3/75.2** | **74.5/57.2** | 78.6/63.3 | 81.4/64.7 |
> | deformable conv + RoI pooling | N/A | N/A | **79.3/66.9** | **82.6/68.5** |
> - Dilation을 계속 키워도(4/6/8) deformable convolution을 넘지 못함 — 적응적 학습이 고정 dilation 탐색보다 근본적으로 우월.
>
> #### 모델 복잡도·런타임 (Table 4)
> | 방법 | # params | forward(sec) |
> |---|---|---|
> | Faster R-CNN(plain) | 58.3M | 0.147 |
> | Faster R-CNN(deformable) | 59.9M | 0.192 |
> | R-FCN(plain) | 47.1M | 0.143 |
> | R-FCN(deformable) | 49.5M | 0.169 |
> - 파라미터·연산량 증가가 미미(약 3~5%) — 성능 향상이 모델 용량 증가가 아니라 기하학적 변환 모델링 능력 자체에서 온다는 근거로 제시.
>
> #### COCO 전체 비교(Table 5, multi-scale testing/iterative bbox average 포함)
> Aligned-Inception-ResNet + R-FCN + multi-scale + iterative bbox average 조합에서 mAP@[0.5:0.95] 37.5(plain 35.5) 달성, 소형 객체(mAP small) 19.4(plain 17.8)로 가장 큰 상대적 개선.

# Discussion

### 이 아이디어의 잠재적 부작용
- 오프셋이 학습 데이터의 통계적 패턴에 과적합될 위험 → <mark style="background: #FF5582A6;">논문은 이를 직접 검증하지 않으며, Figure 5·6·7의 정성적 시각화로만 "적응적으로 학습된다"는 것을 보인다 — 정량적 일반화 검증(예: 학습에 없던 극단적 형태의 객체)은 없음.</mark>
- 배경(background) 영역에도 medium~large 객체 수준의 큰 effective dilation이 학습됨(Table 2) → <mark style="background: #FF5582A6;">논문은 이를 "배경 인식에 큰 수용영역이 필요함을 시사"라고 해석할 뿐, 이것이 배경을 객체로 오인하는 리스크로 이어지는지는 논의하지 않는다.</mark>

### 한계
- <mark style="background: #FF5582A6;">DeepLab과 다른 과제(class-aware RPN, Faster R-CNN, R-FCN)의 최적 deformable layer 수·dilation 값이 서로 다르다(Table 1, 3) — 과제별 재튜닝이 필요해 "하나의 설정으로 모든 과제에 최적"은 아니다.</mark>
- <mark style="background: #FF5582A6;">Aligned-Inception-ResNet은 이 논문에서 처음 상세히 공개되지만, 원 모델(Aligned-Inception-ResNet)의 학습·설계 주체는 이 논문 저자가 아니라 별도로 명시된 "unpublished work"(Acknowledgements) — 이 백본 자체의 기여는 이 논문의 독자적 성과가 아니다.</mark>
- Semantic segmentation·object detection 두 dense/semi-dense 예측 과제에서만 검증되었고, 논문 스스로 image classification 같은 과제에는 적용 우선순위를 두지 않음 — 어떤 과제 유형에서 효과가 클지/작을지에 대한 일반 이론은 제시되지 않는다.

### 생각할 점
- <mark style="background: #A6E3A1A6;">이 위키의 [[DETR]]이 이후 anchor·NMS라는 "수작업 구성요소"를 없앴다면, 이 논문은 그보다 3년 앞서 "고정된 샘플링 위치"라는 CNN의 또 다른 수작업 가정을 없앤 셈이다 — 두 논문 모두 "기존에 고정이라고 여겨지던 구조적 요소를 학습 가능하게 만든다"는 동일한 상위 전략을 공유한다.</mark>
- <mark style="background: #A6E3A1A6;">이후 처리할 Deformable DETR이 이 논문의 오프셋 예측 메커니즘을 attention의 sampling location에 적용한다면, "convolution의 고정 grid를 대체"에서 "attention의 dense global 연산을 대체"로 같은 아이디어의 적용 대상이 바뀐 것으로 볼 수 있다 — 처리 후 실제 메커니즘을 확인해 이 노트에 교차 링크할 필요가 있다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">Table 2에서 관찰된 "필터 크기가 객체 크기와 상관관계를 가진다"는 결과는, 이 위키의 [[ORFENet]]이 다루는 "receptive field별 중요도가 객체마다 다르다"는 문제의식과 본질적으로 같은 관찰이다 — ORFENet은 이를 명시적인 다중 receptive field 브랜치(MRFAFEM)로, 이 논문은 단일 conv의 샘플링 위치 자체를 변형시켜 암묵적으로 해결한다는 점에서 명시적/암묵적 접근의 대비가 흥미롭다.</mark>
- 소형 객체 탐지 관점에서, deformable convolution의 오프셋이 극소 크기 객체(수 픽셀)에서도 안정적으로 학습되는지는 이 논문에서 별도로 다루지 않는다 — 이후 읽을 소형 객체 특화 DETR 계열 논문들이 이 gap을 어떻게 다루는지 비교할 필요가 있다.

# 관련 개념
- [[Deformable_Sampling_Offset]] — 이 논문의 핵심 기여. 표준 convolution/RoI pooling의 고정 grid 샘플링 위치에 입력 조건부 학습된 offset을 더해 수용영역을 동적으로 변형하는 메커니즘.

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]] — DETR과 마찬가지로 "기존 detector에 개입하는 방식" 비교축의 대상이 아니라, 이후 Deformable DETR 등이 계승하는 foundational 아키텍처 모듈로 별도 취급.

# 읽어볼 만한 논문
- 참고문헌 기반: M. Jaderberg, K. Simonyan, A. Zisserman, K. Kavukcuoglu, "Spatial transformer networks" (NeurIPS 2015) [26] — 이 논문이 가장 직접적으로 대조하는 선행 연구. 전역 파라메트릭 변환과 로컬·조밀 변환의 차이를 이해하는 데 필수.
- 참고문헌 기반: W. Luo, Y. Li, R. Urtasun, R. Zemel, "Understanding the effective receptive field in deep convolutional neural networks" (arXiv 2017) [43] — 이 논문이 atrous convolution의 필요성을 설명할 때 인용하는 effective receptive field 분석. Deformable convolution이 "왜" 필요한지의 이론적 배경.
- 참고문헌 기반: J. Dai, Y. Li, K. He, J. Sun, "R-FCN: Object detection via region-based fully convolutional networks" (NeurIPS 2016) [7] — 이 논문의 실험에서 deformable PS RoI pooling이 적용되는 핵심 baseline 아키텍처.
- 자유 추천(검증 필요): Deformable DETR — 이번에 바로 다음 순서로 처리할 예정이라 별도 검색 불필요. 이 논문의 오프셋 아이디어가 attention의 sparse sampling에 어떻게 적용되는지 직접 대조할 것.

---
**보안 참고**: PDF 전체를 확인했으며, 프롬프트 인젝션이나 지시문처럼 보이는 텍스트는 발견되지 않았다.
