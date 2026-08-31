---
pm-task: true
projectId: "paperwiki-small-object-detection"
parentId:
id: "t-fanet-aufqv4u9nn"
title: "FANet"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-07-01"
due:
progress: 0
assignees: []
tags: []
customFields:
  "7l8l795xmtcjvlf6": 2025
  "1frf59rymtcjvske": "MDPI"
subtaskIds: []
dependencies: []
year: 2025
venue: "Remote Sensing (MDPI)"
jcr_quartile: null
task: [small-object-detection]
direction: [improvement]
paper_tags: [paper, small-object-detection, remote-sensing, frequency-domain, attention, two-stage-detector, class-imbalance]
source: "/Users/GyeongSeo/Workspace/논문_pdf/Small_Object_Detection/2025_RemoteSensing_FANet.pdf"
createdAt: "2026-08-18T11:00:00.000Z"
updatedAt: "2026-08-28T00:00:00.000Z"
---

#paper #small-object-detection #remote-sensing #frequency-domain #attention #two-stage-detector #class-imbalance

> [!quote] 원제
> **FANet: Frequency-Aware Attention-Based Tiny-Object Detection in Remote Sensing Images**
> Zixiao Wen, Peifeng Li, Yuhan Liu, Jingming Chen, Xiantai Xiang, Yuan Li, Huixian Wang, Yongchao Zhao, Guangyao Zhou — Aerospace Information Research Institute, Chinese Academy of Sciences, Remote Sensing (MDPI) 2025

# 한 줄 요약
<mark style="background: #FFF3A3A6;">Faster R-CNN(+RFLA) 기반 원격탐사 tiny-object detector의 FPN 최하위 레벨(P2)과 RoI head에 각각 주파수 영역(2D-DFT/2D-DCT) 기반 plug-and-play attention 모듈(MSFFEM, CAREM)을 추가해 spatial 특징만으로는 부족한 tiny object의 경계·텍스처를 보강하고, 카테고리별 주파수 분포 차이를 정량 분석해 few-shot 카테고리 불균형을 다중 방향 flip 증강(SAS)으로 완화함으로써 AI-TOD에서 AP 20.6→24.8을 달성한 논문.</mark>

> [!info] 내 메모
> 

# 정리

| | 문제 ① — 약한 spatial 특징 | 문제 ② — 심한 class imbalance |
|---|---|---|
| **문제 정의** | 원격탐사 tiny object(16×16px 미만, AI-TOD 평균 12.8px)는 픽셀 수가 극히 적어 저대비·저해상도이고, bounding box regression의 작은 오차도 IoU를 크게 흔들 만큼 위치 오차에 민감하다. 조명·구름·촬영 고도·각도에 따라 같은 카테고리도 외형이 크게 달라지는 intra-class variation까지 겹친다(Figure 1, 같은 "ship" 카테고리가 촬영 조건에 따라 확연히 다르게 보임). | AI-TOD trainval에서 vehicle이 88.22%를 차지하는 반면 windmill·swimming pool은 각각 0.1% 미만일 만큼 카테고리 간 인스턴스 수 편차가 극단적이라, few-shot 카테고리의 특징이 실제로 학습되는지조차 의심스럽다. |
| **풀고자 하는 문제** | Spatial 특징만으로는 표현이 부족한 tiny object의 contour/texture를 주파수 영역 정보로 보완해 배경 노이즈를 억제하고 판별력을 높인다. RoI 단위에서도 고주파 응답을 활용해 위치 추정·분류 정확도를 추가로 개선한다. | 카테고리 간 극심한 샘플 불균형으로 인한 few-shot 카테고리의 검출 성능 저하를 완화한다. |
| **선행 연구 접근** | - 멀티스케일 융합: PANet, DetectoRS, BAFNet, FSANet — 타일 분할 전처리 자체가 tiny-object 특징을 손상시키는 문제는 대부분 간과<br>- Context/attention 기반: non-local, DETR 계열, MENet, CFENet, FFCA-YOLO, DQ-DETR — 여전히 spatial-domain feature에 의존, 내재적으로 약한 특징 자체는 그대로<br>- Label assignment 개선: NWD, RFLA — positive sample 품질·recall은 개선하나 "특징 자체가 약하다"는 근본 문제는 미해결<br>- Frequency-domain 갈래: DFT/wavelet 고전 기법, CNN+Fourier 결합(camouflaged object detection·segmentation 등), 학습형 주파수 필터(SpectFormer, HS-FPN) — 방향은 유사하나 범용 목적이거나 원격탐사 tiny object에 미특화, feature map·RoI 레벨을 동시에 다루지 않음 | - Super-resolution·GAN 합성·diffusion 기반(SVDDD) 샘플 증강, UniFusOD의 멀티모달 융합 — 연산 복잡도 증가, 일반화 부족<br>- **갭**: 카테고리별 주파수 특성 차이를 정량적으로 분석해 증강 대상·배수를 근거 있게 정하는 접근은 없었음 |
| **해결 방법** | MSFFEM(feature map 레벨, P2에 패치 단위 2D-DFT + 학습 가능한 주파수 가중치)과 CAREM(RoI 레벨, 2D-DCT 고주파 필터 + 채널 attention)을 동시에 적용해 두 지점에서 주파수 정보를 활용 | 카테고리별 patch에 2D-DFT를 적용한 로그 파워 스펙트럼 방사평균 R_log(k)로 카테고리 간 주파수 분포가 실제로 다름을 정량 확인한 뒤, few-shot 카테고리는 다중 방향(수평/수직/대각) flip으로 최대 8배 증강하고 지배 카테고리(vehicle)는 샘플 일부를 제거(SAS) |
| **예상되는 문제점** | 학습된 주파수 가중치는 배경/객체가 특정 주파수 대역으로 뚜렷이 분리되는 상황을 전제하므로, 이 가정이 깨지는 장면(반복적 텍스처 배경, 안개·저조도)에서는 역효과 가능성(아래 "제안 방법" ③ MSFFEM 참고). | flip 증강은 카테고리 내부의 다양성(예: 촬영 각도·조도 변화)을 새로 만들어내지 못하고 기존 샘플을 거울상으로 반복하는 것에 그친다(아래 "Discussion" 참고). |

