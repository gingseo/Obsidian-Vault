---
pm-task: true
projectId: "paperwiki-small-object-detection"
parentId:
id: "t-sr-tod-mx52anrm32"
title: "Visible and Clear: Finding Tiny Objects in Difference Map"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-05-18"
due:
progress: 0
assignees: []
tags: []
customFields:
  "7l8l795xmtcjvlf6": 2024
  "1frf59rymtcjvske": "ECCV"
subtaskIds: []
dependencies: []
year: 2024
venue: "ECCV"
jcr_quartile: "Q1"
task: [object-detection]
direction: [novel-approach]
paper_tags: [paper, small-object-detection, self-reconstruction, difference-map, feature-enhancement, anti-uav]
source: "Projects/_pdf/Object_Detection/2Stage/2024_ECCV_SR-TOD.pdf"
source_type: personal
createdAt: "2026-08-18T11:00:00.000Z"
updatedAt: "2026-08-31T00:00:00.000Z"
---

#paper #small-object-detection #self-reconstruction #difference-map #feature-enhancement #anti-uav

> [!quote] 원제
> **Visible and Clear: Finding Tiny Objects in Difference Map**
> Bing Cao, Haiyu Yao, Pengfei Zhu, Qinghua Hu — College of Intelligence and Computing / Tianjin Key Lab of Machine Learning, Tianjin University, ECCV 2024
> https://arxiv.org/abs/2405.11276

# 한 줄 요약
<mark style="background: #FFF3A3A6;">탐지기 neck(FPN)의 최하위 feature map(P2)에서 원본 이미지를 복원하는 self-reconstruction head를 붙이고, 원본과 재구성 이미지의 차이(difference map)가 정보 손실이 심한 영역=tiny object 위치와 강하게 상관된다는 최초 발견을 바탕으로, 이를 별도 supervision 없이 tiny object feature 강화의 prior로 쓰는 SR-TOD 프레임워크.</mark>

> [!info] 내 메모
> 

# 정리

## 기존 방법의 한계
- **정보 손실(Information Loss)**:
  Backbone의 downsampling은 노이즈 활성화를 줄이고 feature map 해상도를 낮추는 과정에서 tiny object의 신호를 필연적으로 지운다. AI-TOD 기준 2~8픽셀의 "very tiny" 물체는 feature map에서 신호가 거의 사라진다(Fig. 1) — 이것이 tiny object detection의 가장 근본적인 원인으로 지목된다.
- **생성 기반 강화의 부작용**:
  다수 선행 연구가 GAN 기반 super-resolution으로 저해상도-고해상도 쌍을 학습시켜 왜곡된 구조를 복원하려 하지만, 존재하지 않는 texture와 artifact를 만들어내(spurious textures and artifacts) 오히려 탐지 성능을 떨어뜨린다.

## 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — FPN 구조 재설계**
- FPN[30], PANet[33]: bidirectional path로 feature hierarchy 강화 — 구조 개선일 뿐 정보 손실 자체를 진단하지 않음.
- NAS-FPN[17]·BiFPN[48]·Recursive-FPN[38]: FPN 구조 자체의 변형.
- Gong et al.[19]: 인접 레벨 간 fusion factor 조정.
- QueryDet[54]: cascaded sparse query로 고해상도 feature를 빠르게 활용.
- **타겟/해결**: 정보 손실(①) — 구조를 재설계할 뿐 "정보 손실이 어디서, 얼마나 일어났는지"는 진단하지 않는다.

**갈래 2 — 생성 기반 feature imitation/super-resolution**
- <mark style="background: #FFF3A3A6;">Perceptual GAN[28], SOD-MTGAN[1,2]: GAN으로 tiny object super-resolution 수행.</mark>
- Noh et al.[37]: dilated convolution으로 고/저해상도 feature 간 receptive field mismatch 완화.
- Deng et al.[14]: feature texture transfer 모듈로 새 feature layer에 디테일 정보 채움.
- **타겟/해결**: <mark style="background: #FFF3A3A6;">정보 손실(①)을 "생성"으로 메우려 하지만, 없던 자리에 그럴듯한 texture를 새로 지어내는 셈이 되어 오히려 spurious artifact라는 새 부작용(②)을 낳는다.</mark>

