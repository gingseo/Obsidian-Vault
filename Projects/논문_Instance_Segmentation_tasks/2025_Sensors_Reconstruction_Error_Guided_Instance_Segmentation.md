---
pm-task: true
projectId: "paperwiki-instance-segmentation"
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
customFields:
  "q7pe2swemtck1e8u": 2025
  "yihe8zz0mtck1e8v": "Sensors (MDPI)"
subtaskIds: []
dependencies: []
year: 2025
venue: "Sensors (MDPI)"
jcr_quartile: Q2
task: [instance-segmentation]
direction: [novel-approach]
paper_tags: [paper, instance-segmentation, infrared-inspection, reconstruction-error, feature-enhancement, uav, power-distribution]
source: "Projects/_pdf/Instance_Segmentation/2025_Sensors_Reconstruction-Error-Guided-Instance-Segmentation.pdf"
source_type: personal
createdAt: "2026-08-19T11:16:30.000Z"
updatedAt: "2026-08-31T00:00:00.000Z"
---

#paper #instance-segmentation #infrared-inspection #reconstruction-error #feature-enhancement #uav #power-distribution

> [!quote] 원제
> **Reconstruction Error Guided Instance Segmentation for Infrared Inspection of Power Distribution Equipment**
> Jinbin Luo, Yi Sun, Jian Zhang, Bin Sun — CSG Guangdong Power Grid Corporation / Hunan University, Sensors (MDPI) 2025
> https://doi.org/10.3390/s25196007

# 한 줄 요약
<mark style="background: #FFF3A3A6;">적외선 배전설비 영상에서 backbone feature로부터 원본 이미지를 재구성해 정보 손실 위치를 진단하는 Object Reconstruction Decoder(ORD)와, 그 재구성 오차(difference map)를 Gumbel-sigmoid로 필터링한 뒤 전역 cross-attention으로 backbone feature에 재주입하는 Difference Feature Enhancement(DFE) 모듈을 결합한 model-agnostic instance segmentation 프레임워크(RE), 그리고 이를 검증할 최초의 UAV 기반 적외선 배전설비 데이터셋 PDI.</mark>

> [!info] 내 메모
> 

# 정리

## 기존 방법의 한계
- **소형 객체 content 압축**:
  Bushing 등 배전설비는 UAV 촬영 거리에서 픽셀 수가 극히 적은 소형 객체가 되는데, backbone의 반복적 downsampling으로 feature가 수 픽셀에서 1픽셀 이하로 압축되어 노이즈·배경과 구분이 어려워진다.
- **복잡구조 객체 edge 단편화**:
  Insulator·disconnector처럼 형상이 복잡한 설비는 downsampling의 이산화(discretization) 과정에서 경계가 끊어진 점들의 나열로 조각나(jagged/fragmented), 구조적 연속성이 깨진다.

## 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — 정보 손실 완화(해상도 복원 계열)**
- Super-Resolution(SR)[15,16]: 저해상도 영상에서 고해상도 대응물을 복원 — 존재하지 않는 texture·artifact 생성 위험, 연산 부담 증가.
- 개선된 Down-sampling(IDS)[17,18]: 샘플링 영역 내 픽셀 가중합으로 정보 보존 — 과정 자체가 laborious하고 손실을 "줄일" 뿐 손실 위치를 진단하지 않음.
- **타겟/해결**: 소형 객체 content 압축(문제①) — 정보 손실 완화 시도에 머물 뿐, "어디서 얼마나 손실됐는가"를 명시적 신호로 만들어 feature 강화에 직접 쓰지 않음.

**갈래 2 — 명시적 spatial prior/attention 강화**
- Zhao et al.[22]: category-specific image patch를 spatial prior로 사용 — 시각적으로 유사한 카테고리 구분에 초점.
- Zhou et al.[14]: shuffle-polarized self-attention으로 장거리 의존성 포착.
- Li et al.[8]: SR과 instance segmentation을 결합한 멀티태스크 프레임워크 — SR 계열의 근본 한계(artifact, 연산 비용) 공유.
- Li et al.[23]: spatial transformation network로 multi-scale spatial context 추출.
- **타겟/해결**: 복잡구조 객체 edge 단편화(문제②) — 네 접근 모두 정보 손실의 위치·정도를 직접 신호화하지 않음.