**갭 종합**: <mark style="background: #FFF3A3A6;">두 문제는 서로 달라 보이지만, 둘 다 "spatial-domain(픽셀 배치) 정보만으로 tiny object를 다루려 한다"는 공통 한계에서 나온다 — 약한 spatial feature 문제는 물론이고, class imbalance조차 spatial 통계(인스턴스 개수)만으로 다뤄왔다. 이 논문은 주파수 영역이라는 별도 정보원(feature map·RoI 레벨의 주파수 응답, 카테고리별 주파수 분포)을 도입해 두 문제를 하나의 틀로 동시에 공략한다.</mark>

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">주파수 영역 변환(2D-DFT/2D-DCT)은 정보 손실이 없는 가역 변환이므로, 고주파(경계·텍스처)와 저주파(배경)가 자연스럽게 분리되는 성질을 이용해 <span style="color:#c0392b; font-weight:bold;">학습 가능한 주파수 가중치로 특정 대역을 선택적으로 강조/억제</span>한다 — feature map 레벨(MSFFEM)과 RoI 레벨(CAREM) 두 곳에 plug-and-play로 삽입.</mark>

## 전체 파이프라인 (Fig. 2 기준)

```
입력 이미지
       │
       ▼
① ResNet-50 Backbone              → C2, C3, C4, C5 (다중 스케일 feature)
       │
       ▼
② FPN (top-down 융합)               → P2, P3, P4, P5(, P6)   [P2: (C, 200, 200), AI-TOD 800×800 입력 기준]
       │
       ▼ (P2만)
③ MSFFEM (patch 단위 2D-DFT 기반 주파수 강화)  → P2^fused (C, 200, 200)   [P3~P6은 그대로 통과]
       │
       ▼
④ RPN ({P2^fused, P3, P4, P5, P6} 각 레벨)   → proposal 박스들
       │
       ▼
⑤ RoI-Align                        → RoI feature (C, 7, 7) × N_roi
       │
       ▼
⑥ CAREM (2D-DCT 고주파 필터 + 채널 attention)  → 강화된 RoI feature (C, 7, 7)
       │
       ▼
⑦ Detection Head (FC_shared → FC_cls, FC_reg)  → 클래스 확률 + 박스 좌표
       │
       ▼ (학습 시)
⑧ Loss: L_rpn_cls + λL_rpn_reg + γ(L_head_cls + λL_head_reg)
```

> [!info] 내 메모
> 

### ③ MSFFEM (Multi-Scale Frequency Feature Enhancement Module)
- **역할**: FPN이 만든 여러 레벨의 feature map 중, tiny object가 대부분 매핑되는 최하위 레벨(P2, 가장 해상도가 높고 디테일이 살아있는 레벨)만을 대상으로, 주파수 영역에서 "이 객체의 경계·텍스처에 해당하는 대역"을 학습해 선택적으로 증폭하고 배경에 해당하는 저주파 성분은 상대적으로 억제한다. P3~P6은 건드리지 않는다 — ablation(Table 5)에서 P2·P3 모두에 적용해도 이득이 없었기 때문(대부분의 tiny object가 P2에만 매핑되므로).
- **구현**: 내부는 여러 개의 **FFEM(Frequency Feature Enhancement Module) 분기**로 구성되고, 각 분기가 서로 다른 patch 크기(patch size)로 다음 서브모듈을 수행한다.
  1. **PRM(Patch Rearrange Module)**: `(B, C, H, W)` feature map을 `ps×ps` 크기의 겹치지 않는 patch로 분할해 `(B·N_h·N_w, C, ps, ps)`로 재배열한다(`N_h, N_w`는 세로·가로 방향 patch 개수). Patch size는 데이터셋별로 다르게 설정(AI-TOD: [50, 100], VisDrone2019: [(24,32), (48,64)], DOTA-v1.5: [64, 128]) — 너무 크면 tiny object가 배경 노이즈에 묻히고, 너무 작으면 객체가 잘려 aliasing이 생기기 때문(ResNet 수용영역 계산상 2~32px 객체가 P2에서 약 20~50px 수용영역을 가짐).
  2. **2D-DFT**: 각 patch에 2D 이산 푸리에 변환을 적용해 주파수 영역 표현 `F(u,v)`를 얻는다.
  3. **적응형 주파수 가중치 학습**: 여러 채널의 스펙트럼을 1×1 conv([[1x1_Convolution]])로 1개 채널로 합친 뒤, patch 차원에 대해 합산하고 sigmoid를 통과시켜 가중치 행렬 `W(u,v)`를 얻는다. 이 `W(u,v)`가 "어떤 주파수 성분을 얼마나 강조할지"를 데이터로부터 학습한 필터 역할을 한다.
  4. **가중치 적용 + 2D-IDFT + IPRM**: `F(u,v)`에 `W(u,v)`를 곱한 뒤 역푸리에 변환(2D-IDFT)으로 공간 영역으로 복원하고, IPRM(PRM의 역연산)으로 원래 feature map 배치로 되돌린다.
  5. 원본 feature map과 잔차 연결(residual)로 더해 FFEM 출력을 만든다.
  6. 서로 다른 patch size의 FFEM 분기 출력들을 채널 방향으로 concat → channel shuffle → group convolution으로 융합해 최종 MSFFEM 출력을 만든다. 여러 patch size를 병렬로 두는 이유는 patch size마다 유리한 객체 스케일이 다르기 때문(Table 4: patch 10/25는 매우 작은 객체(APvt)에, patch 100은 중간 크기 객체(APm)에 유리) — intra-class variation을 포함한 다양한 객체 크기에 동시 대응하기 위함.
