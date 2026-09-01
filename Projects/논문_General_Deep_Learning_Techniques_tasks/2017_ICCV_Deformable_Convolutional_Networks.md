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
source: "Projects/논문_pdf/General_Deep_Learning_Techniques/2017_ICCV_Deformable-Convolutional-Networks.pdf"
source_type: personal
createdAt: "2026-08-24T03:00:00.000Z"
updatedAt: "2026-08-24T03:00:00.000Z"
---

#paper #deformable-convolution #geometric-transformation #object-detection #semantic-segmentation #backbone-module

> [!quote] 원제
> **Deformable Convolutional Networks**
> Jifeng Dai, Haozhi Qi, Yuwen Xiong, Yi Li, Guodong Zhang, Han Hu, Yichen Wei — Microsoft Research Asia, ICCV 2017
> https://arxiv.org/abs/1703.06211

# 한 줄 요약
<mark style="background: #FFF3A3A6;">표준 convolution/RoI pooling의 고정된 정사각 grid 샘플링 위치에 학습 가능한 2D offset을 더해, 별도 supervision 없이 객체의 크기·형태에 맞춰 수용영역 자체가 입력 내용에 따라 적응적으로 변형되는 deformable convolution과 deformable RoI pooling을 제안한 논문.</mark>

> [!info] 내 메모
> 

# 정리

## 기존 방법의 한계
- **고정 기하 구조로 인한 변환 모델링 한계**:
  Convolution은 항상 고정된 위치에서 입력을 샘플링하고, pooling은 고정 비율로 공간을 축소하며, RoI pooling은 RoI를 고정 spatial bin으로 나눈다 — 기하학적 변환(형태·크기·시점·부분 변형)을 다룰 내부 메커니즘이 CNN 구조 자체에 없다. 같은 레이어의 모든 activation unit이 동일한 수용영역을 갖는 것도 같은 문제의 연장이다.
- **박스 기반 feature 추출의 한계**:
  Faster R-CNN, R-FCN 등 최신 detector도 여전히 직사각형 bounding box 기반으로 feature를 추출한다 — non-rigid(비강체) 객체에는 명백히 최적이 아니다.

## 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — 전역/정적 기하 변환 모델링**
- Spatial Transform Networks(STN)[26]: affine 변환 같은 전역 파라메트릭 변환을 데이터로부터 학습해 feature map을 warp — 비싼 feature warping과 어려운 변환 파라미터 학습이 필요해 semantic segmentation·object detection 같은 dense/semi-dense 예측 과제에는 사실상 적용되지 못함.
- Active Convolution[27]: convolution 샘플링 위치에 offset을 더해 end-to-end 학습 — 동시대 연구지만 offset이 모든 공간 위치에서 공유되는 정적(static) 모델 파라미터라 이미지 내용에 따라 달라지지 않음.
- Effective Receptive Field 분석[43] 및 atrous convolution[23]: 이론적 수용영역보다 실제 유효 수용영역이 훨씬 좁다는 분석에 기반해 dilation으로 수용영역을 넓히지만, dilation rate가 고정된 정적 하이퍼파라미터라는 한계는 그대로 남음.
- **타겟/해결**: 고정 기하 구조로 인한 변환 모델링 한계(문제①) — 방식은 달라도 전부 "샘플링 위치·수용영역"을 사람이 정하거나(고정 grid) 이미지 내용과 무관한 정적 파라미터(Active Convolution, atrous)로 둔다.

**갈래 2 — 파트 단위 변형 모델링**
- Deformable Part Models(DPM)[11]: 객체 파트 간 공간적 변형을 명시적으로 모델링 — 얕은 모델이라 표현력이 제한적이고, distance transform을 특수 pooling으로 CNN화해도[17] 컴포넌트·파트 크기 선택 같은 수작업 휴리스틱이 필요해 end-to-end가 아님.
- **타겟/해결**: 박스 기반 feature 추출의 한계(문제②) — 파트 단위 변형을 다루는 시도는 있었지만, 깊은 CNN 안에서 RoI feature 추출 자체를 end-to-end로 적응시키는 방법은 없었음.