**갭**: <mark style="background: #FFF3A3A6;">두 갈래 모두 "정보 손실이 어디서, 얼마나 일어났는지를 명시적으로 진단"하지 않는다 — FPN 계열은 구조만 재설계하고, 생성 기반 계열은 진단 없이 곧바로 복원을 시도해 spurious artifact를 만든다.</mark>

## 이 논문이 풀고자 하는 문제
1. 생성(generation) 없이, 이미지 재구성의 어려움 자체를 신호로 삼아 정보 손실이 심한 영역(=tiny object가 있을 가능성이 높은 영역)을 찾아내는 것.
2. 이렇게 찾은 영역 정보(prior)를 실제 detection feature 강화에 어떻게 반영할지 설계하는 것.

**갭 종합**: <mark style="background: #FFF3A3A6;">두 문제 모두 "정보 손실이 어디서, 얼마나 일어났는지를 명시적으로 진단하지 않는다"는 공통 원인에서 나온다. 이미지 재구성이라는 low-level task가 픽셀 변화에 민감하다는 성질을 이용하면, "복원이 어려운 영역=정보가 심하게 손실된 영역"이라는 신호를 별도 생성 없이 얻어낼 수 있다는 것이 SR-TOD의 통찰이다.</mark>

> [!info] 내 메모
> 

# 해결 방법 요약

| | 문제 ① — 정보 손실(Information Loss) | 문제 ② — 생성 기반 강화의 부작용 |
|---|---|---|
| **해결 방법** | 저수준 feature map(P2)으로부터 원본 이미지를 복원하는 reconstruction head를 붙여, 복원이 어려운 영역=정보 손실이 심한 영역이라는 difference map을 얻는다 | Difference map을 element-wise attention 형태로 P2에 곱해 tiny object feature를 선택적으로 강화한다(DGFE) |
| **예상되는 문제점** | 재구성 task 추가로 인한 학습 비용/불안정성, 추론 시 연산 오버헤드가 정량화되지 않음 | Difference map이 정보 손실이 아닌 다른 이유(복잡한 텍스처, 반복 패턴)로 활성화될 수 있어 노이즈 필터링이 별도로 필요함 |

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">탐지 모델의 FPN에서 나온 저수준 feature map(P2)으로부터 원본 이미지를 복원하는 <span style="color:#c0392b; font-weight:bold;">self-reconstruction head</span>를 붙이고, 원본과 재구성 이미지의 차이인 <span style="color:#c0392b; font-weight:bold;">difference map</span>을 별도 supervision 없이 tiny object 위치·구조에 대한 prior로 활용해 P2 feature를 attention 형태로 강화한다.</mark>

## 전체 파이프라인 (Fig. 2 기준)

```
입력 이미지 I_o (3, H, W)
       │
       ▼
Backbone + Neck(FPN)                       → P2 (C, H/4, W/4), P3~P6
       │
       ├──────────────────────────────────┐
       ▼                                    ▼
① Reconstruction Head (RH)                 (P2는 DGFE로도 전달)
   Up Block ×2 → 재구성 이미지 I_r          → (3, H, W)
       │
       ▼
② Difference Map 산출                      → D (1, H, W)
   D = Mean_channel(|I_r - I_o|)
       │
       ▼
③ DGFE — Filtration + Reweighting          → P2' = M ⊗ P2   (C, H/4, W/4)
   (D를 이진화·리사이즈 + P2 채널 가중치)
       │
       ▼
P2'가 원래 P2를 대체해 P3~P6과 함께 Detection Head로 전달
```

> [!info] 내 메모
> 

### ① Self-Reconstruction Head (RH)