- **입출력 shape**: `P2 (C, H, W)` → PRM → `(N_patch, C, ps, ps)` → 2D-DFT/가중치/2D-IDFT/IPRM → `(C, H, W)` 복원 → 잔차합 → 여러 분기 concat+GC → `P2^fused (C, H, W)` (shape 자체는 patch 분해·복원 과정에서 원래 크기로 돌아오므로 P2와 동일).

```python
# 논문 Eq.(1)-(9) 기반 의사코드, 분기 하나(patch size = ps)를 기준으로 작성
p = PRM(P2, ps)                       # (B,C,H,W) -> (B*Nh*Nw, C, ps, ps)
F_uv = DFT2D(p)                       # 공간 -> 주파수 영역, shape 동일
fused = sum(Conv1x1_C_to_1(F_uv))     # 채널 C -> 1로 합친 뒤 patch 차원 합산
W_uv = sigmoid(fused)                 # 적응형 주파수 가중치, (ps, ps)
F_hat = F_uv * W_uv                   # 주파수 대역별 선택적 강조/억제
p_hat = IDFT2D(F_hat)                 # 주파수 -> 공간 영역 복원
P_hat = IPRM(p_hat)                   # patch -> 원래 feature map 배치로 복원
FFEM_out = P2 + P_hat                 # 잔차 연결

# 여러 patch size(ps_1, ps_2, ...) 분기를 병렬 실행 후 융합
branches = [FFEM(P2, ps_i) for ps_i in patch_sizes]      # 각 (C, H, W)
concat = channel_concat(branches)                         # (C * n_branches, H, W)
shuffled = channel_shuffle(concat)
P2_fused = group_conv_1x1(shuffled)                        # (C, H, W)
```

<mark style="background: #FFF9D6A6;">tiny object는 patch 단위 FFT 스펙트럼에서 다른 영역과 뚜렷이 구분되는 고유한 주파수 응답을 보인다(Figure 3). 학습 가능한 주파수 가중치는 이 차이를 직접 이용해 객체 대역만 선택적으로 증폭하므로, spatial-domain 강화보다 더 명시적으로 "약한 spatial 특징" 문제를 겨냥한다 — Table 3에서 FFEM 단독으로도 baseline 대비 AP 20.6→21.2, multi-scale 융합까지 더하면 스케일별 성능이 고르게 개선됨을 확인.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> "고주파=경계·텍스처, 저주파=배경"이라는 가정에 의존하므로, 이 가정이 깨지는 상황(반복적 텍스처를 가진 도심·암반 배경, 안개·저조도로 고주파 성분 자체가 거의 없는 영상)에서는 역효과가 날 수 있다 — 논문은 이런 실패 조건을 별도로 분석하지 않는다(Discussion 참고).

> [!info] 내 메모
> 

### ⑥ CAREM (Channel Attention-based RoI Enhancement Module)
- **역할**: MSFFEM이 feature map 전체 단위로 배경을 억제했다면, CAREM은 RoI-Align으로 잘라낸 개별 RoI feature(7×7 크기) 안에서 한 번 더 "어떤 채널이 tiny object를 잘 대변하는지"를 골라내, 최종 분류·박스 회귀 직전 단계에서 위치 추정·분류 정확도를 직접 강화한다.
- **구현**:
  1. **고주파 필터 추출**: RoI feature `x`에 2D-DCT를 적용하고, Gaussian 가중 필터 `z(u,v) = exp(-d_f²/2σ²)`(`d_f`: 주파수 좌표에서 고주파 코너까지의 유클리드 거리, σ=2로 설정)를 곱한 뒤 2D-IDCT로 복원해 고주파 응답 `x_high`를 얻는다. Gaussian 필터는 저주파 정보를 완전히 버리지 않고 일부 남기는 방식이라, 저주파를 통째로 0으로 만드는 이진(0-1) 필터보다 텍스처 손실이 적어 우수함을 ablation(Table 6)으로 확인했다.
  2. **채널 압축**: `x_high`에 Global Max Pooling(GMP)과 Global Average Pooling(GAP)을 각각 적용해 채널별 스칼라 벡터 `maxp`, `avgp`를 얻는다.
  3. **채널 attention 가중치**: `maxp`, `avgp` 각각을 1×1 conv 두 개에 통과시킨 뒤 합산하고 sigmoid를 적용해 채널 가중치 `W_c`를 얻는다 — squeeze-and-excitation 계열 채널 attention과 동일한 구조([[Squeeze_And_Excitation_Channel_Attention]] 참고).
  4. 원본 RoI feature `x`에 `W_c`를 곱해 채널 선택적으로 강화된 `x̂`를 얻는다.
- **입출력 shape**: RoI feature `(C, 7, 7)` → 고주파 필터링 `x_high (C, 7, 7)` → GMP/GAP `(C,)` × 2 → 1×1 conv + sigmoid → `W_c (C,)` → `x̂ = x · W_c`, shape `(C, 7, 7)` 그대로.

