---
pm-task: true
projectId: "paperwiki-small-object-detection"
parentId:
id: "t-cdatod-diff-9qzk8ozqjp"
title: "Vision-Language Guided Semantic Diffusion Sampling for Small Object Detection in Remote Sensing Imagery"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-08-05"
due: ""
progress: 0
assignees: []
tags: []
customFields:
  "7l8l795xmtcjvlf6": 2025
  "1frf59rymtcjvske": "Remote Sensing (MDPI)"
subtaskIds: []
dependencies: []
year: 2025
venue: "Remote Sensing (MDPI)"
jcr_quartile: Q2
task: [small-object-detection]
direction: [novel-approach]
paper_tags: [paper, small-object-detection, sar, remote-sensing, vision-language-model, clip, diffusion-model, label-assignment, bounding-box-regression]
source: "/Users/GyeongSeo/Workspace/논문_pdf/Small_Object_Detection/2025_RemoteSensing_CDATOD-Diff.pdf"
createdAt: "2026-08-18T11:00:00.000Z"
updatedAt: "2026-08-28T00:00:00.000Z"
---

#paper #small-object-detection #sar #remote-sensing #vision-language-model #clip #diffusion-model #label-assignment #bounding-box-regression

> [!quote] 원제
> **Vision-Language Guided Semantic Diffusion Sampling for Small Object Detection in Remote Sensing Imagery**
> Jian Ma, Mingming Bian, Fan Fan, Hui Kuang, Lei Liu, Zhibing Wang, Ting Li, Running Zhang — Institute of Remote Sensing Satellite, China Academy of Space Technology, Remote Sensing (MDPI) 2025

# 한 줄 요약
<mark style="background: #FFF3A3A6;">GT 박스를 2D Gaussian으로 모델링해 뽑은 anchor 샘플링 포인트를 CLIP의 이미지-텍스트 의미 임베딩을 조건으로 한 diffusion denoising 과정으로 정제해 소형 객체의 양성 샘플 부족을 완화하고, 객체 크기에 따라 corner distance와 IoU의 기여도를 적응적으로 가중합하는 BC-IoU loss로 회귀 불안정성을 줄이는 CDATOD-Diff 프레임워크.</mark>

> [!info] 내 메모
> 

# 정리

| | 문제 ① — 소형 객체 양성 샘플 부족 | 문제 ② — 회귀 손실의 스케일 민감도 |
|---|---|---|
| **문제 정의** | 소형 객체는 크기가 극도로 작아 고정 grid 형태의 균일 샘플링 포인트와 유효한 대응을 형성하지 못하거나, 형성해도 미미한 매칭에 그친다(Fig. 1). 게다가 기존 anchor 샘플링이 맥락적 prior를 충분히 반영하지 않아 학습 단계에서 양성-음성 샘플 불균형이 심화되고 네트워크 최적화 효율이 저해된다. | IoU 기반 손실은 소형 객체의 미세한 위치 오차에 극도로 민감(overlap이 쉽게 0에 가까워짐)한 반면, 중심점 거리 손실은 예측 중심과 GT 중심이 일치하면 박스 크기와 무관하게 0이 되어 버려 크기 회귀에 대한 supervision이 사라진다. |
| **풀고자 하는 문제** | 소형 객체의 양성 샘플 부족 문제를 CLIP의 의미적 prior로 완화하고, Gaussian 생성 + diffusion denoising으로 anchor 샘플링 포인트 자체를 반복적으로 정제하는 것 | 객체 크기에 따라 IoU와 corner distance의 기여도를 동적으로 조정해 회귀 불안정성을 줄이는 것 |
| **선행 연구 접근** | - S3FD[34]: 낮은 초기 IoU 임계값으로 2단계 매칭 + 미매칭 GT에 대한 Top-N 랭킹<br>- Zhang et al.[35](ATSS): 타겟 통계 특성에 기반해 할당 임계값을 동적으로 조정<br>- Zhu et al.[36](AutoAssign): 이진 분류 대신 dual-weight 할당으로 양성/음성 기여도 균형<br>- Xu et al.[37](RFLA): Gaussian receptive field 기반 매칭(거리-점수 순위 top-K 후보 1차 선정 + decayed field radius 2차 정제) — 이 논문이 직접 확장하는 가장 가까운 선행 연구<br>- DiffDet4SAR[23]: bounding box 자체를 diffusion denoising target으로 모델링, 산란 feature 강화 모듈로 clutter 억제<br>- Qiu et al.[41]/Basso[42]/Bazi et al.[43]: CLIP을 feature extraction·검색·VQA에 활용<br>- **갭**: label assignment 계열은 기하학적/통계적 기준(거리, IoU, Gaussian receptive field)만으로 후보를 정제할 뿐 "이 위치가 실제로 어떤 의미를 갖는 객체인가"라는 의미적 정보를 활용하지 않고, VLM 계열은 탐지의 anchor 샘플링 과정 자체에는 개입하지 않는다. | 표준 IoU / GIoU / DIoU / CIoU / EIoU / SIoU 등 overlap·거리·종횡비·각도 기반 손실들이 각각의 기하 요소를 조합해 왔으나, 소형 객체에서 IoU 계열이 위치 오차에 과민 반응하고 중심점 거리 손실이 크기 supervision을 잃는 문제 자체를 스케일에 따라 적응적으로 절충한 시도는 없었다. |
| **해결 방법** | RFLA의 Gaussian receptive field 매칭을 계층적으로 확장하고, GT 박스 면적에 비례한 Gaussian 샘플을 CLIP 조건부 diffusion denoising으로 반복 정제해 의미적으로 타당한 양성 샘플을 늘림 | 중심점 거리 대신 두 모서리(좌상단·우하단) 좌표 거리 기반 corner loss를 정의하고, 객체 면적에 따라 지수적으로 감소하는 가중치로 corner loss와 IoU loss를 혼합(BC-IoU) |
| **예상되는 문제점** | Diffusion denoising은 timestep을 반복하는 과정이라 연산 비용이 커질 수 있고, CLIP은 자연 이미지로 사전학습되어 SAR처럼 도메인이 크게 다른 영상에서 프롬프트-이미지 정렬 품질이 불확실 | 가중치 함수의 스케일 인자(β)·거리 정규화 상수(S)가 데이터셋별 객체 크기 분포에 맞춰 수동 설정되는 하이퍼파라미터라, 분포가 크게 다른 새 도메인에 그대로 이식 가능한지는 검증되지 않음 |