- **역할**:
  FPN의 최하위 feature map P2(대부분의 generic detector에서 tiny object detection을 담당하는 레벨)로부터 원본 이미지를 복원해, "이 위치의 정보가 backbone을 거치며 얼마나 손실됐는지"를 대리 측정할 수단을 만든다. 이미지 재구성은 픽셀 변화에 극도로 민감한 low-level task이므로, 구조·텍스처 정보가 심하게 손실된 영역일수록 복원이 어렵다.
- **구현**:
  U-Net과 FPN의 구조적 유사성에 착안한 단순한 Up Block(Transpose Conv → Conv → ReLU 반복) 2회 스택. P2 `(C, H/4, W/4)`를 2번 업샘플링해 원본 해상도로 복원 후 Sigmoid.
- **입출력 shape**:
  `P2 (C, H/4, W/4)` → `I_r (3, H, W)` (원본 이미지와 동일 shape).

```python
# 논문 Eq.(1)-(3) 기반
def Up(X):
    X = TransConv(X)          # (C, H, W) -> (C/2, 2H, 2W)
    X = ReLU(Conv1(X))
    X = ReLU(Conv2(X))
    return X

I_r = Sigmoid(Conv(Up(Up(P2))))          # (C, H/4, W/4) -> (3, H, W)
D   = Mean_channel(Abs(I_r - I_o))        # (3, H, W) -> (1, H, W), difference map
```

`Conv1`/`Conv2`는 3×3 kernel, `TranConv`는 4×4 kernel·stride 2. Reconstruction head는 원본과 재구성 이미지 간 MSE loss만으로 학습되는 순수 self-supervised 신호이며, 별도의 tiny object 위치 라벨이 필요 없다.

<mark style="background: #FFF9D6A6;">Backbone/FPN을 거치며 정보가 심하게 지워진 영역일수록 원본 복원이 어렵기 때문에, difference map 값 자체가 "이 영역에서 정보가 얼마나 소실됐는가"를 드러내는 대리 신호가 된다 — 생성 기반 방법과 달리 없는 디테일을 새로 만들지 않으므로 spurious texture/artifact 문제가 발생할 여지가 없다(Fig. 1: 사라진 tiny drone이 difference map에서는 뚜렷).</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> 재구성 task 추가로 인한 학습 비용/불안정성, 추론 시 파라미터·FLOPs·latency 증가를 논문이 정량 분석하지 않는다. Ablation(Table 4)에서 RH 단독으로도 성능이 떨어지지 않는다는 간접 증거만 제시한다.

> [!info] 내 메모
> 

### ② Difference Map Guided Feature Enhancement (DGFE)

- **역할**:
  Difference map은 위치 정보만 담고 있어 노이즈가 섞인 spatial-only 신호다. DGFE는 이 difference map을 걸러내고(Filtration) 채널 정보로 보강한 뒤(Reweighting), element-wise attention 형태로 P2에 곱해 tiny object 가능성이 높은 위치·채널만 선택적으로 강조하는 plug-and-play 모듈이다.
- **구현**:
  - **Filtration**: difference map은 재구성 오차 특성상 전 영역이 크든 작든 어느 정도 활성화되는 노이즈가 있음 — 학습 가능한 threshold `t`로 이진화 후 P2 크기로 리사이즈, `+1` 형태로 원래 P2 정보가 지워지지 않도록 보존.
  - **Reweighting**: P2에 avg/max pooling → MLP(FC-ReLU-FC) → sigmoid로 채널 방향 가중치 계산(SE-block 유사 구조).
  - 최종적으로 두 신호를 곱한 attention matrix `M`을 P2에 element-wise로 곱함(concat이나 단순 곱셈이 아님).
- **입출력 shape**:
  `D (1, H, W)`, `P2 (C, H/4, W/4)` → `P2' (C, H/4, W/4)`.