**갭**: <mark style="background: #FFF3A3A6;">두 갈래 모두 정보 손실 완화 시도에 머물 뿐, "어디서 얼마나 손실됐는가"를 명시적 신호로 만들어 feature 강화에 직접 쓰지 않는다는 공통 빈틈이 남는다.</mark>

## 이 논문이 풀고자 하는 문제
1. 정보 손실의 위치·정도를 명시적으로 진단해 소형 객체의 content를 보강하는 것.
2. 정보 손실의 위치·정도를 명시적으로 진단해 복잡구조 객체의 edge 연속성을 보강하는 것.

**갭 종합**: <mark style="background: #FFF3A3A6;">소형 객체 content 압축과 복잡구조 객체 edge 단편화는 서로 다른 문제처럼 보이지만, 둘 다 backbone downsampling이 초래하는 "정보 손실"이라는 같은 원인에서 나온다. 이 논문의 통찰은 정보 손실을 완화하려 시도하는 대신, 원본 이미지를 재구성해보고 그 재구성이 실패하는 정도(reconstruction error)를 손실의 위치·정도를 알려주는 self-supervised 신호로 명시적으로 활용하는 것이다.</mark>

> [!info] 내 메모
> 

# 해결 방법 요약

| | 문제 ① — 소형 객체 content 압축 | 문제 ② — 복잡구조 객체 edge 단편화 |
|---|---|---|
| **해결 방법** | ORD로 원본 이미지를 재구성 — 정보가 심하게 손실된 영역일수록 복원이 어려워 재구성 오차(difference map)가 커지는 성질을 "손실 진단서"로 삼아 DFE가 backbone feature에 재주입 | 재구성 loss가 픽셀 단위 오차에 민감해 edge 근방에서도 오차가 크게 나타나므로, 같은 difference map이 edge 단편화 영역까지 함께 진단·보강 |
| **예상되는 문제점** | Difference map이 재구성 특유의 노이즈로 거의 전 영역에서 약하게 활성화되어, 필터링 없이 쓰면 배경 노이즈가 함께 유입된다(아래 "제안 방법" ② 참고). | 재구성 loss(L_OR)와 segmentation loss(L_IS)를 λ=1.0 고정 가중치로 단순 합산해, 두 태스크 간 최적화 충돌 여부를 별도로 분석하지 않는다. |

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">Backbone feature로부터 원본 입력 이미지를 재구성하는 <span style="color:#c0392b; font-weight:bold;">Object Reconstruction Decoder(ORD)</span>를 두고, 재구성 이미지와 원본의 차이(difference map)를 Gumbel-sigmoid로 이진 필터링한 뒤 backbone feature와 전역 cross-attention으로 융합하는 <span style="color:#c0392b; font-weight:bold;">Difference Feature Enhancement(DFE)</span> 모듈로 정보 손실을 보강한다. 두 모듈 모두 별도 라벨 없이 원본 이미지 자체를 supervision으로 쓰는 self-supervised 구조이며, 기존 one-stage·two-stage instance segmentation 아키텍처에 plug-and-play로 결합 가능(model-agnostic)하다.</mark>

## 전체 파이프라인 (Fig. 2 기준)