**갭**: <mark style="background: #FFF3A3A6;">STN은 dense/semi-dense 예측에 부적합할 만큼 무겁고, Active Convolution의 offset은 이미지 내용과 무관한 정적 파라미터이며, atrous convolution의 dilation은 고정 하이퍼파라미터, DPM은 얕고 end-to-end가 아니다. "입력 내용에 따라 로컬·조밀(dense)하게, 그리고 end-to-end로" 샘플링 위치 자체를 동적으로 바꾸는 경량 메커니즘은 없었다.</mark>

## 이 논문이 풀고자 하는 문제
1. 별도 추가 supervision 없이, convolution의 샘플링 위치를 입력 feature 내용에 따라 동적으로 변형하는 것.
2. RoI pooling의 고정 spatial bin 분할도 객체 형태에 맞게 적응시키는 것.

**갭 종합**: <mark style="background: #FFF3A3A6;">STN은 dense/semi-dense 예측에 부적합할 만큼 무겁고, Active Convolution의 offset은 이미지 내용과 무관한 정적 파라미터이며, atrous convolution의 dilation은 고정 하이퍼파라미터, DPM은 얕고 end-to-end가 아니다. "입력 내용에 따라 로컬·조밀(dense)하게, 그리고 end-to-end로" 샘플링 위치 자체를 동적으로 바꾸는 경량 메커니즘은 없었다는 것이 이 논문의 통찰이다.</mark>

> [!info] 내 메모
> 

# 해결 방법 요약

| | 문제 ① — 고정 기하 구조로 인한 변환 모델링 한계 | 문제 ② — 박스 기반 feature 추출의 한계 |
|---|---|---|
| **해결 방법** | Convolution과 같은 spatial resolution·dilation을 갖는 별도 conv 레이어를 두어 각 grid 위치의 2D offset을 예측하고, 그 offset만큼 이동한(대개 분수 좌표인) 위치의 값을 bilinear interpolation으로 읽어 샘플링 위치 자체를 입력 내용에 따라 동적으로 바꾼다. | RoI pooling의 각 spatial bin 위치에도 같은 방식의 offset을 추가 — pooled feature에 fc 레이어를 적용해 정규화된 offset을 예측한 뒤 RoI 크기로 스케일링해 실제 offset으로 변환한다. |
| **예상되는 문제점** | Offset을 만드는 conv/fc 레이어가 추가되어 파라미터·연산량이 소폭 늘고(Table 4, 약 3~5%), offset이 데이터 통계에 과적합될 위험을 논문이 정량적으로 검증하지 않는다. | Offset 정규화 스칼라 γ(=0.1)처럼 사람이 정한 하이퍼파라미터가 여전히 남아 있고, 오프셋 학습이 명시적 supervision 없이 gradient만으로 이뤄지므로 학습된 offset이 항상 의미 있는 위치로 수렴한다는 보장이 없다(정성적 시각화로만 뒷받침됨). |

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">표준 convolution/RoI pooling의 각 샘플링 위치에 <span style="color:#c0392b; font-weight:bold;">2D offset</span>을 추가하고, 이 offset 자체를 입력 feature map으로부터 별도의 convolution/fc 레이어가 예측하도록 만들어, 오프셋이 이미지 내용에 따라 로컬·조밀·동적으로 결정되게 함으로써 별도 supervision 없이 표준 역전파만으로 end-to-end 학습되는 <span style="color:#c0392b; font-weight:bold;">deformable convolution</span>과 <span style="color:#c0392b; font-weight:bold;">deformable RoI pooling</span>을 만든다.</mark>

## 전체 파이프라인 (Fig. 2, Fig. 3, Fig. 4 기준)