```python
# 논문 Eq.(10)-(15) 기반 의사코드
F_x = DCT2D(x)                       # RoI feature -> 주파수 영역
z = gaussian_high_freq_filter(sigma=2)  # z(u,v) = exp(-d_f^2 / (2*sigma^2))
x_high = IDCT2D(z * F_x)             # 고주파 응답만 복원, shape 동일 (C, 7, 7)

maxp = global_max_pool(x_high)       # (C,)
avgp = global_avg_pool(x_high)       # (C,)
W_c = sigmoid(conv1(conv2(maxp)) + conv1(conv2(avgp)))  # (C,), squeeze-and-excitation 구조

x_hat = x * W_c[:, None, None]       # 채널별 스케일링, (C, 7, 7)
```

<mark style="background: #FFF9D6A6;">tiny object는 RoI 안에서 차지하는 비중이 feature map 전체에서보다도 더 작다. CAREM은 RoI 안에 이미 섞여 들어온 배경/저주파 성분을 한 번 더 걸러내 "어떤 채널이 tiny object를 잘 대변하는가"를 학습함으로써, 검출 파이프라인 마지막 단계에서 위치 추정·분류 정확도를 직접 겨냥한다 — MSFFEM 단독(AP50 +0.6%p, Table 9)보다 CAREM 단독(+1.2%p)의 효과가 더 크고, 둘을 합치면 +2.0%p로 상호 보완적이다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> CAREM은 RPN이 생성한 RoI마다 DCT/IDCT와 채널 attention을 반복 계산해야 해서, 추론 속도가 프레임워크 전반에서 9~19% 감소한다(RFLA 기준 FPS 42.5→38.7) — 논문 스스로 "추가 지연시간 대부분이 MSFFEM이 아니라 CAREM의 RPN 의존적 RoI 처리에서 발생한다"고 명시(§5.1.1). 실시간 응용에서는 트레이드오프 고려가 필요하다.

> [!info] 내 메모
> 

### SAS (Sample Augmentation Strategy, 학습 데이터 전처리 — forward 파이프라인 밖)
- **역할**: "샘플이 적다"는 표면적 현상 아래에, 지배 카테고리(vehicle)에 학습이 쏠려 few-shot 카테고리(windmill, swimming pool 등)의 특징이 애초에 제대로 학습되는지조차 확인되지 않는다는 문제를 다룬다. 카테고리별 주파수 분포가 실제로 다름을 먼저 정량적으로 확인한 뒤, 그 근거 위에서 증강·축소 대상을 정한다.
- **구현**:
  1. **카테고리별 주파수 분포 분석**: 카테고리별 patch에 2D Hann window(경계 효과 억제)를 적용 → 2D-DFT → 로그 파워 스펙트럼을 반지름 방향으로 평균 낸 `R_log(k) = (1/N_k) Σ_{(u,v)∈S_k} log(|F(u,v)|² + 1)`을 계산한다(`S_k`: 반지름 k인 주파수 점들의 집합). Figure 6에서 카테고리마다 이 곡선이 실제로 다르게 나타남을 확인.
  2. **Few-shot 카테고리 증강**: 인스턴스 수에 반비례해 수평/수직/대각 flip을 인스턴스 수에 따라 최대 8배까지 적용(Table 1: airplane/bridge ×4, storage tank ×2, swimming pool/windmill ×8 — vehicle/ship/person은 이미 샘플이 충분해 ×1).
  3. **지배 카테고리 축소**: vehicle 이미지를 무작위로 일부 제거해(369k→163k 인스턴스까지 실험) 중복 샘플을 줄인다.
- **입출력**: 학습 데이터셋(이미지+annotation) 수준에서 작동하는 전처리이므로, 모델 구조에는 shape 변화가 없다 — 카테고리별 인스턴스 비율만 바뀐다(Table 1: 예를 들어 windmill 0.08%→0.45%, vehicle 88.22%→84.85%).

```python
# 의사코드 — 학습 데이터 구성 단계에서 수행, 모델 forward와 무관
Rlog = {c: radial_log_power_spectrum(hann_window(patches[c])) for c in categories}  # Figure 6 근거
for c in few_shot_categories:            # airplane, bridge, storage tank, swimming pool, windmill
    multiply_by_flips(c, multiple=instance_count_inverse(c))   # 수평/수직/대각 flip, 최대 x8
for c in [dominant_category]:            # vehicle
    randomly_drop_samples(c, target_ratio=r)  # r ∈ {100%, 70%, 50%, 20%} 실험 (Table 8)
```