```
입력 적외선 이미지 x (H, W, 3)
       │
       ▼
Shared Encoder Ψ_enc (backbone + FPN)         → F_b = {f_l ∈ (H/2^l, W/2^l, C) | l=1..L}   [multi-scale backbone feature]
       │
       ├──────────────────────────────────────────────┐
       ▼                                                ▼
① Object Reconstruction Decoder Ψ_rdec         → x_r (H, W, 3)   [재구성 이미지, object-centric]
       │
       ▼ (x_r과 원본 x의 차이)
② Difference Feature Enhancement Ψ_st
     (a) difference map d = Abs(x_r - x)        → d (H, W, 3)
     (b) projection + Gumbel-sigmoid filtration → f̂_m (H, W, C)   [배경 노이즈 억제된 binary-ish mask]
     (c) multi-scale 정렬 + cross-attention 융합 → F_e = {f_e ∈ (H/2^l, W/2^l, C) | l=1..L}   [spatially-enhanced feature]
       │
       ▼
③ Instance Segmentation Decoder Ψ_sdec (classification / regression / segmentation 3-branch, 기존 detector 재사용)
       │
       ▼
출력: 카테고리 c, 바운딩박스 b, 인스턴스 마스크 m
```

> [!info] 내 메모
> 

### ① Object Reconstruction Decoder (ORD)
- **역할**:
  Backbone의 downsampling으로 정보가 손실된 위치·정도를 진단하기 위해, multi-scale backbone feature `F_b`로부터 원본 해상도의 이미지를 재구성한다. 정보가 많이 손실된 객체 영역일수록 복원이 어려워지므로, 재구성 결과 자체가 손실의 위치·정도를 반영하는 신호가 된다.
- **구현**:
  Encoder와 대칭적인 구조로, 여러 decoding group을 순차적으로 쌓는다. 각 decoding group은 3×3 conv → ReLU → bicubic upsample(×2)로 구성되고, 대응하는 backbone feature와 skip-connection처럼 element-wise로 더해진다. 재구성은 object-centric — object 영역은 원본 픽셀값을 보존하고 비object 영역은 0으로 억제한 뒤, 이 마스킹된 타깃에 대해서만 loss를 계산.
- **입출력 shape**:
  Backbone feature `F_b = {f_l ∈ (H/2^l, W/2^l, C)}` → `L`번의 decoding group을 거쳐 `x_r ∈ (H, W, 3)` (입력과 동일 해상도).

```python
# 논문 Eq.(5) 기반. l=1..L은 feature level, m=L-l+1
f_{m+1} = Bicubic(ReLU(Conv_3x3(f_m)), scale=2) + f_l   # decoding group, 마지막 layer만 출력 채널 3
# L번 반복 후 x_r 생성
```

<mark style="background: #FFF9D6A6;">Object 영역만 재구성하도록 supervision을 마스킹해(배경 제외), SR처럼 존재하지 않는 texture를 만들어내지 않고도 "정리" 표의 정보 손실 진단에 필요한 신호만 효율적으로 얻는다. 픽셀 단위 오차(MAE)에 민감한 재구성 특성상, content가 압축된 소형 객체와 edge가 끊긴 복잡구조 객체 모두에서 복원이 어려워지므로, 같은 difference map이 두 문제 유형을 동시에 드러내는 공통 신호가 된다(Ablation에서 ORD 단독 추가만으로 AP^box 53.50→54.10, AP^seg 46.80→47.30).</mark>

> [!info] 내 메모
> 

### ② Difference Feature Enhancement (DFE)
- **역할**:
  ORD가 만든 재구성 오차(difference map)를 "정보 손실 위치가 표시된 지도"로 활용해, backbone feature에 다시 주입함으로써 손실된 spatial 표현을 보강한다.
- **구현**:
  세 단계로 구성.
  1) **Projection + Filtration**: `d = Abs(x_r - x)`로 difference map을 계산한 뒤 3×3 conv로 feature 공간에 투영(`f_m`), Gumbel-sigmoid 함수로 이진 마스크에 가까운 형태(`f̂_m`)로 필터링해 재구성 특유의 배경 노이즈(거의 전 픽셀에서 약하게 활성화되는 현상)를 억제.
  2) **Multi-scale 정렬**: bicubic interpolation으로 `f̂_m`을 backbone feature 계층과 같은 scale factor(2¹, 2², ..., 2^l)로 리샘플링.
  3) **Cross-attention 융합**: difference feature에서 query, backbone feature에서 key/value를 [[1x1_Convolution|선형변환]]으로 만들어 [[Multi_Head_Self_Attention|attention]] 방식의 전역(pixel-to-pixel) 유사도 가중합으로 융합.