```
[A] Deformable Convolution (Fig. 2)
입력 feature map (C, H, W)
       │
       ├──────────────────────────────┐
       ▼                               ▼
① Offset 예측 conv (같은 resolution/  ② 원래 conv 커널 (3×3, dilation 1)
   dilation의 별도 conv, kernel=3×3)
       │
       ▼
   offset field (2N, H, W)             [N=|R|=9, 즉 3×3 커널마다 (Δx,Δy) 9쌍 → 18채널]
       │
       ▼
③ Bilinear Interpolation Sampling     → 위치 p0+pn+Δpn (분수 좌표)의 값을 읽음
       │
       ▼
④ 가중합 (②의 커널 가중치 w(pn) 적용) → deformable convolution 출력 (C', H, W)


[B] Deformable RoI Pooling (Fig. 3)
입력 feature map (C, H, W) + RoI(w,h)
       │
       ▼
① 표준 RoI Pooling (k×k bin)          → pooled feature (k×k, C)
       │
       ▼
② fc layer                            → 정규화 offset Δp̂_ij (k×k, 2)
       │
       ▼
③ Δp_ij = γ·Δp̂_ij∘(w,h)  (γ=0.1)     → 실제 offset (RoI 크기로 스케일링)
       │
       ▼
④ Bilinear Interpolation Sampling + 재풀링  → deformable RoI pooling 출력 (k×k, C)


[C] Deformable PS(Position-Sensitive) RoI Pooling (Fig. 4, R-FCN용)
입력 feature map (C, H, W)
       │
       ├── top branch: conv → offset field (2k²(C+1), H, W) → PS RoI Pooling → 정규화 offset Δp̂_ij (k×k, 2)
       │
       └── bottom branch: conv → score maps (k²(C+1), H, W)
                                          │
                                          ▼ (top branch offset을 γ·(w,h)로 스케일해 적용)
                                   Deformable PS RoI Pooling
                                          │
                                          ▼
                              출력 per-RoI per-class score map (C+1, k, k)
```

> [!info] 내 메모
> 

### ① Deformable Convolution
- **역할**:
  같은 레이어의 모든 위치가 동일한 크기·형태의 수용영역을 갖는다는 CNN 구조 자체의 제약을 없애, 객체마다 다른 크기·형태에 맞춰 위치별로 수용영역이 스스로 조정되게 한다.
- **구현**:
  입력 feature map과 같은 spatial resolution·dilation을 갖는 별도 conv 레이어(커널 크기·dilation은 원래 conv와 동일)를 하나 더 두어, grid `R`의 각 샘플링 위치 `p_n`(n=1..N, N=|R|)마다 2D offset `Δp_n`을 출력한다. 출력 채널은 `2N`(예: 3×3 kernel → N=9 → 18채널) — [[1x1_Convolution]]과 달리 이 offset-conv는 원래 conv와 동일한 3×3/dilation 구조를 그대로 쓴다. Offset은 대개 분수(fractional) 좌표이므로 bilinear interpolation으로 값을 읽는다. Offset 예측 conv는 0 가중치로 초기화되어, 학습 초기에는 표준 convolution과 동일하게 시작한다. Deformable convolution은 마지막 몇 개 conv 레이어(kernel size>1)에만 선택 적용한다 — Table 1 ablation에서 3개 레이어가 여러 과제에 걸쳐 가장 좋은 trade-off로 확인됨.
- **입출력 shape**:
  `(C, H, W)` → offset field `(2N, H, W)` (N=커널 크기², 예: 3×3→18) → deformable conv 출력 `(C', H, W)` (공간 크기는 padding으로 유지, 표준 conv와 동일).

```python
# 논문 Eq.(1)-(4) 기반. R: 표준 grid(예: 3x3 dilation1 → {(-1,-1),...,(1,1)}), N=|R|
# 표준 convolution: y(p0) = Σ_{pn∈R} w(pn) · x(p0+pn)
def deformable_conv(x, offset_conv_weight, conv_weight):
    offsets = conv(x, offset_conv_weight)          # (C,H,W) -> (2N,H,W), 0으로 초기화된 채 학습 시작
    y = zeros_like_output()
    for p0 in spatial_positions(x):
        for pn in R:                                 # N개 grid 위치
            dpn = offsets[p0, pn]                     # 학습된 2D offset (fractional)
            sample = bilinear_interpolate(x, p0 + pn + dpn)   # Eq.(3)-(4)
            y[p0] += conv_weight[pn] * sample
    return y   # (C, H, W) -> (C', H, W)
```

<mark style="background: #FFF9D6A6;">동일 레이어의 모든 위치가 동일 수용영역을 갖는다는 "정리" 표 문제①을, 위치마다 다른 offset을 예측해 실질적인 수용영역·샘플링 형태를 입력 내용(객체의 크기·형태)에 맞춰 조정함으로써 직접 해소한다 — Table 2의 effective dilation 통계가 실제로 필터 크기가 객체 크기와 상관관계를 가짐을 정량적으로 뒷받침한다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> Offset을 만드는 conv 레이어가 추가되어 파라미터·연산량이 소폭 늘어난다(Table 4에서 Faster R-CNN 58.3M→59.9M, forward time 0.147s→0.192s). 또한 offset이 학습 데이터의 통계적 패턴에 과적합될 위험을 논문은 Fig. 5~7의 정성적 시각화로만 보이며, 정량적 일반화 검증(학습에 없던 극단적 형태의 객체 등)은 제시하지 않는다.