**갭 종합**: <mark style="background: #FFF3A3A6;">Label assignment 계열은 기하학적/통계적 기준만으로 후보를 정제할 뿐 "이 위치가 실제로 어떤 의미를 갖는 객체인가"라는 의미적 정보를 활용하지 않고, VLM 계열은 탐지의 anchor 샘플링 프로세스 자체에는 개입하지 않는다. CLIP의 크로스모달 의미 정보를 diffusion 기반 생성적 샘플링의 조건으로 직접 결합해 "의미적으로 타당한 위치에 편향된" 샘플을 생성하는 접근은 없었으며, 여기에 스케일 적응적 회귀 손실을 결합하면 샘플링과 회귀 두 단계 모두에서 소형 객체 특유의 취약점을 보완할 수 있다는 것이 이 논문의 통찰이다.</mark>

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">GT 박스를 2D Gaussian 분포로 모델링해 그 분포에서 샘플링 포인트를 생성하고, 이 포인트들을 <span style="color:#c0392b; font-weight:bold;">CLIP의 이미지-텍스트 의미 임베딩을 조건으로 한 diffusion denoising 과정</span>으로 반복 정제함으로써 순수 기하학적 기준보다 의미적으로 더 신뢰할 수 있는 양성 샘플을 생성한다. 여기에 스케일에 따라 corner distance와 IoU의 비중을 적응적으로 바꾸는 <span style="color:#c0392b; font-weight:bold;">BC-IoU(Balanced Corner-IoU) loss</span>를 결합해 회귀 안정성을 높인다.</mark>

## 전체 파이프라인 (Fig. 2, Fig. 3 기준)

```
입력: RS 이미지 (3, H, W) + GT 박스 + 클래스 텍스트 프롬프트 "an image of [CLASS]"
       │
       ▼
① CLIP 이미지/텍스트 인코딩                       → 이미지 F_image={F¹_clip..F¹²_clip}(각 H_l×W_l×D), 텍스트 F_text(768,)
       │
       ▼
② CLIP-Driven Dynamic Anchor Point Sampling      → 계층적 label(양성 anchor 후보) R = R1·m + R2·(1-m)
   (RFLA 확장: WDS 기반 1차 R1 + decayed field 2차 R2, Deformable offset 보정)
       │
       ▼
③ Diffusion-Based Anchor Point Sampling + CLIP 조건화 → 정제된 샘플링 포인트 위치 (DDIM, 학습 시 3회 반복)
   (Gaussian sampler로 GT 면적 비례 n개 샘플 → forward 노이즈 주입 → CLIP 조건부 reverse denoising)
       │
       ▼
④ Spatial Calibration (Deformable Conv 보정)       → 보정된 feature map F_CD (H×W, C)
       │
       ▼
⑤ Shared Detection Head (Reg + Cls)               → Reg 출력 (H×W, 4) + Cls 출력 (H×W, num_classes)
       │
       ▼ (학습 시에만)
⑥ BC-IoU Loss                                     → L_BC-IoU = w·L_Corner + (1-w)·L_IoU  (스칼라 loss)
```