```python
# 논문 Eq.(4)-(6) 기반
Db  = (Sign(D - t) + 1) * 0.5              # 학습 가능한 threshold t로 이진화, D in (1,H,W)
Filtration = Resize(Db) + 1                 # (1, H/4, W/4), +1로 원본 정보 보존

Reweighting = Sigmoid(MLP(AvgPool(P2)) + MLP(MaxPool(P2)))   # (C, 1, 1)

M   = Reweighting * Filtration              # broadcast: (C,1,1) x (1,H/4,W/4) -> (C,H/4,W/4)
P2_ = M * P2                                 # element-wise attention (EA)
```

강화된 `P2'`가 FPN 최하위 레벨 P2를 대체해 detection head로 전달된다. FPN 기반 generic detector에 plug-and-play 결합 가능하며, RetinaNet처럼 P3만 쓰는 one-stage detector는 P3 기반으로도 재구성 가능하다.

<mark style="background: #FFF9D6A6;">Filtration으로 노이즈를 걸러내고(Tab. 5) Reweighting으로 채널 정보를 보태 attention 형태로 곱하므로, 원본 P2를 해치지 않으면서 tiny object 가능성이 높은 위치·채널만 선택적으로 강조한다 — Concat(36.6 AP)·단순 곱셈(36.2 AP)보다 EA(38.3 AP)가 확연히 나은 것(Tab. 6)이 이 설계의 효과를 뒷받침한다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> Difference map이 정보 손실이 아닌 다른 이유(복잡한 텍스처, 반복 패턴, 저조도/블러)로 활성화될 위험이 있다 — Filtration 모듈이 필요했던 이유 자체가 이 노이즈 문제이며(3.3절), threshold로 완화했지만 tiny-object-specific 신호와 일반 재구성 난이도 신호를 분리하는 장치는 없다.

> [!info] 내 메모
> 

추가 기여로, 평균 물체 크기 약 7.9픽셀(기존 anti-UAV 데이터셋 중 최소)의 새 anti-UAV 데이터셋 **DroneSwarms**를 공개했다. 기존 MAV-VID(평균 166px)·Drone-vs-Bird(평균 28px)·DUT Anti-UAV 대비 훨씬 작은 물체 위주이며 multi-instance 상황을 포함한다.

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| ① RH | P2 (C, H/4, W/4) | I_r (3, H, W) | 원본 이미지 복원, difference map 산출용 | Up Block(TransConv+Conv+ReLU) ×2 + Sigmoid |
| ② Difference Map | I_r, I_o (3, H, W) | D (1, H, W) | 정보 손실 정도의 대리 신호 | 채널 평균 절대차 |
| ③ DGFE | D (1,H,W) + P2 (C,H/4,W/4) | P2' (C, H/4, W/4) | tiny object 위치·채널 선택적 강화 | Learnable threshold Filtration + SE류 Reweighting → element-wise attention |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table 1 (DroneSwarms)
**표를 보는 법**: 각 행이 baseline detector와 SR-TOD 결합 버전 쌍이고, AP_vt/AP_t/AP_s가 각각 2~8px/8~16px/16~32px 크기 구간별 성능이다 — 최고 성능 조합(RFLA w/ SR-TOD)과 baseline(RFLA)을 비교하면 된다.

| 벤치마크 | 모델 | 지표 | Before | After |
|---|---|---|---|---|
| DroneSwarms | RFLA | AP | 36.9 | 39.0 (+2.1, 최고) |
| DroneSwarms | Cascade R-CNN | AP | 36.4 | 38.3 (+1.9) |