> [!info] 내 메모
> 

### ② Deformable RoI Pooling (+ Deformable PS RoI Pooling)
- **역할**:
  표준 RoI pooling이 RoI를 고정된 k×k grid bin으로 균등 분할하는 것과 달리, 각 bin의 위치 자체를 객체의 실제 전경(foreground) 영역으로 이동시켜 non-rigid 객체에서도 의미 있는 part-level feature를 뽑는다. R-FCN처럼 클래스별 score map을 쓰는 Position-Sensitive(PS) RoI pooling에도 동일한 원리를 확장해 deformable PS RoI pooling을 만든다.
- **구현**:
  표준 RoI pooling(Eq. 5, `y(i,j) = Σ x(p0+p)/n_ij`)의 각 bin `(i,j)`에 offset `Δp_ij`를 추가한다. Offset은 RoI pooling으로 얻은 pooled feature map에 fc 레이어를 적용해 정규화된 offset `Δp̂_ij`를 먼저 예측한 뒤, RoI의 width·height와 element-wise 곱하고 스칼라 `γ=0.1`로 추가 스케일링해 실제 offset `Δp_ij`로 변환한다(offset 크기를 RoI 크기에 불변하게 만들기 위함). Deformable PS RoI pooling(Fig. 4)은 "fully convolutional" 구조를 유지하기 위해, 입력을 두 branch로 나눠 top branch(conv)가 offset field(`2k²(C+1)` 채널)를 전체 해상도로 만들고, bottom branch(conv)가 클래스별 score map(`k²(C+1)` 채널)을 만든 뒤, top branch에서 나온 offset을 bottom branch의 PS RoI pooling에 적용해 최종 `(C+1)` 채널의 per-RoI per-class score map을 얻는다.
- **입출력 shape**:
  입력 feature `(C, H, W)` + RoI `(w, h)` → pooled feature `(k×k, C)` → fc → 정규화 offset `(k×k, 2)` → deformable RoI pooling 출력 `(k×k, C)`. PS 버전은 score map 채널이 `k²(C+1)`이고 출력은 `(C+1, k, k)`.

```python
# 논문 Eq.(5)-(6) 기반. gamma=0.1
def deformable_roi_pooling(x, roi, fc_weight, k=7):
    pooled = roi_pooling(x, roi, k)                        # (k,k,C), 표준 RoI pooling
    norm_offsets = fc(pooled.flatten(), fc_weight)          # (k*k, 2), 정규화된 offset
    w, h = roi.width, roi.height
    real_offsets = 0.1 * norm_offsets * [w, h]               # gamma=0.1로 RoI 크기에 맞춰 스케일
    y = zeros((k, k, C))
    for (i, j) in bins(k):
        for p in bin(i, j):
            sample = bilinear_interpolate(x, roi.p0 + p + real_offsets[i, j])
            y[i, j] += sample / n_ij
    return y   # (k, k, C)
```