> [!info] 내 메모
> 

### ① CLIP 이미지/텍스트 인코딩
- **역할**: 입력 이미지와 클래스 이름을 CLIP의 공유 임베딩 공간으로 각각 인코딩해, 이후 diffusion 조건화 단계에서 "이 위치가 실제로 [CLASS]와 의미적으로 부합하는지" 판단할 근거를 만든다. CLIP(Contrastive Language-Image Pre-training)은 이미지-텍스트 쌍을 대조학습(contrastive learning)으로 정렬한 vision-language model이다.
- **구현**: 이미지는 ViT-B/16(Vision Transformer, 16×16 패치)로 12개 transformer layer를 거쳐 계층적 feature `{F¹_clip, ..., F¹²_clip}`(각 층 H_l×W_l×D 유지)를 추출한다. 텍스트는 "an image of [CLASS]" 프롬프트를 transformer 기반 텍스트 인코더에 넣어 768차원 벡터로 인코딩한다. 최종적으로 이미지·텍스트 인코딩을 concat 후 1×1 conv + projection으로 통합 CLIP feature `F_clip`을 만든다([[1x1_Convolution]] 참고).
- **입출력 shape**: 이미지 `(3, H, W)` → 계층적 feature `{(H_l, W_l, D)}_{l=1}^{12}`. 텍스트 프롬프트 → `(768,)`.

```python
# 의사코드 — 논문 Fig.3, 식(1)(2) 기반
F_image = [image_transformer_layer_i(patches) for i in range(12)]  # 각 (H_l, W_l, D)
F_text = text_encoder("an image of " + class_name)                 # (768,)
F_clip = Conv1x1(Projection(concat(F_image, F_text)))
```

> [!info] 내 메모
> 

### ② CLIP-Driven Dynamic Anchor Point Sampling
- **역할**: [[RFLA]] #pending:rfla 가 제안한 "GT 박스를 2D Gaussian receptive field로 모델링해 anchor와의 거리 점수로 양성 샘플을 뽑는" 방식을 계층적으로 확장해, GT 하나당 배정되는 양성 샘플 수를 늘린다. 순수 거리 기반 매칭에 학습 가능한 위치 보정(offset)까지 더해 고정 grid의 한계를 넘어선다.
- **구현**: receptive field `n_e = N(μ_e, Σ_e)`와 GT 박스 `n_g = N(μ_g, Σ_g)` 사이의 Wasserstein distance 기반 점수(WDS)로 1차 후보 `R1`을 뽑고, receptive field 반경을 줄여 재랭킹한 2차 결과 `R2`를 mask `m`으로 결합(`R = R1·m + R2·(1-m)`)해 GT당 양성 샘플 수를 늘린다. 이후 해상도가 가장 높은 두 feature map에서, 3×3 conv로 예측한 offset `Δp=(d_x, d_y)`를 deformable convolution 방식으로 샘플링 위치에 더한다.
- **입출력 shape**: feature map `(H, W, C)` + GT 박스 좌표 → 1차·2차 결합 label `R (ΣN_i, 2)` (`N_i`는 GT i에 배정된 양성 샘플 개수의 합).

```python
# 의사코드 — 논문 식(3)~(6) 기반
WDS = 1 / (1 + ||[x_n, y_n, e^{r_n}, e^{r_n}] - [x_g, y_g, w_g/2, h_g/2]||_2**2)
R1, m = rank_top_k(WDS)                       # 1차 선정 + mask
R2 = rank_top_k(WDS, reduced_radius=True)     # 반경 축소 후 재랭킹(2차)
R = R1 * m + R2 * (1 - m)                     # 최종 결합 label
x_new, y_new = x + stride * dx / 2, y + stride * dy / 2   # deformable offset 보정
```