> [!note]- 세부 결과 및 Ablation
> #### 설정
> - **데이터셋**: DroneSwarms(자체 제안, anti-UAV, 평균 물체 크기 약 7.9px), VisDrone2019(10 class), AI-TOD(8 class, tiny object 특화)
> - **지표**: MS COCO AP는 32px 이하를 APs 하나로 뭉뚱그려 부적합 — AI-TOD 기준 AP, AP0.5, AP0.75, AP_vt(2~8px), AP_t(8~16px), AP_s(16~32px) 사용
> - Backbone ImageNet 사전학습 ResNet-50, MMDetection 구현. DroneSwarms는 lr 0.0025로 20 epoch(SGD, momentum 0.9, weight decay 0.0001, batch 2), VisDrone2019/AI-TOD는 RFLA[53] 세팅 그대로(lr 0.005, 12 epoch, 8/11 epoch decay)
>
> #### DroneSwarms 전체 (Table 1)
> | Method | AP | AP0.5 | AP0.75 | AP_vt | AP_t | AP_s |
> |---|---|---|---|---|---|---|
> | RetinaNet | 27.3 | 75.8 | 11.3 | 21.0 | 40.5 | 56.2 |
> | RetinaNet w/ SR-TOD | 28.7 (+1.4) | 77.2 | 12.9 | 22.3 | 42.1 | 56.9 |
> | Faster R-CNN | 35.0 | 83.9 | 20.9 | 27.3 | 44.3 | 56.5 |
> | Faster R-CNN w/ SR-TOD | 36.3 (+1.3) | 86.4 | 21.7 | 29.5 | 45.1 | 56.0 |
> | Cascade R-CNN | 36.4 | 85.0 | 23.5 | 28.8 | 45.7 | 58.3 |
> | Cascade R-CNN w/ SR-TOD | 38.3 (+1.9) | 87.4 | 25.4 | 30.8 | 47.4 | 59.4 |
> | DetectoRS | 37.9 | 87.4 | 24.8 | 30.5 | 46.9 | 59.3 |
> | DetectoRS w/ SR-TOD | 38.8 (+0.9) | 87.9 | 26.3 | 31.6 | 47.7 | 59.0 |
> | RFLA | 36.9 | 86.3 | 23.4 | 29.5 | 45.3 | 58.0 |
> | RFLA w/ SR-TOD | **39.0** (+2.1) | **88.9** | 25.8 | **31.8** | 47.6 | 59.2 |
> | DINO | 35.4 | 85.9 | 20.3 | 28.8 | 44.7 | 57.3 |
> | DINO w/ SR-TOD | 35.6 (+0.2) | 86.0 | 20.7 | 28.4 | 44.8 | 58.2 |
>
> Transformer 계열(DINO, Deformable-DETR도 50 epoch 스케줄로 별도 비교됨)의 개선폭(+0.2 AP)이 CNN 계열(대부분 +1.3~+2.1 AP)보다 뚜렷이 작음.
>
> #### VisDrone2019 (Table 2) / AI-TOD (Table 3)
> | 벤치마크 | 모델 | AP | AP0.5 | AP0.75 | AP_vt | AP_t | AP_s |
> |---|---|---|---|---|---|---|---|
> | VisDrone2019 | RFLA | 27.2 | 48.0 | 26.6 | 4.5 | 13.0 | 23.6 |
> | VisDrone2019 | RFLA w/ SR-TOD | 27.8 (+0.6) | 48.8 | 27.5 | 4.8 | 13.2 | 24.5 |
> | VisDrone2019 | Cascade R-CNN | 25.2 | 42.6 | 25.9 | 0.1 | 7.0 | 22.5 |
> | VisDrone2019 | Cascade R-CNN w/ SR-TOD | 27.3 (+2.1) | 46.9 | 27.5 | 2.3 | 11.5 | **24.7** |
> | AI-TOD | RFLA | 21.7 | 50.5 | 15.3 | 8.3 | 21.8 | 26.3 |
> | AI-TOD | RFLA w/ SR-TOD | 21.8 (+0.1) | 50.8 | 15.4 | 9.7 | 21.8 | 27.4 |
> | AI-TOD | HANet[21] (비SR-TOD 경쟁기법) | 22.1 | 53.7 | 14.4 | **10.9** | 22.2 | 27.3 |
> | AI-TOD | DetectoRS | 14.6 | 31.8 | 11.5 | 0.0 | 11.0 | 27.4 |
> | AI-TOD | DetectoRS w/ SR-TOD | **24.0** (+9.4) | **54.6** | **17.1** | 10.1 | **24.8** | **29.3** |
>
> AI-TOD에서 DetectoRS w/ SR-TOD가 HANet(scale-specific feature subspace 기반)을 대부분 지표에서 상회(AP 22.1→24.0, AP_t 22.2→24.8). RFLA w/ SR-TOD는 AP_vt·AP_t 개선폭이 가장 크지만(+1.4/0.0) AP 절대 개선(+0.1)은 작음 — shallow feature가 이미 강한 baseline일수록 개선 여지가 작다는 저자 해석.
>
> #### Ablation — 개별 모듈 기여 (Table 4, DroneSwarms/Cascade R-CNN)
> | 구성 | AP | AP0.5 | AP_vt | AP_t | AP_s |
> |---|---|---|---|---|---|
> | Baseline | 36.4 | 85.0 | 28.8 | 45.7 | 58.3 |
> | + RH만 | 36.5 (+0.1) | 84.9 | 28.7 | 45.9 | 58.5 |
> | + RH + DGFE | 38.3 (+1.8 vs RH단독) | 87.4 | 30.8 | 47.4 | 59.4 |
>
> #### Ablation — Feature enhancement 방식 (Table 6)
> | 방식 | AP | AP0.5 | AP_vt | AP_t | AP_s |
> |---|---|---|---|---|---|
> | Element-wise multiplication(EM) | 36.2 | 84.9 | 28.8 | 45.7 | 58.4 |
> | Concatenate | 36.6 | 85.3 | 29.0 | 46.0 | 58.7 |
> | Element-wise attention(EA, 채택) | **38.3** | **87.4** | **30.8** | **47.4** | **59.4** |
>
> #### Ablation — Threshold filtration (Table 5, VisDrone2019)
> | 방식 | AP | AP0.5 | AP_vt | AP_t | AP_s |
> |---|---|---|---|---|---|
> | threshold 없음 | 27.0 | 46.6 | 2.2 | 11.3 | 24.2 |
> | Fixed threshold | 27.1 | 46.8 | **2.3** | 11.2 | 24.0 |
> | Learnable threshold (채택) | **27.3** | **46.9** | **2.3** | **11.5** | **24.7** |
>
> #### Ablation — Difference map 종류 (Table 7, DroneSwarms, baseline 대비)
> | 종류 | AP | AP0.5 | AP_vt | AP_t | AP_s |
> |---|---|---|---|---|---|
> | baseline(difference map 없음) | 36.4 | 85.0 | 28.8 | 45.7 | 58.3 |
> | Pixel difference map(PD, 채택) | 38.3 | 87.4 | 30.8 | 47.4 | **59.4** |
> | High-frequency difference map(HFD, FFT 기반) | **38.4** (+0.1) | 87.6 | **31.1** (+0.3) | **47.6** (+0.2) | 59.2 (−0.2) |
>
> #### 세부 발견
> - Reconstruction Head 단독은 성능 개선이 거의 없다(+0.1 AP) — difference map을 실제로 활용하는 DGFE가 있어야 유의미한 개선(+1.8 AP). "재구성한다" 자체보다 "difference map을 어떻게 쓰는가"가 핵심.
> - HFD가 PD보다 AP_vt/AP_t에서는 소폭 우세하지만 AP_s에서는 열세(59.2 vs 59.4) — 고주파 성분 추출이 일부 작은 드론 신호를 노이즈로 오인해 흐릿하게 만드는 trade-off. 계산 효율 고려해 PD를 기본값 채택.
> - 다른 detector(Cascade R-CNN, DetectoRS, RFLA)와 결합해도 일관되게 개선 — 특정 아키텍처에 국한된 효과 아님.
> - Fig. 4 정성 시각화: 강화 전(baseline)에는 일부 tiny drone의 feature 신호가 거의 사라지는데, SR-TOD 적용 후에는 해당 위치의 활성이 뚜렷해짐.

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- 재구성 task 추가로 인한 학습 비용/불안정성 위험 → <mark style="background: #FF5582A6;">논문은 reconstruction loss와 detection loss의 결합 가중치, 학습 곡선/수렴 안정성을 정량 분석하지 않음 — Table 4에서 RH 단독으로도 성능이 떨어지지 않는다는 간접 증거만 제시.</mark>
- 추론 시 연산 오버헤드 위험 → <mark style="background: #FF5582A6;">파라미터 수·FLOPs·latency 비교표 없음. "plug-and-play로 쉽게 결합 가능"은 구조적 편의성 주장이지 연산 비용이 작다는 근거는 아님.</mark>
- difference map이 정보 손실이 아닌 다른 이유(복잡한 텍스처, 반복 패턴, 저조도/블러)로 활성화될 위험 → <mark style="background: #FF5582A6;">Filtration 모듈이 필요했던 이유 자체가 이 노이즈 문제(3.3절)이며, threshold로 완화했지만 tiny-object-specific 신호와 일반 재구성 난이도 신호를 분리하는 장치는 없다.</mark>

