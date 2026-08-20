---
pm-task: true
projectId: "paperwiki-reading-unified"
parentId:
id: "t-reconstruction_error_guided_instance_segmentation-el4vr9jz4o"
title: "Reconstruction Error Guided Instance Segmentation for Infrared Inspection of Power Distribution Equipment"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-08-05"
due:
progress: 0
assignees: []
tags: []
subtaskIds: []
dependencies: []
year: 2025
venue: "Sensors (MDPI)"
jcr_quartile: null
task: [instance-segmentation]
direction: [novel-approach]
paper_tags: [paper, instance-segmentation, infrared-inspection, reconstruction-error, feature-enhancement, uav, power-distribution]
source: "Projects/논문 읽기_pdf/Instance_Segmentation/2025_Sensors_Reconstruction-Error-Guided-Instance-Segmentation.pdf"
createdAt: "2026-08-19T11:16:30.000Z"
updatedAt: "2026-08-19T11:16:30.000Z"
---

Project: [[논문 읽기|논문 읽기]]
#paper #instance-segmentation #infrared-inspection #reconstruction-error #feature-enhancement #uav #power-distribution

# 한 줄 요약
<mark style="background: #FFF3A3A6;">적외선 배전설비 영상에서 소형·복잡구조 객체의 backbone 정보 손실을 진단하기 위해 object reconstruction decoder(ORD)로 원본 이미지를 복원하고, 그 재구성 오차(difference map)를 difference feature enhancement(DFE) 모듈로 backbone feature에 전역적으로 재주입해 instance segmentation 정확도를 끌어올리는 model-agnostic 프레임워크(RE)와, 이를 검증할 최초의 UAV 기반 적외선 배전설비 데이터셋 PDI.</mark>


# 문제 정의

### 기존 방법의 한계
- **소형 객체 content 압축**:
  bushing 등 배전설비는 UAV 촬영 거리에서 픽셀 수가 극히 적은 소형 객체가 되는데, backbone의 반복적 downsampling으로 feature가 수 픽셀에서 1픽셀 이하로 압축되어 노이즈·배경과 구분이 어려워진다.
- **복잡구조 객체 edge 단편화**:
  insulator, disconnector처럼 형상이 복잡한 설비는 downsampling의 이산화(discretization) 과정에서 경계가 끊어진 점들의 나열로 조각나, 구조적 연속성이 깨진다.
- **적외선 영상 특유의 저해상도 제약**:
  적외선 센서의 하드웨어 한계로 원본 해상도 자체가 낮아(640×512), 위 두 문제가 가시광 영상보다 더 심각하게 나타난다.

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Super-Resolution(SR) 기반 정보 손실 완화**
- SR [15,16]: 저해상도 영상에서 고해상도 대응물을 복원해 픽셀 밀도를 높임 — 존재하지 않는 texture·artifact를 만들어낼 위험이 있고, 고해상도 입력 자체가 모델 연산 부담을 키움.

**갈래 2 — 개선된 Down-sampling(IDS)**
- IDS [17,18]: 샘플링 영역 내 픽셀을 가중합해 정보를 보존 — 이 과정 자체가 laborious하며 근본적으로 정보 손실을 "줄일" 뿐 손실 위치를 진단하지는 않음.

**갈래 3 — 적외선 전력설비 특화 instance segmentation**
- Zhao et al. [22]: category-specific image patch를 spatial prior로 사용 — 시각적으로 유사한 카테고리 구분에 초점, 정보 손실 자체는 다루지 않음.
- Zhou et al. [14]: shuffle-polarized self-attention으로 장거리 의존성 포착 — 복잡구조 객체 강화가 목적이나 정보 손실 위치를 명시적으로 진단하지 않음.
- Li et al. [8]: super-resolution과 instance segmentation을 결합한 multi-task 프레임워크 — 고해상도 texture 전이를 시도하나 SR 계열의 근본 한계(artifact, 연산 비용) 공유.
- Li et al. [23]: spatial transformation network로 multi-scale spatial context 추출 — 역시 정보 손실의 위치·정도를 직접 신호화하지 않음.