<mark style="background: #FFF9D6A6;">Gaussian receptive field 기반 거리 점수만으로는 여전히 기하학적 유사도만 반영하므로 "정리" 표의 소형 객체 양성 샘플 부족 문제를 완전히 해소하지 못한다. 계층적 2단계 매칭으로 GT당 양성 샘플 수 자체를 늘리고, deformable offset으로 샘플링 위치를 학습 가능하게 만들어 고정 grid의 한계를 넘어서는 유연성을 확보한다.</mark>

> [!info] 내 메모
> 

### ③ Diffusion-Based Anchor Point Sampling + CLIP 조건화
- **역할**: ②에서 만든 기하학적 후보만으로는 "의미적으로 실제 객체가 맞는지"를 판단하지 못하므로, 샘플링 포인트를 diffusion denoising 과정으로 반복 정제하면서 CLIP의 의미 임베딩을 조건으로 주입해 의미적으로 타당한 위치로 편향시킨다. 자세한 동작 원리는 [[CLIP_Conditioned_Diffusion_Anchor_Sampling]] 참고.
- **구현**: GT 박스 면적에 비례하는 개수의 샘플 포인트를 Gaussian sampler로 무작위 생성 후 DDPM forward process로 노이즈를 주입하고, DDIM으로 가속한 역방향 디노이징을 거쳐 위치를 복원한다. CLIP 텍스트·이미지 임베딩을 modality-specific mapping으로 정렬한 뒤 diffusion U-Net의 각 단계에 residual 방식으로 주입한다. 디노이징 후 위치가 원래 feature map 격자와 어긋나므로, spatial calibration 모듈이 균일 참조점과의 위치 편차를 측정해 deformable convolution으로 feature map을 보정한다.
- **입출력 shape**: 초기 샘플 `x_0 (n, 2)` (n=GT 면적 비례 개수) + CLIP feature `F_clip` → 정제된 샘플 위치 `x_p (n, 2)` → 보정된 feature map `(H, W, C)`.

```python
# 의사코드 — 논문 식(7)~(13) 기반
x_t = sqrt(alpha_bar_t) * x_0 + eps * sqrt(1 - alpha_bar_t)         # forward: 노이즈 주입
f_ta = spatial_replicate(F_text)                                    # 텍스트 임베딩 공간 복제
f_is = MLP_projection(F_image)                                      # 이미지 임베딩 투영
F_fusion = sigmoid(W_text * f_ta) + (W_image * f_is)                # CLIP 조건 융합 (식10)
# DDIM reverse (식8, 9), 학습 시 3회 반복
sigma2 = eta * (1 - alpha_bar_p) / (1 - alpha_bar_t) * (1 - alpha_bar_t / alpha_bar_p)
x_p = sqrt(alpha_bar_p) * ((x_t - sqrt(1 - alpha_bar_t) * eps_t) / sqrt(alpha_bar_t)) \
      + sqrt(1 - alpha_bar_p - sigma2) * eps_t + sigma * eps
F_CD = Conv1x1(ReLU(concat(F_trans_clip, F_diff)))                   # CLIP 조건 결합 (식13)
```

<mark style="background: #FFF9D6A6;">CLIP의 크로스모달 의미 임베딩을 diffusion 조건으로 주입함으로써, 순수 기하학적 거리(②의 WDS)만으로는 구분할 수 없는 "의미적으로 타당한 객체 위치"를 반영한 샘플을 생성한다 — 이는 "정리" 표의 양성 샘플 불균형을 기하학적 정제와 의미적 정제 두 층위에서 동시에 완화하는 설계다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> Diffusion denoising은 timestep을 반복하는 과정이라(학습 시 1000 timestep 중 3회 순차 샘플링) 단일 forward로 끝나는 기존 label assignment보다 연산 비용이 늘어날 가능성이 크다. 논문은 DDIM으로 가속했다고만 밝힐 뿐, 실제 FPS·추론 시간 비교는 제시하지 않는다("Discussion" 참고).

> [!info] 내 메모
> 

### ④ Spatial Calibration + Shared Detection Head
- **역할**: 디노이징을 거친 샘플 위치는 연속 좌표라 원래 feature map의 정수 격자와 정확히 맞지 않는다. Spatial calibration이 이 편차를 deformable convolution의 offset으로 변환해 feature map 자체를 보정하고, 보정된 feature를 분류(Cls)·회귀(Reg) 두 head가 공유해 최종 예측을 만든다.
- **구현**: 균일하게 배치한 참조점과 디노이즈된 샘플점 사이의 위치 차이를 계산해 deformable convolution의 학습 가능한 offset으로 변환, feature map의 공간 구조를 이 offset만큼 재배열한다. 이후 회귀·분류 head는 baseline인 FCOS 구조를 그대로 공유(shared heads)한다.
- **입출력 shape**: `F_CD (H, W, C)` → Reg 출력 `(H, W, 4)` + Cls 출력 `(H, W, num_classes)`.