### 한계
- <mark style="background: #FF5582A6;">저자가 결론에서 직접 명시: "향후 더 정확한 difference map을 구성하는 방법을 탐색하겠다"</mark> — 현재 형태가 최선이 아님을 스스로 인정.
- <mark style="background: #FF5582A6;">High-frequency difference map이 일부 더 작은 드론 객체를 오히려 흐릿하게 만드는 부작용 관찰(4.4절, Table 7 AP_s 59.4→59.2).</mark>
- <mark style="background: #FF5582A6;">Transformer 계열(DINO) 개선폭(+0.2 AP)이 CNN 계열(RFLA +2.1, Cascade R-CNN +1.9)보다 뚜렷이 작음.</mark> "DETR은 효과적인 multi-scale feature를 갖지 못하고 sparse query 패러다임이 tiny object를 충분히 커버하지 못한다"는 일반론([8, 11] 인용)만 제시, 구체적 원인 분석은 없음.
- FPN 기반 P2/P3 레벨 의존 — FPN 미사용 아키텍처(순수 sparse query 기반 최신 transformer 탐지기)로의 적용성은 원천적으로 제한.

### 생각할 점
- <mark style="background: #A6E3A1A6;">"재구성 난이도를 정보 손실의 대리 신호로 쓴다"는 아이디어는 tiny object detection에 국한되지 않는다 — anomaly detection에서 autoencoder reconstruction error를 이상 신호로 쓰는 것과 본질적으로 같은 원리이며, 저자 중 한 명(Cao, B.)의 선행 연구([7], 의료 영상 합성)에도 유사한 발상이 쓰인다. 세그멘테이션 경계 모호 영역, 저조도 영상 신뢰도 추정에도 이식 여지가 있다.</mark>
- Reconstruction target을 원본 전체가 아니라 tiny object 영역만 선택적으로 재구성하도록 유도하면 filtration 없이도 노이즈를 줄일 수 있을 가능성 — 다만 이는 결국 위치 라벨을 다시 요구해 "label-free"라는 핵심 장점과 상충하는 trade-off.

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">[[Object_Detection_Approaches]]가 feature 강화 계열 여러 편(FANet의 주파수, Feature_Info_Driven_Gaussian의 정보량+위치, SR-TOD의 reconstruction 부산물, RS-TOD의 공간 attention 등)을 "어떤 신호가 tiny object 위치를 가장 잘 드러내는가" 축으로 묶어두었다. SR-TOD의 difference map(재구성 난이도 기반)과 다른 신호들을 동일 벤치마크에서 직접 비교하면 "정보 손실 진단의 여러 방식" 우열을 실증할 수 있다.</mark>
- [[2026_TIP_Unc-SOD|Unc-SOD]]는 sampling 축(어떤 proposal을 학습에 쓸지)에서 instance-level uncertainty를 도입하는데, difference map을 uncertainty의 또 다른 소스로 결합할 만하다 — "difference map 활성도가 큰 영역=정보가 불확실한 영역"이라는 해석은 Unc-SOD의 aleatoric uncertainty 개념과 방향이 통한다.