<mark style="background: #FFF9D6A6;">class imbalance는 단순히 "샘플이 적다"가 아니라 "지배 카테고리에 학습이 쏠려 few-shot 카테고리 특징이 애초에 제대로 학습되는지 알 수 없다"는 문제였다. R_log(k) 분석으로 카테고리 간 주파수 특성이 실제로 다름을 확인했기에, vehicle 샘플을 줄여도(369k→163k) 다른 카테고리 성능 저하 없이 개선이 가능함을 사전에 근거 있게 예상했고 실제로도 확인됐다(Table 8: 100%→20%로 축소해도 AI/BR/SH/PE 성능 유지, VE만 24.6→21.3으로 하락).</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> Flip 증강은 기존 샘플을 거울상으로 반복 생성할 뿐, 촬영 각도·조도 등 실제 intra-class variation(문제 정의 ①)을 새로 만들어내지는 못한다 — "정리" 표의 intra-class variation 문제 자체는 SAS로 해결되지 않는다.

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| ① ResNet-50 Backbone | 입력 이미지 | C2, C3, C4, C5 | 다중 스케일 semantic feature 추출 | ImageNet 사전학습 ResNet-50 |
| ② FPN | C2~C5 | P2~P6 | top-down 멀티스케일 feature 융합 | 표준 FPN |
| ③ MSFFEM | P2 (C, H, W) | P2^fused (C, H, W) | 주파수 영역에서 tiny object 경계·텍스처 강조, 배경 억제 | 다중 patch-size FFEM 분기(PRM+2D-DFT+적응형 가중치+2D-IDFT+IPRM) + channel shuffle + group conv |
| ④ RPN | P2^fused, P3~P6 | proposal 박스 | 객체 후보 영역 생성 | 표준 RPN(모든 FPN 레벨) |
| ⑤ RoI-Align | proposal + feature map | RoI feature (C, 7, 7) | 고정 크기 RoI feature 추출 | 표준 RoI-Align |
| ⑥ CAREM | RoI feature (C, 7, 7) | 강화 RoI feature (C, 7, 7) | RoI 내 고주파 채널 선택적 강화 | 2D-DCT Gaussian 고주파 필터 + [[Squeeze_And_Excitation_Channel_Attention]] |
| ⑦ Detection Head | RoI feature (C, 7, 7) | 클래스 확률 + 박스 좌표 | 최종 분류·회귀 | FC_shared → FC_cls, FC_reg |
| (전처리) SAS | 학습 데이터셋 | 재구성된 학습 데이터셋 | few-shot 카테고리 증강 + 지배 카테고리 축소 | 다중방향 flip + 무작위 샘플 제거 |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table 11 (AI-TOD test set)
**표를 보는 법**: 각 행이 하나의 detector이고, `Ours`(FANet, SAS 미적용)와 `FANet‡`(SAS 포함)를 앞선 anchor-based/anchor-free/state-of-the-art 방법들과 같은 backbone(ResNet-50)·같은 12-epoch 학습으로 비교하면 된다 — baseline인 "Faster R-CNN w/ RFLA"와 직접 비교가 핵심.

| 벤치마크 | 지표 | Faster R-CNN w/ RFLA (baseline) | FANet (SAS 미적용) | FANet‡ (SAS 포함) |
|---|---|---|---|---|
| AI-TOD test | AP / AP50 | 20.6 / 50.4 | 21.4 / 52.4 | 24.8 / 58.1 |
| AI-TOD test | APvt (2~8px) | 7.0 | 8.2 | 10.7 |