```python
# 의사코드
offset = measure_positional_deviation(uniform_ref_points, x_p)
F_calibrated = deformable_conv(F_CD, offset)     # (H, W, C) -> (H, W, C), 격자 정렬
reg_out = reg_head(F_calibrated)                 # (H, W, 4)
cls_out = cls_head(F_calibrated)                 # (H, W, num_classes)
```

> [!info] 내 메모
> 

### ⑤ Balanced Corner-IoU (BC-IoU) Loss
- **역할**: 예측 박스와 GT 박스 사이의 회귀 손실을 계산하되, 소형 객체에서 IoU 손실이 과민 반응하고 중심점 거리 손실이 크기 supervision을 잃는 두 문제를 동시에 완화한다.
- **구현**: 중심점 거리 대신 예측·GT 박스의 두 모서리(좌상단, 우하단) 좌표 거리 합으로 corner loss `L_Corner = 1 - e^{-D_corn/S}`(S=4)를 정의해, 중심이 일치해도 크기 오차가 남아있으면 loss가 0이 되지 않도록 한다. 객체 면적 A에 따라 지수적으로 감소하는 가중치 `w = e^{-A/β}`(β=12)로 corner loss와 IoU loss를 혼합한다 — 소형 객체일수록 corner loss 비중이 커지고, 큰 객체일수록 IoU loss 비중이 커진다.
- **입출력 shape**: 예측 박스 `(4,)` + GT 박스 `(4,)` + 면적 A(스칼라) → loss 스칼라.

```python
# 의사코드 — 논문 식(14)~(19) 기반
L_IoU = 1 - (P ∩ G) / (P ∪ G)                    # 식(14)
D_corn = sum(dist(pred_corner_i, gt_corner_i) for i in [top_left, bottom_right])  # 식(17)
L_Corner = 1 - exp(-D_corn / S)                  # S=4, 식(15)
w = exp(-A / beta)                               # beta=12, 식(18)
L_BC_IoU = w * L_Corner + (1 - w) * L_IoU         # 식(19)
```

<mark style="background: #FFF9D6A6;">중심점 거리 손실이 중심 일치 시 크기 회귀 supervision을 잃는 문제와, IoU 손실이 소형 객체에서 과민 반응하는 문제를 각각 corner 기반 재정의와 스케일 적응 가중치로 해결해, "정리" 표의 회귀 불안정성 문제를 스케일 전 구간에서 완화한다(Fig. 5에서 corner loss는 스케일에 걸쳐 안정적, IoU loss는 소형 객체에서 급격히 변함을 실험적으로 확인).</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> 가중치 함수의 스케일 인자 β(=12)와 corner loss의 거리 정규화 상수 S(=4)는 AI-TOD 등 실험에 쓰인 데이터셋의 객체 크기 분포에 맞춰 고정된 하이퍼파라미터다 — 객체 크기 분포가 크게 다른 도메인에 그대로 옮겨 쓸 수 있는지는 논문에서 검증되지 않았다.

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| ① CLIP 인코딩 | 이미지(3,H,W) + 텍스트 | {(H_l,W_l,D)}×12 + (768,) | 이미지-텍스트 공유 임베딩 생성 | ViT-B/16(12층) + 텍스트 transformer + 1×1 conv |
| ② CLIP-Driven Dynamic Sampling | feature(H,W,C) + GT 박스 | 양성 label R(ΣN_i, 2) | RFLA 확장 계층적 매칭 + 학습 가능 offset | Wasserstein distance 랭킹(WDS) + deformable conv |
| ③ Diffusion Sampling + CLIP 조건화 | 초기 샘플(n,2) + CLIP feature | 정제된 샘플(n,2) → 보정 feature(H,W,C) | 의미적으로 타당한 양성 샘플 정제 | DDPM forward + DDIM reverse(3회) + CLIP 조건 융합 |
| ④ Spatial Calibration + Head | F_CD(H,W,C) | Reg(H,W,4) + Cls(H,W,K) | 격자 정렬 + 최종 예측 | Deformable conv 보정 + FCOS 공유 head |
| ⑤ BC-IoU Loss | 예측/GT 박스 + 면적 | 스칼라 loss | 스케일 적응 회귀 안정화 | Corner loss + IoU loss 가중합 |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table 1(MSAR-1.0), Table 3(AI-TOD)
**보는 법**: 각 행이 하나의 방법, 열이 벤치마크별 지표 — `Before`는 각 벤치마크에서 이전 최고 성능(SOTA) 방법의 값, `After`는 CDATOD-Diff(Proposed)의 값이다.