- **입출력 shape**:
  `d ∈ (H, W, 3)` → filtration 후 `f̂_m ∈ (H, W, C)` → multi-scale 정렬 후 `F_d = {f_d ∈ (H/2^l, W/2^l, C)}` → backbone feature `F_b`와 cross-attention 융합 → `F_e = {f_e ∈ (H/2^l, W/2^l, C)}` (backbone feature와 동일 shape, 값만 보강됨).

```python
# 논문 Eq.(8)-(13) 기반
d = Abs(x_r - x)                                          # difference map
f_m = Conv_3x3(d)                                         # 이미지 공간 -> feature 공간 투영
f_hat_m = 1 / (1 + exp(-(f_m + G1 - G2) / tau))            # Gumbel-sigmoid 이진 필터링, G1,G2 ~ Gumbel(0,1)
F_d = [Bicubic(f_hat_m, scale=2**l) for l in range(1, L+1)]  # multi-scale 정렬

q = Linear(F_d)      # difference feature에서 query
k = Linear(F_b)      # backbone feature에서 key
v = Linear(F_b)      # backbone feature에서 value
F_e = softmax(q @ k.T / sqrt(0.5)) @ v                     # 전역 cross-attention 융합
```

<mark style="background: #FFF9D6A6;">Difference map을 필터링 없이 그대로 쓰면 재구성 특유의 노이즈로 거의 전 영역이 활성화되는데, Gumbel-sigmoid가 이를 억제해 실제 정보 손실 신호만 남긴다. Concat이나 단순 곱셈이 아닌 전역 cross-attention 융합을 쓴 이유는, 정보 손실 위치와 그 위치를 보강할 backbone feature 간의 관계를 pixel-to-pixel로 전역 탐색해야 소형 객체 content와 복잡구조 객체 edge를 동시에 정확히 겨냥해 복원할 수 있기 때문이다(Ablation에서 filtration 유무로 AP^box 53.20→55.20, AP^seg 47.00→48.00 차이).</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> Filtration 없이 DFE만 추가하면 baseline 대비 AP^seg는 개선되지만(+0.43%p) AP^box는 오히려 0.57%p 하락한다(Table 2, row 3) — DFE가 배경 노이즈를 함께 끌어들일 수 있음을 논문이 직접 인정하며, Gumbel-sigmoid 필터링이 필수적임을 보여준다.

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| Shared Encoder | (H, W, 3) | F_b = {(H/2^l, W/2^l, C)} | 이미지 → multi-scale feature | Backbone(ResNet 등) + FPN |
| ① ORD | F_b | x_r (H, W, 3) | 원본 이미지 재구성 → 정보 손실 진단 | 대칭 decoding group(3×3conv+ReLU+bicubic ×L), object-centric masking |
| ② DFE | d(H,W,3) + F_b | F_e = {(H/2^l, W/2^l, C)} | difference map 필터링 + 전역 융합으로 feature 보강 | Conv+Gumbel-sigmoid 필터링 + cross-attention |
| ③ Instance Seg. Decoder | F_e | 카테고리 c + 박스 b + 마스크 m | 최종 예측 (기존 detector 헤드 재사용) | Classification/Regression/Segmentation 3-branch |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table 2 (Ablation), Table 3 (PDI 전체 비교, Mask RCNN 기준)
**표를 보는 법**: Table 2는 ORD/DFE/Filtration 조합별 AP^box/AP^seg(Avg)를, Table 3은 PDI 데이터셋에서 baseline 대비 RE(제안 프레임워크) 결합 모델의 전체 성능을 비교한다.