> [!info] 내 메모
> 

# 관련 개념
- [[Self_Reconstruction_Difference_Map]] — 이 논문의 핵심 기여로, 탐지 모델 내부에 이미지 self-reconstruction을 넣어 원본과의 차이(difference map)를 tiny object 위치·구조에 대한 prior로 활용하는 아이디어. 이 논문이 원조.

# 관련 문서
- 비교 후보: [[2026_TIP_Unc-SOD|Unc-SOD]] — 두 논문 모두 small/tiny object detection에서 baseline detector(Cascade R-CNN, RFLA, DetectoRS 등)에 plug-in 모듈을 추가하는 접근이며, `Unc-SOD.md`의 "관련 문서" 절에서도 이 논문을 비교 후보로 언급하고 있음. Unc-SOD는 sampling 축, SR-TOD는 feature 강화 축이라는 점에서 직교적.
- 참고(혼동 주의): 이름이 유사한 [[2025_RSASE_RS-TOD|RS-TOD]](RS-TOD, 2025, YOLOv8+attention 기반)는 저자·방법론 모두 무관한 별개 논문이다 — `RS-TOD.md`에서도 이 문서를 언급하며 동일한 혼동 방지 각주를 남겨두었다.
- 비교: [[Object_Detection_Approaches]] — feature 강화 계열(self-reconstruction 축, 이 계열 중 가장 가벼운 방식)로 분류