| 벤치마크 | 지표 | Before(이전 SOTA) | After(CDATOD-Diff) |
|---|---|---|---|
| MSAR-1.0 | AP / AP50 / AP75 / APs | 63.4 / 80.6 / 62.2 / 51.6 (3SD-Net) | 64.1 / 83.8 / 68.9 / 58.9 |
| AI-TOD | AP / AP50 / APt | 16.3 / 39.1 / 18.5 (RFLA) | 19.4 / 47.0 / 20.8 |

> [!note]- 세부 결과 및 Ablation
> #### Table 2 — HRSID(SAR 선박 탐지) SOTA 비교
> **보는 법**: ResNet50 backbone 기준 각 방법의 AP/AP50/AP75/APs/APm/APl 비교.
> | 방법 | AP | AP50 | AP75 | APs | APm | APl |
> |---|---|---|---|---|---|---|
> | Faster R-CNN | 50.8 | 80.5 | 56.2 | 33.0 | 67.6 | 43.2 |
> | AutoAssign(이전 최고 AP) | 52.0 | 85.6 | 55.8 | 35.0 | 67.3 | 44.2 |
> | **Proposed** | **55.4** | **85.5** | **61.5** | **39.7** | **69.8** | **50.7** |
> - Proposed가 소형(APs)·중형(APm)·대형(APl) 전 구간에서 최고 또는 최상위권 성능을 냄.
>
> #### Table 4 — VEDAI(차량 세부 클래스별) 비교
> **보는 법**: 열은 차량 서브클래스(BO=보트, CP=캠핑카, CA=승용차, OT=오일탱크, PI=픽업, TR=트랙터, TK=트럭, VA=밴), 값은 클래스별 AP, 마지막 열이 mAP.
> | 방법 | BO | CP | CA | OT | PI | TR | TK | VA | mAP |
> |---|---|---|---|---|---|---|---|---|---|
> | DiffusionDet(이전 최고 mAP) | 0.433 | 0.427 | 0.442 | 0.177 | 0.443 | 0.317 | 0.175 | 0.298 | 0.340 |
> | **Proposed** | **0.458** | 0.444 | 0.495 | 0.162 | 0.457 | **0.353** | 0.193 | **0.359** | **0.365** |
> - OT(오일탱크)는 0.162로 DiffusionDet(0.177)보다 낮아 유일하게 이전 최고를 넘지 못한 클래스.
>
> #### Table 5 — 모듈별 Ablation (AI-TOD/USOD/VEDAI, FCOS 기준)
> **보는 법**: 같은 베이스라인(FCOS)에 RFLA → Diffusion → Diff-CLIP을 순서대로 추가하며 AP가 누적 개선되는지 확인.
> | 데이터셋 | 구성 | AP | APvt | APt | APs | APm |
> |---|---|---|---|---|---|---|
> | AI-TOD | FCOS(baseline) | 0.107 | 0.023 | 0.11 | 0.151 | 0.207 |
> | AI-TOD | FCOS+RFLA | 0.163 | 0.073 | 0.185 | 0.198 | 0.218 |
> | AI-TOD | FCOS+Diffusion(CLIP 미적용) | 0.179 | 0.061 | 0.167 | 0.246 | 0.317 |
> | AI-TOD | **FCOS+Diff-CLIP(전체)** | **0.194** | **0.13** | **0.208** | 0.229 | 0.245 |
> | USOD | FCOS(baseline) | 0.106 | - | - | 0.14 | 0.208 |
> | USOD | **FCOS+Diff-CLIP(전체)** | **0.246** | - | - | **0.296** | 0.245 |
> | VEDAI | FCOS(baseline) | 0.017 | - | - | 0.012 | 0.055 |
> | VEDAI | **FCOS+Diff-CLIP(전체)** | **0.365** | - | - | **0.345** | **0.388** |
> - 저자 서술(본문 4.6.2): AI-TOD의 극소형(APvt)·타이니(APt) 객체에서 CLIP 조건화가 diffusion 단독 대비 각각 2.1, 4.1 추가 향상된다고 명시(표의 raw 값 0.061→0.13, 0.167→0.208 자체는 소수점 스케일이 달라 보이지만, 본문 서술을 그대로 인용).
>
> #### Table 6 — Loss ablation (AI-TOD)
> **보는 법**: IoU만 쓴 경우, Corner만 쓴 경우, 둘을 합친 BC-IoU를 비교.
> | 구성 | AP | AP50 | AP75 |
> |---|---|---|---|
> | IoU only | 10.6 | 26.8 | 6.3 |
> | Corner only | 11.2 | 27.7 | 6.6 |
> | BC-IoU(결합) | 12.5 | 30.5 | 8.0 |
> - BC-IoU가 IoU only 대비 +1.9 AP, Corner only 대비 +1.3 AP 개선.
>
> #### Fig. 8 — 다른 IoU 계열 손실과 비교
> **보는 법**: x축이 손실 종류(IoU/GIoU/DIoU/CIoU/EIoU/SIoU/BCIoU), y축이 AP/AP50/AP75 막대.
> - BC-IoU가 AP 12.5로 최고(다음은 EIoU 11.4), AP50도 30.5로 최고.
>
> #### Fig. 7 — USOD 데이터셋 loss 수렴 곡선
> **보는 법**: x축 epoch, y축 loss — FCOS(파랑)는 0.6대에서 정체, FCOS-RFLA(주황)는 완만히 감소, Proposed(초록)는 가장 빠르고 낮은 값(약 0.2대)으로 수렴.
>
> #### Fig. 9 — Grad-CAM 정성 비교
> **보는 법**: 각 행이 방법(FCOS/RFLA/Proposed), 열이 이미지/GT/Grad-CAM 히트맵 — Proposed가 실제 소형 객체 위치에 더 강하고 정확한 attention을 부여하는지 육안 확인.
> - VEDAI·USOD 양쪽에서 FCOS·RFLA 대비 오탐/누락이 시각적으로 감소.
>
> #### Fig. 10~13 — 데이터셋별 정성 비교 (VEDAI/AI-TOD/USOD/HRSID)
> **보는 법**: 초록=TP(정탐), 파랑/갈색원=FP(오탐), 빨강=FN(누락) 박스로 표시된 실제 탐지 결과 비교.
> - Fig. 10(VEDAI): 밀집 배경에서 baseline은 놓치거나(빨강) 중복 예측(파랑)하는 반면 Proposed는 더 완전하게 탐지.
> - Fig. 11(AI-TOD): 밀집 타겟에서 Proposed가 더 완전한 커버리지, 누락 감소.
> - Fig. 12(USOD): 극소형·저대비 객체에서 FCOS는 박스가 조각나거나 부정확한 반면 Proposed는 안정적.
> - Fig. 13(HRSID): FCOS는 강한 배경 산란체 주변에 오탐(갈색 원), RFLA는 실제 선박을 누락(빨강)하는 반면 Proposed는 배경 간섭을 억제하며 더 완전한 탐지.

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- Diffusion 기반 샘플링은 1000 timestep 중 학습 시 3회의 순차 디노이징 반복을 거치므로 연산 비용이 커질 가능성 → <mark style="background: #FF5582A6;">논문은 DDIM으로 가속했다고 언급할 뿐, 실제 FPS·추론 시간·파라미터 수 비교는 어디에도 제시하지 않는다.</mark>
- CLIP 텍스트 프롬프트가 고정된 "an image of [CLASS]" 템플릿에 의존 → <mark style="background: #FF5582A6;">SAR 영상처럼 시각 도메인이 CLIP의 원 학습 분포(자연 이미지)와 크게 다른 경우 프롬프트-이미지 정렬 품질이 보장되는지에 대한 검증이 없다.</mark>