<mark style="background: #FFF9D6A6;">"정리" 표 문제②(박스 기반 feature 추출이 non-rigid 객체에 최적이 아님)를, RoI의 각 spatial bin이 고정 격자에서 벗어나 실제 객체 전경(foreground) 영역으로 이동하도록 학습시켜 해소한다 — Figure 7에서 다양한 형태의 객체마다 bin이 실제 부분(part) 위치로 이동하는 정성적 근거를 보인다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> Offset 정규화(γ=0.1, RoI 크기로 스케일링)라는 사람이 정한 하이퍼파라미터가 여전히 필요하며, 부록 A는 이 정규화가 "RoI 크기에 무관하게 offset 학습이 일반화되도록" 역전파 관점에서 필수적임을 유도하지만, 이는 여전히 설계자가 미리 정한 스케일링 규칙에 의존한다는 뜻이기도 하다. 또한 학습된 offset이 정말 의미 있는 위치로 수렴했는지는 Fig. 6, 7의 정성적 시각화 외에 정량적 검증 수단이 논문에 제시되지 않는다.

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| ① Deformable Convolution | (C, H, W) | (C', H, W) | 위치별 적응적 수용영역 | 병렬 offset-conv(2N채널) + bilinear interpolation |
| ② Deformable RoI Pooling | (C, H, W) + RoI(w,h) | (k×k, C) | RoI bin을 객체 전경으로 이동 | RoI Pooling → fc → γ·offset·(w,h) 스케일 |
| ②' Deformable PS RoI Pooling | (C, H, W) | (C+1, k, k) | 클래스별 score map에 동일 원리 확장 | offset-conv branch + score-map branch (fully conv) |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table 5 (COCO test-dev, ResNet-101, mAP@[0.5:0.95])
**표를 보는 법**: 각 행 쌍이 (plain ConvNet, 굵게 표시된 Ours=deformable) 비교다 — 같은 backbone·같은 방법끼리 Before/After로 대응시켜 보면 된다.

| 방법 | Before(plain ConvNet) | After(Deformable ConvNet) |
|---|---|---|
| class-aware RPN | 23.2 | 25.8 (+11% 상대) |
| Faster R-CNN | 29.4 | 33.1 (+13% 상대) |
| R-FCN | 30.8 | 34.5 (+12% 상대) |

> [!note]- 세부 결과 및 Ablation
> #### Table 1 — Deformable convolution 레이어 수 (VOC 2007 test, ResNet-101)
> **보는 법**: 열이 레이어 수(0/1/2/3/6), 행이 과제(DeepLab mIoU, class-aware RPN/Faster R-CNN/R-FCN mAP@0.5) — 레이어가 늘수록 성능이 어디까지 오르는지 확인.
>
> | 레이어 수 | DeepLab mIoU@V | class-aware RPN mAP@0.5 | Faster R-CNN mAP@0.5 | R-FCN mAP@0.5 |
> |---|---|---|---|---|
> | 0(baseline) | 69.7 | 68.0 | 78.1 | 80.0 |
> | 1 | 73.9 | 73.5 | 78.6 | 80.6 |
> | 2 | 74.8 | 74.3 | 78.5 | 81.0 |
> | 3(default) | 75.2 | 74.5 | 78.6 | 81.4 |
> | 6 | 74.8 | 74.6 | 78.7 | 81.5 |
> - 3 레이어에서 DeepLab은 최고(75.2)를 찍고, 나머지 과제는 3 이후로도 소폭씩 더 오르지만 거의 saturate. 이후 실험은 3 레이어를 기본값으로 사용.
>
> #### Table 2 — Effective dilation 통계 (R-FCN, 3-deformable-layer, ResNet-101)
> **보는 법**: 행이 conv 레이어(res5a/b/c), 열이 객체 크기 카테고리(small/medium/large/background) — effective dilation(인접 샘플링 위치 간 평균 거리, 수용영역 크기의 대리 지표) 값이 객체 크기와 함께 커지는지 확인.
>
> | 레이어 | small | medium | large | background |
> |---|---|---|---|---|
> | res5c | 5.3±3.3 | 5.8±3.5 | 8.4±4.5 | 6.2±3.0 |
> | res5b | 2.5±1.3 | 3.1±1.5 | 5.1±2.5 | 3.2±1.1 |
> | res5a | 2.2±1.2 | 2.9±1.3 | 4.2±1.6 | 3.1±1.1 |
> - 필터 크기가 객체 크기와 명확히 상관관계 — deformation이 실제로 image content로부터 학습됨을 증명. background 영역 필터 크기는 medium~large 객체 사이 — 배경 인식에도 상당히 큰 수용영역이 필요함을 시사.
>
> #### Table 3 — Deformable convolution vs atrous convolution (ResNet-101)
> **보는 법**: atrous dilation을 2/4/6/8로 키워가며 deformable convolution·RoI pooling과 비교 — dilation을 계속 키워도 deformable을 넘는지 확인.
>
> | 방법 | DeepLab mIoU@V/@C | class-aware RPN mAP@0.5/@0.7 | Faster R-CNN mAP@0.5/@0.7 | R-FCN mAP@0.5/@0.7 |
> |---|---|---|---|---|
> | atrous(2,2,2) 기본 | 69.7/70.4 | 68.0/44.9 | 78.1/62.1 | 80.0/61.8 |
> | atrous(4,4,4) | 73.1/71.9 | 72.8/53.1 | 78.6/63.1 | 80.5/63.0 |
> | atrous(6,6,6) | 73.6/72.7 | 73.6/55.2 | 78.5/62.3 | 80.2/63.2 |
> | atrous(8,8,8) | 73.2/72.4 | 73.2/55.1 | 77.8/61.8 | 80.3/63.2 |
> | deformable convolution | **75.3/75.2** | **74.5/57.2** | 78.6/63.3 | 81.4/64.7 |
> | deformable conv + RoI pooling | N/A | N/A | **79.3/66.9** | **82.6/68.5** |
> - Dilation을 계속 키워도(4/6/8) deformable convolution을 넘지 못함 — 적응적 학습이 고정 dilation 탐색보다 근본적으로 우월함을 시사. Deformable RoI pooling까지 더하면(Faster R-CNN·R-FCN) mAP@0.7에서 특히 크게 개선.
>
> #### Table 4 — 모델 복잡도·런타임 (ResNet-101)
> **보는 법**: plain과 Ours(deformable) 쌍마다 파라미터·forward time을 비교 — 성능 향상 대비 비용 증가가 작은지 확인.
>
> | 방법 | # params | forward(sec) |
> |---|---|---|
> | DeepLab@C (plain / Ours) | 46.0M / 46.1M | 0.610 / 0.696 |
> | DeepLab@V (plain / Ours) | 46.0M / 46.1M | 0.084 / 0.088 |
> | class-aware RPN (plain / Ours) | 46.0M / 46.1M | 0.142 / 0.152 |
> | Faster R-CNN (plain / Ours) | 58.3M / 59.9M | 0.147 / 0.192 |
> | R-FCN (plain / Ours) | 47.1M / 49.5M | 0.143 / 0.169 |
> - 파라미터·연산량 증가가 미미(약 3~5%) — 성능 향상이 모델 용량 증가가 아니라 기하학적 변환 모델링 능력 자체에서 온다는 근거로 제시.
>
> #### Table 5 — COCO 전체 비교 (multi-scale testing/iterative bbox average 포함)
> **보는 법**: M(multi-scale testing)·B(iterative bbox average) 체크 여부별로 plain/Ours를 비교, 오른쪽 열들은 크기별 mAP(small/mid/large).
> Aligned-Inception-ResNet + R-FCN + M + B 조합에서 Ours가 mAP@[0.5:0.95] 37.5(plain 35.5) 달성, 소형 객체(mAP small) 19.4(plain 17.8)로 가장 큰 상대적 개선폭.

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- 오프셋이 학습 데이터의 통계적 패턴에 과적합될 위험 → <mark style="background: #FF5582A6;">논문은 이를 직접 검증하지 않으며, Figure 5·6·7의 정성적 시각화로만 "적응적으로 학습된다"는 것을 보인다 — 정량적 일반화 검증(예: 학습에 없던 극단적 형태의 객체)은 없음.</mark>
- 배경(background) 영역에도 medium~large 객체 수준의 큰 effective dilation이 학습됨(Table 2) → <mark style="background: #FF5582A6;">논문은 이를 "배경 인식에 큰 수용영역이 필요함을 시사"라고 해석할 뿐, 이것이 배경을 객체로 오인하는 리스크로 이어지는지는 논의하지 않는다.</mark>

### 한계
- <mark style="background: #FF5582A6;">DeepLab과 다른 과제(class-aware RPN, Faster R-CNN, R-FCN)의 최적 deformable layer 수·dilation 값이 서로 다르다(Table 1, 3) — 과제별 재튜닝이 필요해 "하나의 설정으로 모든 과제에 최적"은 아니다.</mark>
- <mark style="background: #FF5582A6;">Aligned-Inception-ResNet은 이 논문에서 처음 상세히 공개되지만, 이 백본 자체는 저자들이 아니라 별도로 명시된 "unpublished work"(Acknowledgements, Kaiming He 외)가 학습·검증한 것 — 이 백본의 기여는 이 논문의 독자적 성과가 아니다.</mark>
- Semantic segmentation·object detection 두 dense/semi-dense 예측 과제에서만 검증되었고, 논문 스스로 image classification 같은 과제에는 적용 우선순위를 두지 않음 — 어떤 과제 유형에서 효과가 클지/작을지에 대한 일반 이론은 제시되지 않는다.

### 생각할 점
- <mark style="background: #A6E3A1A6;">이 위키의 [[2020_ECCV_DETR|DETR]]이 이후 anchor·NMS라는 "수작업 구성요소"를 없앴다면, 이 논문은 그보다 3년 앞서 "고정된 샘플링 위치"라는 CNN의 또 다른 수작업 가정을 없앤 셈이다 — 두 논문 모두 "기존에 고정이라고 여겨지던 구조적 요소를 학습 가능하게 만든다"는 동일한 상위 전략을 공유한다.</mark>
- <mark style="background: #A6E3A1A6;">이후 처리할 Deformable DETR이 이 논문의 오프셋 예측 메커니즘을 attention의 sampling location에 적용한다면, "convolution의 고정 grid를 대체"에서 "attention의 dense global 연산을 대체"로 같은 아이디어의 적용 대상이 바뀐 것으로 볼 수 있다 — 실제로 [[Deformable_Sampling_Offset]] 개념 문서에 이미 이 계승 관계가 정리되어 있다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">Table 2에서 관찰된 "필터 크기가 객체 크기와 상관관계를 가진다"는 결과는, 이 위키의 [[2024_TGRS_ORFENet|ORFENet]]이 다루는 "receptive field별 중요도가 객체마다 다르다"는 문제의식과 본질적으로 같은 관찰이다 — ORFENet은 이를 명시적인 다중 receptive field 브랜치(MRFAFEM)로, 이 논문은 단일 conv의 샘플링 위치 자체를 변형시켜 암묵적으로 해결한다는 점에서 명시적/암묵적 접근의 대비가 흥미롭다.</mark>
- 소형 객체 탐지 관점에서, deformable convolution의 오프셋이 극소 크기 객체(수 픽셀)에서도 안정적으로 학습되는지는 이 논문에서 별도로 다루지 않는다 — 이후 읽을 소형 객체 특화 DETR 계열 논문들이 이 gap을 어떻게 다루는지 비교할 필요가 있다.

> [!info] 내 메모
> 

# 관련 개념
- [[Deformable_Sampling_Offset]] — 이 논문이 원조인 핵심 개념. 표준 convolution/RoI pooling의 고정 grid 샘플링 위치에 입력 조건부 학습된 offset을 더해 수용영역을 동적으로 변형하는 메커니즘. Deformable-DETR이 이 메커니즘을 attention으로 확장한 계승 관계까지 이미 이 개념 문서에 정리되어 있다.
- [[1x1_Convolution]] — offset 예측 branch·채널 조정에 쓰이는 기본 연산.
- [[Dilated_Convolution]] — "선행 연구 접근"에서 대조 대상인 atrous convolution의 정체, Table 3에서 deformable convolution과 직접 비교됨.

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]] — DETR과 마찬가지로 "기존 detector에 개입하는 방식" 비교축의 대상이 아니라, 이후 Deformable DETR 등이 계승하는 foundational 아키텍처 모듈로 별도 취급.