**갭**: <mark style="background: #FFF3A3A6;">기존 접근들은 정보 손실을 완화하려는 시도(SR, IDS, 구조 개선)에 머물 뿐, "어디서 얼마나 정보가 손실됐는가"를 명시적으로 진단해 그 신호를 feature 강화에 직접 활용하지는 않는다. 또한 적외선 배전설비를 다루는 대규모 UAV 기반 instance segmentation 데이터셋 자체가 없어, 이 도메인의 벤치마크가 부재했다.</mark>

### 이 논문이 풀고자 하는 문제
1. Backbone의 downsampling으로 인한 소형 객체 content 손실과 복잡구조 객체 edge 단편화를 명시적으로 진단하는 방법
2. 진단한 정보 손실 신호를 별도 라벨 없이 self-supervised 방식으로 instance segmentation feature 강화에 재주입하는 방법
3. 적외선 UAV 배전설비 검사를 위한 대규모·다양한 벤치마크 데이터셋의 부재

# 제안 방법

<mark style="background: #FFF3A3A6;">Backbone feature로부터 원본 입력 이미지를 재구성(self-supervised reconstruction)하면, 정보가 심하게 손실된 객체 영역일수록 복원이 어려워 재구성 오차(difference map)가 크게 나타난다. 이 오차 지도를 "위치와 정도가 표시된 정보 손실 진단서"로 삼아, 원래의 backbone feature에 전역적으로 다시 주입해 소형 객체의 content와 복잡구조 객체의 edge를 동시에 보강한다.</mark>

### ① Object Reconstruction Decoder (ORD)
- Shared encoder(backbone+FPN)가 만든 multi-scale feature `F_b`로부터, encoder와 대칭적인 다단계 decoding group을 거쳐 원본 해상도의 재구성 이미지 `x_r`을 생성.
- 각 decoding group은 3×3 conv → ReLU → bicubic upsample(×2)로 구성되고, 대응하는 backbone feature와 skip-connection처럼 더해짐.
- 재구성은 object-centric — 객체 영역은 원본 픽셀값을 보존하고 비객체 영역은 0으로 억제한 뒤, 이 마스킹된 타깃에 대해 L1(MAE) reconstruction loss로 학습.
- MAE를 택한 이유는 MSE보다 outlier(예: 반사광 같은 밝은 픽셀)에 덜 민감해 더 선명한 복원 결과를 주기 때문.

> [!example]- 구현 디테일
> ```
> f_{m+1} = Bicubic(ReLU(Conv_3x3(f_m)), scale=2) + f_l     (decoding group, l=1..L, m=L-l+1)
> L_OR = ||x_r - x||_1   (object 영역 내에서만 계산)
> ```
> Bicubic interpolation은 4×4 이웃을 참조하는 3차 커널(nearest/bilinear보다 부드러운 upsampling).

<mark style="background: #FFF9D6A6;">SR과 달리 배경까지 복원하지 않고 객체 영역만 재구성하도록 supervision을 마스킹해, "문제 정의"의 정보 손실 진단에 필요한 신호만 효율적으로 얻는다. 픽셀 단위 오차에 민감한 reconstruction 특성상, content가 압축된 소형 객체와 edge가 끊긴 복잡구조 객체 모두에서 재구성이 어려워지므로, 재구성 오차가 두 문제 유형을 동시에 드러내는 공통 신호가 된다.</mark>

### ② Difference Feature Enhancement (DFE) 모듈
- 재구성 이미지 `x_r`과 원본 `x`의 채널별 절대차로 difference map `d = Abs(x_r - x)`를 계산(reconstruction error map).
- **Projection + Filtration**: 3×3 conv로 `d`를 이미지 공간에서 feature 공간(`f_m`)으로 투영한 뒤, Gumbel-sigmoid 함수로 이진 마스크에 가까운 형태로 필터링해 재구성 오차 특유의 배경 노이즈(거의 모든 픽셀에서 약하게 활성화되는 현상)를 억제.
- **Cross-attention 기반 global fusion**: 필터링된 difference feature를 multi-scale로 resize한 뒤, difference feature에서 query, backbone feature에서 key/value를 만들어 전역(pixel-to-pixel) cross-attention으로 융합, 공간적으로 보강된 feature `F_e`를 생성.
- `F_e`는 classification/regression/segmentation 세 헤드를 공유하는 instance segmentation decoder에 그대로 입력되며, 기존 one-stage·two-stage 아키텍처에 plug-and-play로 결합 가능(model-agnostic).