### 한계
- <mark style="background: #FF5582A6;">추론 속도(FPS)나 파라미터 수 등 연산 비용 지표가 논문 전체에 전혀 보고되지 않아, diffusion+CLIP 결합이 실제 배포 환경에서 어느 정도 비용을 요구하는지 알 수 없다.</mark>
- <mark style="background: #FF5582A6;">VEDAI에서 OT(오일탱크) 클래스는 AP 0.162로, 비교 대상인 DiffusionDet(0.177)보다도 낮아 유일하게 이전 최고를 넘지 못했다 — 저자가 별도로 원인을 분석하지 않았다.</mark>
- 학습 스케줄이 12 epoch(SGD, lr 0.005, 배치 크기 2)로 매우 짧게 고정되어 있어, 이 성능이 더 긴 학습에서도 일관되게 재현되는지는 확인되지 않는다.

### 생각할 점
- <mark style="background: #A6E3A1A6;">RFLA의 Gaussian receptive field 매칭을 명시적으로 확장한다는 점에서, [[Position_Gaussian_Saliency_Map]]과 마찬가지로 "Gaussian으로 무언가를 모델링"하는 이 위키의 사례 중 하나다 — 다만 이 논문은 saliency map이 아니라 "샘플링 포인트 자체의 분포"를 모델링하고, 여기에 diffusion denoising이라는 시간축 반복 정제를 추가로 결합한다는 점에서 구별된다.</mark>
- <mark style="background: #A6E3A1A6;">CLIP을 label assignment/샘플링 단계에 결합한 시도는 이 위키에서 처음 등장 — 기존 feature 강화 계열([[SR-TOD]], [[FANet]] 등)과 직교적인 축(무엇을 강화할지가 아니라 어디를 양성 샘플로 볼지)이라 결합 가능성이 있다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">[[QueryDet]]의 Cascade Sparse Query가 "어디를 계산할지"를 저해상도 예측으로 좁히는 반면, 이 논문은 "어디를 양성 샘플로 볼지"를 diffusion으로 정제한다 — 두 메커니즘을 결합하면 연산 효율과 샘플 품질을 동시에 개선할 여지가 있다.</mark>
- <mark style="background: #A6E3A1A6;">SAR 영상은 원격탐사 도메인 중에서도 시각적으로 독특한(광학 이미지와 매우 다른) 특성을 갖는데, CLIP이 자연 이미지로 사전학습되었다는 점을 고려하면 SAR 특화 vision-language 사전학습으로 이 방법을 더 개선할 여지가 있어 보인다.</mark>