| 벤치마크 | 지표 | Before | After |
|---|---|---|---|
| PDI (Table 2, Mask RCNN) | AP^box / AP^seg (Avg) | 53.50 / 46.80 | 55.20 / 48.00 (ORD+DFE+Filtration) |
| PDI (Table 3, Cascade RCNN → RE-Cascade RCNN) | AP^box / AP^seg | 55.60 / 46.90 | 57.70 / 47.70 |
| PDI (Table 3, YOLOACT → RE-YOLOACT) | AP^box / AP^seg | 50.30 / 40.90 | 52.50 / 43.10 |

> [!note]- 세부 결과 및 Ablation
> #### Table 2 — 모듈별 Ablation (PDI, Mask RCNN 기준)
> **보는 법**: ORD/DFE/Filtration을 하나씩 추가하며 AP^box/AP^seg 변화를 관찰.
>
> | ORD | DFE | Filtration | AP^box/AP^seg (Avg) |
> |---|---|---|---|
> | | | | 53.50/46.80 |
> | ✓ | | | 54.10/47.30 |
> | ✓ | ✓ | | 53.20/47.00 |
> | ✓ | ✓ | ✓ | 55.20/48.00 |
>
> ORD 단독으로 AP^box +1.12%p, AP^seg +1.07%p. DFE를 filtration 없이 추가하면 AP^seg는 개선되지만(+0.43%p) AP^box는 하락(-0.57%p, 배경 노이즈 유입 추정). Filtration까지 갖춰야 AP^box +3.18%p, AP^seg +2.56%p로 최대 개선.
>
> #### Table 3 — PDI 전체 정량 비교
> **보는 법**: baseline(Mask RCNN/Cascade RCNN/YOLOACT/SOLOv2)과 RE-결합 버전을 카테고리별(bushing/insulator/disconnector/arrester/breaker/switch/terminal)로 비교.
> RE-Cascade RCNN이 AP^box 57.70/AP^seg 47.70으로 원본 대비 +3.78%/+1.71%p, RE-YOLOACT는 AP^box 52.50/AP^seg 43.10으로 +4.37%/+5.38%p. 소형 객체(bushing)는 RE-Mask RCNN이 AP^box +2.00%p·AP^seg +2.86%p, 복잡구조(insulator)는 AP^box +1.79%p·AP^seg +1.70%p 개선.
>
> #### Table 4 — 클래스 불균형 대응 (CBS/LRW, PDI)
> **보는 법**: Category-Balanced Sampling(CBS)과 Loss Re-Weighting(LRW)을 RE-Mask RCNN 위에 추가 적용했을 때 tail 카테고리(terminal, arrester) 개선폭.
> CBS: RE-Mask RCNN+CBS가 terminal AP^seg 11.70→13.50, arrester AP^box 44.60→45.10, 주요 카테고리 저하 없이 Avg 55.80/48.40. LRW: terminal AP^box 41.00→42.40, AP^seg 11.70→12.90, Avg 55.40/48.30 — 주로 recognition 신뢰도 개선.
>
> #### Table 5 — Recall/F1-score (PDI)
> **보는 법**: 카테고리별 recall/F1-score, RE-계열이 baseline 대비 recall을 얼마나 올리는지 확인.
> RE-Mask RCNN 평균 recall 84.33(Mask RCNN 대비 상승), terminal(+3.28)·arrester(+1.31)에서 개선폭 큼. RE-Cascade RCNN은 recall +2.70(switch +4.80)이나 F1-score는 81.80→81.70으로 소폭 하락(precision-recall trade-off). RE-YOLOACT·RE-SOLOv2는 insulator/disconnector에서 baseline과 거의 동일(제한적 개선).
>
> #### Table 6 — HRSeg(송전설비) 데이터셋 일반화
> **보는 법**: 배전이 아닌 송전(power transmission) 공개 데이터셋에서도 RE 프레임워크가 일관되게 개선되는지 확인.
> RE-Mask RCNN이 Mask RCNN 대비 AP^box +2.80(48.90→51.70), AP^seg +2.10(39.50→41.60). RE-YOLOACT는 AP^box +1.30(43.30→44.60), AP^seg +0.80(30.60→31.40).
>
> #### Table 7, Fig.10 — 연산 복잡도 & Jetson 플랫폼 적합성
> **보는 법**: ART(ms)·파라미터(M)·FPS를 baseline과 RE-결합 모델 간 비교, Fig.10은 이를 Jetson Nano/TX2(<10FPS)·Xavier NX(10~30FPS)·AGX Orin(>30FPS) 영역에 매핑.
> RE-Mask RCNN은 Mask RCNN 대비 ART +8.80ms, 파라미터 +1.04M(44.00M→45.04M)로 AP^box +3.18%p·AP^seg +2.56%p 개선. YOLOACT/RE-YOLOACT는 Xavier NX급에서 실시간 근접(각각 23.80ms/34.01FPS, 31.00ms/32.26FPS 영역), Cascade RCNN/RE-Cascade RCNN은 AGX Orin급 고성능 플랫폼 필요(각각 26.11/22.32 FPS).
>
> #### Fig.11 — ORD·DFE 시각화
> **보는 법**: 입력 → difference map → DFE 전/후 feature map → baseline/제안 방법 instance mask 순서로, difference map의 고오차 영역이 소형 객체·구조적 불연속 위치와 일치하는지, DFE 이후 마스크가 얼마나 정교해지는지 눈으로 확인.
> 소형 insulator는 content가 보강되고, disconnector의 끊긴 edge가 이어지는 것을 시각적으로 확인 가능.

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- DFE의 배경 노이즈 유입으로 인한 AP^box 하락 위험 → <mark style="background: #FF5582A6;">Filtration 없이 DFE만 추가하면 baseline 대비 AP^box가 오히려 0.57%p 하락(Table 2, row 3) — Gumbel-sigmoid 필터링이 필수적이며, 잔존 노이즈가 완전히 해소됐다는 정량적 근거는 제시되지 않는다.</mark>
- 재구성 loss와 segmentation loss 간 최적화 충돌 가능성 → <mark style="background: #FF5582A6;">λ=1.0 고정값으로 두 loss를 동등 가중했다고만 서술할 뿐, 가중치 민감도 분석(ablation)은 제시하지 않아 다른 데이터셋에서도 최적인지 불확실하다.</mark>