> [!example]- 구현 디테일
> ```
> d = Abs(x_r - x)
> f_m = Conv_3x3(d)
> f_hat_m = 1 / (1 + exp(-(f_m + G1 - G2)/τ))     (Gumbel-sigmoid 이진 필터링, G1,G2 ~ Gumbel(0,1))
> F_d = {Bicubic(f_hat_m, scale=2^l) | l=1..L}     (multi-scale 정렬)
> q = Linear(F_d), k = Linear(F_b), v = Linear(F_b)
> F_e = softmax(q·k^T / sqrt(0.5)) · v              (전역 cross-attention 융합)
> L_total = L_IS + λ·L_OR   (λ=1.0, 두 loss 동등 가중)
> ```

<mark style="background: #FFF9D6A6;">difference map을 그대로 쓰면 재구성 특유의 노이즈로 거의 전 영역이 활성화되는데, Gumbel-sigmoid 필터링이 이를 억제해 실제 정보 손실 신호만 남긴다(Ablation에서 filtration 유무로 AP 53.20→55.20, APseg 47.00→48.00 차이). Concat이나 단순 곱셈이 아닌 전역 cross-attention 융합을 쓴 이유는, 정보 손실 위치와 그 위치를 보강할 backbone feature 간의 관계를 pixel-to-pixel로 전역 탐색해야 소형 객체의 content와 복잡구조 객체의 edge를 동시에 정확히 겨냥해 복원할 수 있기 때문이다.</mark>

또한 이 논문은 PDI라는 UAV 기반 적외선 배전설비 instance segmentation 데이터셋을 함께 제안한다 — 16,596장, 126,570 인스턴스, 7개 설비 카테고리(bushing, insulator, disconnector, arrester, breaker, switch, terminal). 기존 적외선 데이터셋 대비 최대 규모이며 distribution/scale/category 불균형이라는 세 가지 도전 과제를 명시적으로 포함한다.

# 실험 결과

### 핵심 결과 (PDI, Mask RCNN 기준)
| 벤치마크 | 지표 | Before(Mask RCNN) | After(RE-Mask RCNN) |
|---|---|---|---|
| PDI | AP^box / AP^seg (Avg) | 53.50 / 46.80 | 55.20 / 48.00 |
| PDI | Cascade RCNN → RE-Cascade RCNN (AP^box/AP^seg) | 55.60/46.90 | 57.70/47.70 |
| PDI | YOLOACT → RE-YOLOACT (AP^box/AP^seg) | 50.30/40.90 | 52.50/43.10 |