> [!note]- 세부 결과 및 Ablation
> #### 설정
> - 데이터셋: AI-TOD(8클래스, 평균 객체 12.8px, 800×800, 11,214 train/2,804 val/14,018 test), VisDrone2019(UAV, 10클래스, 6,471 train/548 val/1,610 test-dev, 약 2000×1500), DOTA-v1.5(항공영상, 16클래스, 2,806 images, sliding-window crop, horizontal box 기준 학습·평가)
> - 지표: COCO 스타일 AP/AP50/AP75, AI-TOD 크기별 AP(APvt[2,8px]/APt[8,16px]/APs[16,32px]/APm[32px+])
> - Baseline: Faster R-CNN w/ RFLA(ResNet-50) — RFLA가 이미 tiny object에 유리한 label assignment를 제공해 공정 비교 위해 채택
> - 학습: MMDetection, ResNet-50(ImageNet 사전학습), SGD(momentum 0.9, wd 1e-4), batch 2, 12 epoch, lr 0.005(8·11 epoch에서 decay), RPN proposal 3000개, 단일 RTX 4090, seed 2025
>
> #### Table 2 — 다른 프레임워크로의 일반화 (AI-TOD test, MSFFEM+CAREM 결합 = "+Ours")
> **보는 법**: 4개 서로 다른 baseline(Faster R-CNN, NWD-RKA, RFLA, Cascade R-CNN) 각각에 "+Ours"를 얹었을 때 AP50이 일관되게 오르는지, FLOPs/파라미터 증가가 무시할 만한지 확인.
>
> | Method | AP | AP50 | AP75 | FLOPs | #Params | FPS |
> |---|---|---|---|---|---|---|
> | Faster R-CNN → +Ours | 11.1→12.1 | 26.3→27.9 | 7.6→8.7 | 134.42G→134.65G | 41.16M→41.17M | 45.9→37.3 |
> | NWD-RKA → +Ours | 18.8→20.1 | 47.5→50.1 | 11.1→12.0 | 〃 | 〃 | 40.7→38.9 |
> | RFLA → +Ours | 20.6→21.4 | 50.4→52.4 | 12.9→13.4 | 〃 | 〃 | 42.5→38.7 |
> | Cascade R-CNN → +Ours | 13.7→13.9 | 30.6→31.3 | 10.0→10.5 | 162.21G→162.51G | 68.95M→68.97M | 33.7→27.2 |
>
> 파라미터·FLOPs 증가는 미미(0.01~0.3M, 0.2~0.3G)하지만 FPS는 9~19% 감소 — 대부분 CAREM의 RoI 단위 반복 연산에서 발생.
>
> #### Table 9 — MSFFEM/CAREM/SAS 조합 ablation (AI-TOD test)
> **보는 법**: 체크(✓) 조합별 행을 비교 — 두 feature 강화 모듈(MSFFEM·CAREM)이 개별로도, 결합해도 이득이 있는지, SAS가 추가로 얼마나 기여하는지 확인.
>
> | MSFFEM | CAREM | SAS | AP | AP50 | AP75 | APvt |
> |---|---|---|---|---|---|---|
> | - | - | - | 20.6 | 50.4 | 12.9 | 7.0 |
> | ✓ | - | - | 21.0 | 51.0 | 13.5 | 8.9 |
> | - | ✓ | - | 21.4 | 51.6 | 13.8 | 8.0 |
> | ✓ | ✓ | - | 21.4 | 52.4 | 13.7 | 8.2 |
> | - | - | ✓ | 24.4 | 58.1 | 16.7 | 9.0 |
> | ✓ | ✓ | ✓ | 24.8 | 58.1 | 17.5 | 10.7 |
>
> #### Table 3 — FFEM 단일 patch(50) vs Multi-Scale(MS) 융합
> **보는 법**: FFEM 하나만 켰을 때와, 여러 patch size 분기를 MS로 융합했을 때 스케일별(APvt/APt/APs/APm) 성능이 어떻게 갈리는지 비교.
>
> | FFEM | MS | AP | AP50 | AP75 | APvt | APt | APs | APm |
> |---|---|---|---|---|---|---|---|---|
> | - | - | 20.6 | 50.4 | 12.9 | 7.0 | 20.8 | 25.7 | 32.1 |
> | ✓ | - | **21.2** | **51.8** | 13.5 | 8.0 | **21.5** | 26.0 | 32.5 |
> | ✓ | ✓ | 21.0 | 51.0 | 13.5 | **8.9** | 21.3 | **26.3** | **33.1** |
>
> #### Table 4 — MSFFEM patch size 영향 (AI-TOD, P2 크기 200×200)
> **보는 법**: patch size가 작을수록 APvt(매우 작은 객체)에, 클수록 APm(중간 크기 객체)에 유리한 trade-off를 확인. Patch 50이 전체 AP·AP50 최적.
>
> | Patch | AP | AP50 | AP75 | APvt | APt | APs | APm |
> |---|---|---|---|---|---|---|---|
> | - | 20.6 | 50.4 | 12.9 | 7.0 | 20.8 | 25.7 | 32.1 |
> | 10 | 20.4 | 50.5 | 12.7 | 8.8 | 20.4 | 25.4 | 32.6 |
> | 25 | 20.6 | 50.2 | 13.0 | **9.1** | 20.7 | 25.4 | 32.7 |
> | 50 | **21.2** | **51.8** | **13.5** | 8.0 | **21.5** | 26.0 | 32.5 |
> | 100 | 21.0 | 51.2 | 13.2 | 7.6 | 21.1 | 26.0 | **32.9** |
>
> #### Table 5 — 주파수 강화를 적용할 FPN 레벨
> **보는 법**: P2에만 적용한 경우와 {P2, P3} 둘 다 적용한 경우를 비교 — 여러 레벨에 적용한다고 항상 이득이 아님을 확인(대부분 tiny object가 P2에만 매핑되기 때문).
>
> | Level | AP | AP50 | AP75 | APvt | APt | APs | APm |
> |---|---|---|---|---|---|---|---|
> | - | 20.6 | 50.4 | 12.9 | 7.0 | 20.8 | 25.7 | 32.1 |
> | {P2} | **21.2** | **51.8** | **13.5** | 8.0 | **21.5** | 26.0 | 32.5 |
> | {P2, P3} | 20.9 | 50.8 | 13.3 | **8.1** | 21.2 | 26.0 | **32.6** |
>
> #### Table 6 — CAREM 고주파 필터·채널attention(CAM) ablation
> **보는 법**: 필터 없음/CAM만/0-1 필터/Gauss 필터 4가지 조합을 비교 — Gauss 필터+CAM 조합이 최고(AP 21.4).
>
> | Filter | CAM | AP | AP50 | AP75 | APvt | APt | APs | APm |
> |---|---|---|---|---|---|---|---|---|
> | - | - | 20.6 | 50.4 | 12.9 | 7.0 | 20.8 | 25.7 | 32.1 |
> | - | ✓ | 20.9 | 51.4 | 13.0 | 7.8 | 20.8 | 25.8 | 33.0 |
> | 0–1 | ✓ | 21.2 | 51.9 | 13.5 | 7.8 | 21.5 | 26.3 | 32.1 |
> | Gauss | ✓ | **21.4** | 51.6 | **13.8** | **8.0** | **21.6** | **26.7** | **33.3** |
>
> #### Table 7 — CAREM Gaussian 필터 σ 영향
> **보는 법**: σ가 작을수록 저주파를 강하게 억제(APvt/APt에 유리), 클수록 저주파를 많이 남김(전체적으로 완만) — σ=2가 종합 최적.
>
> | σ | AP | AP50 | AP75 | APvt | APt | APs | APm |
> |---|---|---|---|---|---|---|---|
> | - | 20.6 | 50.4 | 12.9 | 7.0 | 20.8 | 25.7 | 32.1 |
> | 0.2 | 21.2 | **52.2** | 13.4 | 8.3 | 21.3 | 26.3 | 32.9 |
> | 1 | 21.3 | 52.0 | **13.8** | 8.1 | 21.5 | 26.5 | 32.2 |
> | 2 | **21.4** | 51.6 | **13.8** | 8.0 | **21.6** | 26.7 | **33.3** |
> | 5 | 21.0 | 51.7 | 13.1 | **8.7** | 21.1 | 26.9 | 32.1 |
> | 10 | 20.9 | 51.3 | 13.1 | 7.4 | 21.0 | 26.9 | 32.0 |
>
> #### Table 8 — SAS·vehicle 인스턴스 축소 효과 (AI-TOD test, 카테고리별 AP)
> **보는 법**: Ratio_VE(vehicle 유지 비율)를 100%→20%로 낮춰가며 다른 카테고리 성능이 유지되는지, few-shot 카테고리(AI/BR/SP/WM)가 개선되는지 확인.
>
> | Ratio_VE | Instance_VE | AI | BR | ST | SH | SP | VE | PE | WM | AP | AP50 | AP75 |
> |---|---|---|---|---|---|---|---|---|---|---|---|---|
> | - | - | 24.0 | 14.9 | 35.5 | 38.7 | 11.1 | 24.3 | 10.4 | 5.8 | 20.6 | 50.4 | 12.9 |
> | 100% | 369k | 32.3 | 20.7 | 36.9 | 38.9 | 22.0 | 24.6 | 9.9 | 10.6 | 24.4 | 58.1 | 16.7 |
> | 70% | 296k | 32.6 | 20.2 | 36.6 | 39.1 | 19.6 | 24.0 | 10.3 | 9.4 | 24.0 | 56.4 | 16.3 |
> | 50% | 241k | 32.1 | 20.2 | 36.1 | 39.2 | 18.9 | 22.9 | 10.5 | 9.3 | 23.6 | 56.0 | 15.9 |
> | 20% | 163k | 32.2 | 20.4 | 36.0 | 40.4 | 19.1 | 21.3 | 10.5 | 8.6 | 23.6 | 55.6 | 16.5 |
>
> few-shot 카테고리(AI/BR/SP/WM) 각각 +8.3%p/+5.8%p/+10.9%p/+4.8%p AP 개선. Vehicle 인스턴스를 369k→163k로 줄여도 다른 카테고리 저하 없이 vehicle 자체만 24.6%→21.3%로 하락 — 중복 제거 효과로 해석.
>
> #### Table 12 — VisDrone2019·DOTA-v1.5 일반화 (SAS 미적용, MSFFEM patch size는 각각 [(24,32),(48,64)] / [64,128]로 조정)
> **보는 법**: RetinaNet·FCOS·Faster R-CNN·Cascade R-CNN·DetectoRS·NWD-RKA·RFLA 등 기존 방법과 비교, baseline(RFLA) 대비 개선폭 확인.
>
> | Method | VisDrone AP/AP50/APt | DOTA-v1.5 AP/AP50/APt |
> |---|---|---|
> | Faster R-CNN w/ RFLA (baseline) | 21.1/41.6/7.0 | 40.0/67.5/13.3 |
> | FANet (Ours) | 21.7/43.0/7.5 | 40.5/68.5/14.2 |
>
> #### Table 10 — VisDrone/DOTA ablation (validation set)
> MSFFEM·CAREM 각각 단독 적용보다 결합 적용이 두 데이터셋 모두에서 우수 — AI-TOD 국한 효과가 아님을 시사(VisDrone AP 21.1→21.7, DOTA-v1.5 AP 40.0→40.5).
>
> #### Figure 3, Figure 4 — 패치 단위 FFT 스펙트럼 시각화
> **보는 법**: 왼쪽 원본 이미지의 빨간 원(tiny object 위치)에 대응하는 패치의 FFT 스펙트럼(오른쪽)이 다른 패치와 뚜렷이 다른 밝기 패턴을 보이는지 확인 — MSFFEM이 이 차이를 학습 가중치로 활용한다는 근거.
>
> #### Figure 7 — Eigen-CAM 기반 MSFFEM 효과 시각화
> **보는 법**: MSFFEM 처리 전/후 P2 feature map의 CAM 히트맵을 비교 — storage tank처럼 기하학적 구조가 뚜렷한 객체에서 활성화가 더 선명해지는지 눈으로 확인.
>
> #### Figure 8, Figure 9, Figure 10 — 정성적 검출 결과
> **보는 법**: baseline/FANet/GT 3열 비교(Figure 8, AI-TOD) — 개활 해상(open sea) 등 저대비 장면에서 FANet의 오탐(false positive)이 baseline보다 적은지 확인. Figure 9·10은 각각 AI-TOD·VisDrone2019에서 카테고리별 색상 박스로 표시된 FANet 단독 결과.

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- 초극소 객체(2~8px)는 정보 자체가 거의 없어 주파수 필터링 효과가 제한적일 위험 → <mark style="background: #FF5582A6;">실제로 patch size 100 설정에서 APvt가 baseline(7.0)보다 개선폭이 작고(7.6), MSFFEM+CAREM만으로는 APvt가 7.0→8.2에 그침 — 반면 SAS까지 포함한 전체 조합에서는 APvt가 10.7까지 오르는데, 논문은 이 개선이 주파수 필터링과 SAS(샘플 증가)의 기여를 분리해서 밝히지 않는다.</mark>
- 학습된 주파수 가중치가 특정 데이터셋 통계에 최적화되어 도메인 전이 시 부적합할 위험 → <mark style="background: #FF5582A6;">저자도 "contextual pattern이 도메인 간 크게 바뀌면 효과가 달라질 수 있다"고 명시(§6.2)하며 domain adaptation을 향후 과제로 남겨 미해결.</mark>