### 한계
- <mark style="background: #FF5582A6;">저자가 결론에서 명시: 적외선 영상 특유의 저대비(low contrast)로 인해 객체와 배경이 혼동되는 문제가 여전히 남아있음.</mark>
- <mark style="background: #FF5582A6;">현재 프레임워크는 pruning, quantization, distillation 등 경량화 기법이 전혀 적용되지 않은 상태 — 무거운 아키텍처(Cascade RCNN 계열)는 저전력 UAV 탑재 플랫폼(Jetson Nano/TX2)에서 실시간 구동이 어려움(Table 7, Fig.10).</mark>
- RE-YOLOACT·RE-SOLOv2는 insulator·disconnector 등 일부 카테고리에서 recall 개선이 baseline과 거의 동일해(Table 5), 아키텍처에 따라 프레임워크의 효과가 제한적일 수 있음.

### 생각할 점
- <mark style="background: #A6E3A1A6;">이 논문의 ORD+DFE는 [[2024_ECCV_SR-TOD|SR-TOD]]의 self-reconstruction difference map 아이디어를 (1) 단일 레벨 attention에서 multi-level decoder + 전역 cross-attention으로, (2) object detection에서 instance segmentation으로, (3) 가시광/드론 영상에서 적외선 영상으로 확장한 사례로 읽힌다 — 동일 원리가 도메인·태스크를 넘어 일반화된다는 근거로 볼 수 있다.</mark>
- <mark style="background: #A6E3A1A6;">Gumbel-sigmoid 필터링은 미분 가능성을 유지하면서 이산적 선택에 가까운 필터링을 한다는 점에서, 단순 fixed threshold 이진화보다 학습 친화적인 설계로 보인다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">[[2024_ECCV_SR-TOD|SR-TOD]]와 이 논문을 동일 벤치마크(예: PDI 또는 AI-TOD)에서 직접 비교하면, "단일 레벨 element-wise attention"과 "multi-level cross-attention 전역 융합" 중 어느 difference map 활용 방식이 더 효율적인지 검증할 수 있을 것으로 보인다.</mark>
- <mark style="background: #A6E3A1A6;">[[2026_TPAMI_Detection_Oriented_Rectification|Detection_Oriented_Rectification]]의 "task-oriented rectification"(pixel fidelity가 아닌 detection 지향 복원) 철학을 이 프레임워크에 적용하면, 현재 pixel-wise L1 reconstruction loss만 쓰는 ORD를 instance segmentation에 더 특화된 형태로 개선할 여지가 있다.</mark>