# 읽어볼 만한 논문
- 참고문헌 기반: M. Jaderberg, K. Simonyan, A. Zisserman, K. Kavukcuoglu, "Spatial transformer networks" (NeurIPS 2015) [26] — 이 논문이 가장 직접적으로 대조하는 선행 연구. 전역 파라메트릭 변환과 로컬·조밀 변환의 차이를 이해하는 데 필수.
- 참고문헌 기반: W. Luo, Y. Li, R. Urtasun, R. Zemel, "Understanding the effective receptive field in deep convolutional neural networks" (arXiv 2017) [43] — 이 논문이 atrous convolution의 필요성을 설명할 때 인용하는 effective receptive field 분석. Deformable convolution이 "왜" 필요한지의 이론적 배경.
- 참고문헌 기반: J. Dai, Y. Li, K. He, J. Sun, "R-FCN: Object detection via region-based fully convolutional networks" (NeurIPS 2016) [7] — 이 논문의 실험에서 deformable PS RoI pooling이 적용되는 핵심 baseline 아키텍처.
- 자유 추천(검증 필요): Deformable DETR — 이 논문의 오프셋 아이디어가 attention의 sparse sampling에 어떻게 적용되는지 직접 대조할 것. [[Deformable_Sampling_Offset]] 개념 문서에 이미 이 계승 관계가 요약되어 있으므로, 원문을 읽어 세부 diff를 검증할 가치가 있음. 검색 키워드: `Deformable DETR deformable attention module`.