> [!note]- 세부 결과 및 Ablation
> #### Ablation (PDI, Mask RCNN 기준, Table 2)
> | ORD | DFE | Filtration | Avg (AP^box/AP^seg) |
> |---|---|---|---|
> | - | - | - | 53.50/46.80 |
> | ✓ | - | - | 54.10/47.30 |
> | ✓ | ✓ | - | 53.20/47.00 |
> | ✓ | ✓ | ✓ | 55.20/48.00 |
>
> ORD 단독으로 AP^box +1.12%p, AP^seg +1.07%p. DFE를 filtration 없이 추가하면 AP^seg는 개선되지만(+0.43%p) AP^box는 오히려 하락(-0.57%p, 배경 노이즈 유입 추정). filtration까지 갖춰야 AP^box +3.18%p, AP^seg +2.56%p로 최대 개선.
>
> #### 소형/복잡구조 카테고리별 개선 (Table 3)
> - 소형 객체(bushing): RE-Mask RCNN이 Mask RCNN 대비 AP^box +2.00%p, AP^seg +2.86%p.
> - 복잡구조 객체(insulator): RE-Mask RCNN이 AP^box +1.79%p, AP^seg +1.70%p.
> - RE-Cascade RCNN이 PDI 전체에서 AP^box 57.70(최고), RE-YOLOACT가 AP^box +4.37%p·AP^seg +5.38%p로 개선폭 최대.
>
> #### 클래스 불균형 대응 실험 (Table 4, 5.6.1)
> - Category-Balanced Sampling(CBS): RE-Mask RCNN+CBS가 terminal AP^seg 11.70→13.50으로 개선, 주요 카테고리 성능 저하 없음.
> - Loss Re-Weighting(LRW): terminal AP^box 41.00→42.40, AP^seg 11.70→12.90으로 주로 recognition 신뢰도 개선.
> - 두 전략 모두 RE-Mask RCNN 위에서 baseline(RE-Mask RCNN) 대비 추가 개선(CBS: 55.80/48.40, LRW: 55.40/48.30).
>
> #### 공개 데이터셋(HRSeg, 송전설비) 일반화 (Table 6, 5.6.3)
> - RE-Mask RCNN이 Mask RCNN 대비 AP^box +2.80%p, AP^seg +2.10%p. RE-YOLOACT도 AP^box +1.30%p, AP^seg +0.80%p — 배전이 아닌 송전 설비 도메인에서도 일관된 개선.
>
> #### 연산 복잡도 (Table 7, Figure 10)
> - RE-Mask RCNN은 Mask RCNN 대비 ART +8.80ms, 파라미터 +1.04M 증가로 정확도 개선(AP^box +3.18%p, AP^seg +2.56%p) 대비 오버헤드가 작음.
> - Jetson 플랫폼 적합성 분석: YOLOACT/RE-YOLOACT는 Xavier NX급에서 실시간(30 FPS 근접) 가능, Cascade RCNN/RE-Cascade RCNN은 AGX Orin급 고성능 플랫폼이 필요.

# Discussion

### 이 아이디어의 잠재적 부작용
- DFE의 배경 노이즈 유입으로 인한 AP^box 하락 위험 → <mark style="background: #FF5582A6;">filtration 없이 DFE만 추가하면 baseline 대비 AP^box가 오히려 0.57%p 하락(Table 2, row 3) — Gumbel-sigmoid 필터링이 필수적이며, 이것으로도 완전히 해소됐다는 정량적 근거(잔존 노이즈 비율 등)는 제시되지 않음.</mark>
- 재구성 loss와 segmentation loss 간 최적화 충돌 가능성 → <mark style="background: #FF5582A6;">λ=1.0 고정값으로 두 loss를 동등 가중했다고만 서술할 뿐, 가중치 민감도 분석(ablation)은 제시하지 않아 다른 데이터셋에서도 최적인지 불확실.</mark>

### 한계
- <mark style="background: #FF5582A6;">저자가 결론에서 명시: 적외선 영상 특유의 저대비(low contrast)로 인해 객체와 배경이 혼동되는 문제가 여전히 남아있음.</mark>
- <mark style="background: #FF5582A6;">현재 프레임워크는 pruning, quantization, distillation 등 경량화 기법이 전혀 적용되지 않은 상태 — 무거운 아키텍처(Cascade RCNN 계열)는 저전력 UAV 탑재 플랫폼(Jetson Nano/TX2)에서 실시간 구동이 어려움(Table 7, Figure 10).</mark>
- 적외선 영상의 온도 보정(NUC)이나 방사율 차이로 인한 도메인 특이적 노이즈가 difference map 활성화에 미치는 영향은 별도로 분석되지 않음.