> [!info] 내 메모
> 

# 관련 개념
- [[Self_Reconstruction_Difference_Map]] — 이 논문의 ORD+DFE가 확장·적용하는 원조 개념. Reconstruction 대상(단일 FPN 레벨 → multi-level decoder), 융합 방식(element-wise attention → 전역 cross-attention), 도메인(가시광/드론 → 적외선), 태스크(detection → instance segmentation)를 모두 확장한 사례로 "등장 논문"에 이미 반영되어 있음(내용 검증 후 그대로 재사용).
- [[Multi_Head_Self_Attention]] — DFE의 cross-attention 융합에 쓰인 query-key-value 유사도 가중합 연산의 기반.
- [[1x1_Convolution]] — DFE의 query/key/value 선형변환에 해당하는 채널 변환 연산.

# 관련 문서
- 비교 후보: [[2024_ECCV_SR-TOD|SR-TOD]] — 동일한 reconstruction difference 기반 feature 강화 원리를 공유하지만 태스크(detection vs segmentation)와 도메인(가시광/드론 vs 적외선)이 달라 직접 비교 문서는 아직 만들지 않음. 추후 같은 원리를 공유하는 논문이 더 쌓이면 별도 비교 문서를 만들 근거가 될 수 있다.

# 읽어볼 만한 논문
- 참고문헌 기반: D. Li, Y. Sun, Z. Zheng, F. Zhang, B. Sun, C. Yuan, "A real-world large-scale infrared image dataset and multitask learning framework for power line surveillance" (IEEE Trans. Instrum. Meas. 2025) [8] — 이 논문이 related work에서 직접 비교하는 SR+instance segmentation 멀티태스크 프레임워크. 실험 표에서도 HRSeg 데이터셋 출처로 등장해, ORD/DFE 접근과 SR 기반 접근의 실증적 차이를 이해하는 데 도움.
- 참고문헌 기반: K. P. Alexandridis, J. Deng, A. Nguyen, S. Luo, "Long-tailed instance segmentation using Gumbel optimized loss" (ECCV 2022) [25] — 이 논문의 DFE 모듈이 Gumbel-sigmoid 필터링을 도입할 때 근거로 삼은 원 기법. Gumbel 기반 이산 근사가 어떻게 instance segmentation에 쓰이는지 배경 이해에 필요.
- 참고문헌 기반: J. Zhou, L. Liu, G. Gu, Y. Wen, S. Chen, "A box-supervised instance segmentation method for insulator infrared images based on shuffle polarized self-attention" (IEEE Trans. Instrum. Meas. 2023) [14] — related work에서 비교되는 shuffle-polarized self-attention 기반 복잡구조 객체 강화 접근. ORD+DFE와 다른 방식(명시적 attention vs reconstruction 기반 진단)으로 같은 문제를 푸는 대조군.
- 자유 추천(검증 필요): 적외선/열화상 영상에서의 도메인 특이적 노이즈(NUC 보정 잔여 오차, 방사율 차이)가 reconstruction 기반 anomaly/difference 신호에 미치는 영향을 다룬 연구 — 검색 키워드: `infrared thermal image non-uniformity correction residual noise reconstruction-based detection`. 이 논문의 한계로 지적한 "적외선 특유의 저대비·노이즈 문제"를 더 깊이 이해하려면 참고할 만함.