# 읽어볼 만한 논문
- 참고문헌 기반: C. Xu, J. Wang, W. Yang, H. Yu, L. Yu, G.-S. Xia, "RFLA: Gaussian receptive field based label assignment for tiny object detection" [53] (ECCV 2022) — SR-TOD의 모든 실험에서 baseline이자 최고 성능 조합(RFLA w/ SR-TOD)의 기반이 되는 label assignment 기법. VisDrone2019/AI-TOD 실험 세팅도 이 논문을 그대로 따랐다.
- 참고문헌 기반: C. Deng, M. Wang, L. Liu, Y. Liu, Y. Jiang, "Extended feature pyramid network for small object detection" [14] (IEEE TMM 2021) — feature imitation 계열의 대표 선행 연구로, 이 논문이 "생성 기반 접근은 spurious artifact를 만든다"고 비판할 때 직접 인용하는 대상. SR-TOD가 대체하려는 접근을 이해하는 데 필요.
- 참고문헌 기반: G. Cheng, X. Yuan, X. Yao, K. Yan, Q. Zeng, X. Xie, J. Han, "Towards large-scale small object detection: Survey and benchmarks" [11] (IEEE TPAMI 2023) — tiny object detection 전체를 5갈래(data augmentation/scale awareness/context modeling/feature imitation/label assignment)로 정리한 서베이. SR-TOD의 related work 구성이 이 서베이의 분류를 그대로 따르고 있어, 분야 전체 지형을 파악하는 배경 자료로 적합.
- 참고문헌 기반: L. Jiang, B. Dai, W. Wu, C. C. Loy, "Focal frequency loss for image reconstruction and synthesis" [24] (ICCV 2021) — SR-TOD가 high-frequency difference map을 설계할 때 참고한 주파수 도메인 image reconstruction 연구. Ablation에서 다룬 HFD 변형의 배경 이해에 도움되며, [[2025_RemoteSensing_FANet|FANet]]·[[2025_arXiv_UAV-DETR|UAV-DETR]] 등 이 위키의 다른 주파수 도메인 접근 논문과도 연결됨.
- 자유 추천(검증 필요): reconstruction error를 이상 신호로 쓰는 autoencoder 기반 anomaly detection 계열 연구 — 검색 키워드: `autoencoder reconstruction error anomaly detection survey`. SR-TOD의 핵심 발상("재구성이 어려운 영역=정보가 특이한 영역")이 anomaly detection의 표준 패러다임과 원리적으로 같아 보이므로, 위 Discussion의 "생각할 점"에서 제기한 교차 적용 가능성을 검증할 때 참고할 만하다.