### 한계
- <mark style="background: #FF5582A6;">Very tiny object는 객체 영역 내 고유 정보 자체가 부족해 주변 문맥에 의존할 수밖에 없다 — 주파수 강화는 경계를 부각시킬 뿐 없던 정보를 만들어낼 수는 없다(저자 명시, §6.2).</mark>
- <mark style="background: #FF5582A6;">극소 스케일에서는 bounding box 위치의 미세한 차이도 IoU에 큰 영향을 줘 annotation·평가 자체가 불안정할 수 있음(저자 지적, §5.2.1) — FANet만의 한계는 아니나 수치 해석 시 감안 필요.</mark>
- <mark style="background: #FF5582A6;">정적 이미지에 국한 — 영상 시퀀스의 시간적 정보는 미활용(저자가 향후 과제로 명시, §7).</mark>
- CAREM 도입으로 추론 속도 9~19% 저하(예: RFLA 기준 FPS 42.5→38.7) — 실시간 애플리케이션에서는 트레이드오프 고려 필요.

### 생각할 점
- <mark style="background: #A6E3A1A6;">MSFFEM/CAREM은 "고주파=경계·텍스처, 저주파=배경"이라는 가정에 기반한다. 이 가정이 깨지는 상황(고주파 텍스처를 가진 도심·암반 배경, 또는 안개·저조도로 고주파 성분이 거의 없는 영상)에서는 역효과가 날 수 있으나 논문은 이런 실패 조건을 별도 분석하지 않는다 — 배포 전 검증 필요.</mark>
- SAS의 R_log(k) 분석은 카테고리 간 주파수 특성이 다름을 보였을 뿐, 카테고리별 최적 증강 배수까지 최적화하지는 않는다 — 현재는 인스턴스 수 반비례 휴리스틱(×1~×8)인데, R_log(k) 유사도 자체를 증강 강도에 반영하는 것도 가능해 보인다.

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">[[Frequency_Domain_Feature_Attention]]으로 분류된 MSFFEM/CAREM은 [[Small_Object_Detection_Approaches]] 비교표 기준 "feature 강화(주파수 영역)" 축에 속한다. [[Unc-SOD]]의 instance-level uncertainty 기반 sampling(label assignment 축)과는 직교적 개선이므로, RFLA에 FANet을 얹었을 때 이미 상호 보완적 이득이 확인된 것처럼(Table 2, RFLA AP50 50.4→52.4) 두 축을 결합하면 추가 이득 여지가 있다.</mark>
- <mark style="background: #A6E3A1A6;">[[SR-TOD]]의 self-reconstruction difference map은 정보 손실이 큰 영역을 공간적으로 찾는 방식인데, FANet의 R_log(k) 같은 주파수 통계와 결합하면 "공간적으로 어디에" + "주파수적으로 어떤 대역에" 정보가 몰려 있는지 동시에 활용하는 하이브리드 강화가 가능할 것으로 보인다.</mark>