### 생각할 점
- <mark style="background: #A6E3A1A6;">이 논문의 ORD+DFE는 [[SR-TOD]]의 self-reconstruction difference map 아이디어를 (1) 단일 레벨(P2) attention에서 multi-level decoder + 전역 cross-attention으로, (2) object detection에서 instance segmentation으로, (3) 가시광/드론 영상에서 적외선 영상으로 확장한 사례로 읽힌다 — 동일 원리가 도메인과 태스크를 넘어 일반화된다는 근거로 볼 수 있다.</mark>
- <mark style="background: #A6E3A1A6;">Gumbel-sigmoid 필터링은 [[SR-TOD]]의 learnable threshold 이진화보다 미분 가능성을 유지하면서 이산적 선택에 가까운 필터링을 한다는 점에서 구현상 진일보한 선택으로 보인다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">[[SR-TOD]]와 이 논문을 동일 벤치마크(예: PDI 또는 AI-TOD)에서 직접 비교하면, "단일 레벨 element-wise attention"과 "multi-level cross-attention global fusion" 중 어느 difference map 활용 방식이 더 효율적인지 검증할 수 있을 것으로 보인다.</mark>
- <mark style="background: #A6E3A1A6;">[[Detection_Oriented_Rectification]]의 "task-oriented rectification"(pixel fidelity가 아닌 detection 지향 복원) 철학을 이 프레임워크에 적용하면, 현재 pixel-wise L1 reconstruction loss만 쓰는 ORD를 instance segmentation에 더 특화된 형태로 개선할 여지가 있다.</mark>

# 관련 개념
- [[Self_Reconstruction_Difference_Map]] — 이 논문의 ORD+DFE가 확장·적용하는 원조 개념. Reconstruction 대상(단일 FPN 레벨 → multi-level decoder), 융합 방식(element-wise attention → 전역 cross-attention), 도메인(가시광/드론 → 적외선), 태스크(detection → instance segmentation)를 모두 확장한 사례로 "등장 논문"에 추가.

# 관련 문서
- 비교 후보: [[SR-TOD]] — 동일한 reconstruction difference 기반 feature 강화 원리를 공유하지만 태스크(detection vs segmentation)와 도메인(가시광/드론 vs 적외선)이 달라 직접 비교 문서는 아직 만들지 않음. 추후 같은 원리를 공유하는 논문이 더 쌓이면 [[Small_Object_Detection_Approaches]]와 별개로 "Reconstruction 기반 Feature 강화" 비교 문서를 만들 근거가 될 수 있다.

# 읽어볼 만한 논문
- 참고문헌 기반: B. Cao, H. Yao, P. Zhu, Q. Hu, "Visible and clear: Finding tiny objects in difference map" (ECCV 2024) [26, 인용상 논문 자체 언급은 아니나 이 논문의 참고문헌 26번 Liu et al. "Tiny object detection ... object reconstruction"과 함께 SR-TOD 계열 원리의 직접적 배경] — 이미 위키에 [[SR-TOD]]로 존재. Reconstruction difference를 활용하는 원조 격 논문이라 이 논문의 ORD+DFE 설계와 직접 비교하며 읽기 좋음.
- 참고문헌 기반: D. Li, Y. Sun, Z. Zheng, F. Zhang, B. Sun, C. Yuan, "A real-world large-scale infrared image dataset and multitask learning framework for power line surveillance" (IEEE Trans. Instrum. Meas. 2025) [8] — 이 논문이 related work에서 직접 비교하는 SR+instance segmentation 멀티태스크 프레임워크(PowerNet). 실험 표에서도 baseline으로 등장해, ORD/DFE 접근과 SR 기반 접근의 실증적 차이를 이해하는 데 도움.
- 참고문헌 기반: K. P. Alexandridis, J. Deng, A. Nguyen, S. Luo, "Long-tailed instance segmentation using Gumbel optimized loss" (ECCV 2022) [25] — 이 논문의 DFE 모듈이 Gumbel-sigmoid 필터링을 도입할 때 근거로 삼은 원 기법. Gumbel 기반 이산 근사가 어떻게 instance segmentation에 쓰이는지 배경 이해에 필요.
- 자유 추천(검증 필요): 적외선/열화상 영상에서의 도메인 특이적 노이즈(NUC 보정 잔여 오차, 방사율 차이)가 reconstruction 기반 anomaly/difference 신호에 미치는 영향을 다룬 연구 — 검색 키워드: `infrared thermal image non-uniformity correction residual noise reconstruction-based detection`. 이 논문의 한계로 지적한 "적외선 특유의 저대비·노이즈 문제"를 더 깊이 이해하려면 참고할 만함.

Project: [[논문 읽기|논문 읽기]]