> [!info] 내 메모
> 

# 관련 개념
- [[Position_Gaussian_Saliency_Map]] — 둘 다 "GT 박스를 Gaussian으로 모델링"하지만 용도가 다르다(이 논문은 샘플링 포인트 분포, 저쪽은 attention prior).
- [[CLIP_Conditioned_Diffusion_Anchor_Sampling]] — 이 논문이 처음 제시하는 핵심 기여, CLIP 조건부 diffusion denoising으로 anchor 샘플링 포인트를 정제하는 구조.
- [[1x1_Convolution]] — CLIP feature 통합, spatial calibration 등 여러 지점에서 채널 재조합에 사용.

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]]

# 읽어볼 만한 논문
- 참고문헌 기반: C. Xu, J. Wang, W. Yang, H. Yu, L. Yu, G.-S. Xia, "RFLA: Gaussian receptive field based label assignment for tiny object detection" (ECCV 2022) [37] — 이 논문이 직접 확장하는 baseline. 여러 논문에서 반복 인용되는 만큼 우선순위가 매우 높음.
- 참고문헌 기반: A. Radford et al., "Learning transferable visual models from natural language supervision" (CLIP, ICML 2021) [4] — 이 논문의 CLIP 조건화 전체가 기반하는 원조 논문. Vision-language 정렬의 기본 원리 이해에 필수.
- 참고문헌 기반: S. Chen, P. Sun, Y. Song, P. Luo, "DiffusionDet: Diffusion model for object detection" (ICCV 2023) [57] — 바운딩 박스 자체를 diffusion target으로 삼는 원조 접근. 이 논문이 "박스가 아닌 anchor 샘플링 포인트를 diffusion으로 정제한다"는 차별점을 이해하려면 대조 비교가 필요.
- 참고문헌 기반: J. Zhou, C. Xiao, B. Peng, Z. Liu, Y. Liu, X. Li, "DiffDet4SAR: Diffusion-based aircraft target detection network for SAR images" (IEEE Geosci. Remote Sens. Lett. 2024) [23] — SAR 영상에서 diffusion을 탐지에 적용한 가장 가까운 선행 연구. Bounding box 자체를 diffusion target으로 삼는 방식과 이 논문의 anchor 샘플링 포인트 정제 방식의 차이를 비교하기 좋음.
- 자유 추천(검증 필요): SAR 도메인 특화 CLIP 사전학습(원격탐사 vision-language foundation model) 관련 연구 — 검색 키워드: `SAR remote sensing CLIP domain-specific pretraining vision-language foundation model 2025`