> [!info] 내 메모
> 

# 관련 개념
- [[Frequency_Domain_Feature_Attention]] — 이 논문의 핵심 기여인 MSFFEM/CAREM이 사용하는 주파수 영역(DFT/DCT) 기반 적응형 attention 기법. 이 개념 문서의 "등장 논문"에 FANet이 이미 등재되어 있음.
- [[1x1_Convolution]] — MSFFEM의 다채널 스펙트럼 융합, CAREM의 채널 attention 내부 연산에 사용.
- [[Squeeze_And_Excitation_Channel_Attention]] — CAREM의 채널 attention 서브모듈이 그대로 따르는 범용 구조(GMP/GAP → 1×1 conv → sigmoid → 채널별 스케일링).

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]] — feature 강화 계열 중 "주파수 영역" 축으로 분류되며, label assignment 축인 [[Unc-SOD]], self-reconstruction 축인 [[SR-TOD]] 등과 대비됨.

# 읽어볼 만한 논문
- 참고문헌 기반: Z. Shi et al., "HS-FPN: High Frequency and Spatial Perception FPN for Tiny Object Detection" [44] (AAAI 2025) — FANet의 CAREM이 필터 설계(0-1 필터 비교)를 참고한 논문으로, 고주파 응답을 attention으로 활용하는 가장 직접적인 선행 연구. MSFFEM/CAREM과의 차이(feature map+RoI 이중 적용 vs 단일 지점)를 이해하는 데 필수적.
- 참고문헌 기반: C. Xu et al., "RFLA: Gaussian receptive field based label assignment for tiny object detection" [36] (ECCV 2022) — 이 논문의 baseline이자 비교 대상. Label assignment 축(gap 서술의 갈래 1)의 대표 논문으로, FANet의 feature 강화 축과 직교적인 접근이라 결합 가능성을 검토할 때 먼저 읽을 만함.
- 참고문헌 기반: B.N. Patro et al., "SpectFormer: Frequency and Attention is what you need in a Vision Transformer" [43] (arXiv 2023) — self-attention 자체를 spectral attention으로 대체하는 접근. FANet은 CNN 기반 R-CNN 구조에 주파수 모듈을 plug-in하는 방식인데, Transformer 백본에서 주파수 정보를 원천적으로 다루는 방식과 비교하면 "어디에 주파수 정보를 넣을지"에 대한 설계 스펙트럼을 넓게 볼 수 있다.
- 참고문헌 기반: B. Cao et al., "Visible and Clear: Finding Tiny Objects in Difference Map" (SR-TOD) [64] (ECCV 2024) — 이미 [[SR-TOD]]로 위키에 등재된 논문. FANet과 같은 RFLA baseline 위에서 비교되며(Table 11, AP 20.6%로 동률), self-reconstruction 기반 difference map이라는 전혀 다른 신호로 "약한 feature 보강"이라는 같은 문제를 푼다 — Discussion의 후속 연구 아이디어(공간+주파수 하이브리드)를 검토할 때 재독 가치가 있음.
- 자유 추천(검증 필요): 원격탐사·항공 이미지에서 반복적 텍스처(건물 격자, 농경지 패턴 등)가 주파수 도메인 필터링에 미치는 영향을 다루는 연구 — 검색 키워드: `periodic texture frequency domain false positive remote sensing object detection`. Discussion에서 제기한 "고주파=경계라는 가정이 깨지는 상황"에 대한 실증 연구가 있는지 확인할 때 유용할 것으로 예상.
