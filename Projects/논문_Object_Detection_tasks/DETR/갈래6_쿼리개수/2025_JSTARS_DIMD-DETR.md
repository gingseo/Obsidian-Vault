---
pm-task: true
projectId: "paperwiki-object-detection"
parentId:
id: "t-dimd-detr-b3n8w1zqxc"
title: "DIMD-DETR: DDQ-DETR With Improved Metric Space for End-to-End Object Detector on Remote Sensing Aircrafts"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-09-11"
due:
progress: 0
assignees: []
tags: []
customFields:
  "nh3oelhxmtcnb377": 2025
  "gx1mmrf0mtcnb37a": "JSTARS"
subtaskIds: []
dependencies: []
year: 2025
venue: "IEEE Journal of Selected Topics in Applied Earth Observations and Remote Sensing (JSTARS)"
jcr_quartile: Q1
task: [object-detection]
direction: [improvement]
paper_tags: [paper, object-detection, remote-sensing, aircraft-detection, detr, query-mechanism, metric-learning, loss-function, backbone]
source: "Projects/_pdf/Object_Detection/DETR/갈래6_쿼리개수/2025_JSTARS_DIMD-DETR.pdf"
source_type: personal
createdAt: "2026-09-11T10:40:00.000Z"
updatedAt: "2026-09-11T10:40:00.000Z"
---

Project: [[논문_Object_Detection|Object Detection]]
#paper #object-detection #remote-sensing #aircraft-detection #detr #query-mechanism #metric-learning #loss-function #backbone

> [!quote] 원제
> **DIMD-DETR: DDQ-DETR With Improved Metric Space for End-to-End Object Detector on Remote Sensing Aircrafts**
> Huan Liu, Xuefeng Ren, Yang Gan, Yongming Chen, Ping Lin — Hubei Normal University, IEEE JSTARS 2025
> https://doi.org/10.1109/JSTARS.2025.3530141

# 한 줄 요약
<mark style="background: #FFF3A3A6;">DDQ-DETR(Dense Distinct Query)을 베이스로, decoder 레이어 간 gradient 흐름을 개선하는 BLTP(Bilayer Targeted Prediction), 저해상도·복잡 배경에서 다중 스케일 feature를 강화하는 PVTv2 백본(pyramid+self-attention), 항공기 특화 Albu 증강, 4개 loss 요소(GHMC/Smooth L1/SIoU)를 1:5:2 비율로 결합한 metric-space 기반 손실 함수, 학습 전반부 선형·후반부 코사인 감쇠를 결합한 동적 learning rate 스케줄까지 5가지 요소를 한 파이프라인에 통합해, 자체 구축한 MDMF 항공기 데이터셋에서 DDQ-DETR 대비 AP +2.8%p(64.7→67.5), 특히 AP_S +4.6%p(48.5→53.1)를 달성해 비교 대상 19개 모델 중 AP·AP50·AP75·AP_S 전 지표 최고를 기록한 원격탐사 항공기 탐지 논문(다만 파라미터 63.2M은 DDQ-DETR(42.5M)보다는 많고, Cascade R-CNN(108.2M) 등 일부 고성능 2-stage 모델보다는 적음).</mark>

> [!info] 내 메모
> 

# 정리

## 기존 방법의 한계
- **항공기 크기·형태의 극단적 변화**:
  위성 이미지의 촬영 각도가 다양해 같은 항공기라도 이미지 안에서 크기·형태(size and appearance)가 크게 달라져, 모델이 이 변화에 적응하기 어렵다(Fig. 1(b)).
- **낮은 해상도로 인한 특징 추출 어려움**:
  위성 원격탐사 이미지는 해상도가 낮아 항공기 타겟이 보통 10~20픽셀에 불과해, feature 추출 자체가 어렵다(Fig. 1(a)).
- **복잡한 배경으로 인한 오탐과 가려짐**:
  구름·지형 기복 등 배경 간섭과 물체 가려짐(occlusion)이 실시간 탐지 요구사항까지 겹쳐 오탐(false positive)을 높이고 탐지 난이도를 키운다(Fig. 1(c), (d)).

## 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — DDQ-DETR(dense distinct query) 계열**
- <mark style="background: #FFF3A3A6;">Zhang et al.(DDQ-DETR, [18]): DETR의 query가 dense하면서도 서로 구별되도록(dense yet diverse) 생성해, sparse query의 느린 수렴·낮은 recall 문제를 완화 — ResNet-50 기반, 1500개 dense query로 시작해 distinct query selection으로 중복 박스를 줄임.</mark>
- Gao et al.([19]): DDQ-DETR을 crowd pedestrian detection에 적용해 Crowdhuman에서 miss rate 39.4% 달성.
- Huang et al.([20]): DDQ-DETR을 AI-TOD-V2(소형 객체 데이터셋)에서 CNN·DETR 계열과 비교해 AP 30.2로 최고 성능 확인.
- **타겟/해결**: 기존 방법의 한계 중 낮은 해상도로 인한 특징 추출 어려움(문제②)을 dense query 자체로 일부 완화하지만, 원격탐사 항공기 특유의 크기 변화·복잡 배경(문제①③)까지 정면으로 다루지는 않는다.

**갈래 2 — Backbone(원격탐사 특화 feature 추출)**
- Dosovitskiy et al.(ViT, [21]), Li/Feng/Jun([22]), Ye et al.([23]): Transformer 기반(ViT, Swin) 백본으로 항공기 탐지를 시도 — 정확도는 높였지만 ViT는 서로 다른 스케일의 fine-to-coarse 정보를 동시에 포착하기 어려움([24]).
- Wang et al.([25]), Chen et al.(PVTv2 backbone, [26]), Kanca et al.([27]): CNN의 feature pyramid와 transformer의 전역 인식을 결합 — PVTv2가 NWPU-VHR-10/RSOD에서 mAP를 각각 8.4%/1.7% 개선.
- **타겟/해결**: 낮은 해상도로 인한 특징 추출 어려움(문제②)을 겨냥하지만, 어느 backbone도 항공기 크기·형태 변화(문제①)와 복잡 배경(문제③)까지 종합적으로 다루지 못했다.

**갈래 3 — Loss function 설계**
- Li et al.([28]): L1+GIoU 조합으로 드론 프라이버시 탐지에서 정확도 +2.3%.
- Bai et al.([29]): GIoU loss를 수술 부위 탐지에 적용.
- Zhang et al.([30]): Smooth L1을 의료 영상 분할에 적용해 AP 85.8% 달성.
- Qi et al.([31]): YOLOv5+SIoU 조합이 실내 물체 인식에서 최고 성능.
- **타겟/해결**: 개별 loss 요소(분류/회귀/IoU)의 개선은 각자 이루어졌지만, 항공기 탐지처럼 여러 문제(문제①②③)가 동시에 얽힌 상황에서 이 요소들을 하나의 확장 가능한 구조로 결합·조율한 시도는 없었다.

**갭**: <mark style="background: #FFF3A3A6;">DDQ-DETR·PVTv2·개별 loss 개선 모두 각자의 축에서는 진전을 이뤘지만, 아직 어떤 탐지기도 원격탐사 항공기 이미지 특유의 크기·형태 변화, 낮은 해상도, 복잡 배경이라는 세 문제를 동시에 효과적으로 다루지 못했다.</mark>

## 이 논문이 풀고자 하는 문제
1. DDQ-DETR decoder의 레이어 간 gradient 분리(SLTP)로 인한 정보 손실을 보완해, 가려짐·높은 오탐률 환경에서 탐지 민감도와 정확도를 높이는 것.
2. 저해상도 타겟에 대한 전역·지역 feature의 공동 학습을 강화해, 소형 항공기 탐지 성능을 끌어올리는 것.
3. 크기·형태 변화에 대한 모델의 일반화 능력을 데이터 증강으로 확보하는 것.
4. 모듈형 구조의 여러 구성 요소(분류/회귀/IoU 손실) 간 시너지를 조율하는 손실 함수를 설계하는 것.
5. 학습 초반 빠른 수렴과 후반 전역 탐색(global exploration)을 동시에 달성하는 학습률 스케줄을 만드는 것.

**갭 종합**: <mark style="background: #FFF3A3A6;">이 논문의 통찰은 "단일 장치 하나로는 원격탐사 항공기 탐지의 복합적 난제를 풀 수 없다"는 것이다 — decoder 구조(BLTP), backbone(PVTv2), 데이터 증강(Albu), 손실 함수(metric space), 학습 스케줄(dynamic LR) 다섯 요소를 모듈식으로 조합해야 각 문제를 서로 보완하며 해결할 수 있다고 본다.</mark>

> [!info] 내 메모
> 

# 해결 방법 요약

| | 문제①②③ — 크기/형태 변화, 저해상도, 복잡 배경(오탐·가려짐) | 문제④⑤ — 구성요소 시너지, 학습 안정성 |
|---|---|---|
| **해결 방법** | BLTP로 decoder 레이어 간 gradient 정보를 보존하고, PVTv2 backbone으로 다중 스케일 global+local feature를 강화하며, Albu 증강으로 크기·형태 변화에 대한 일반화를 확보 | Metric-space 기반 손실 함수(GHMC+Smooth L1+SIoU, 1:5:2)로 분류·회귀·IoU를 통합 조율하고, Lin-LR(0~100 epoch)+Cos-LR(100~200 epoch) 동적 스케줄로 수렴 속도와 탐색 범위를 균형 |
| **예상되는 문제점** | PVTv2-B3 도입으로 파라미터·연산량이 ResNet-50 대비 증가(Table X: 63.2M/243.7 GFLOPs, DDQ-DETR 대비 +20.7M/+13.3 GFLOPs) | 손실 함수의 가중 비율(1:5:2)과 LR 스케줄의 epoch 경계(100/200)가 MDMF 데이터셋에 맞춰 튜닝되어, 다른 데이터셋에서의 재조정 필요성이 검증되지 않음 |

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">DDQ-DETR을 베이스로, decoder 레이어 사이의 gradient 상호작용을 강화하는 <span style="color:#c0392b; font-weight:bold;">BLTP(Bilayer Targeted Prediction)</span>, PVTv2의 피라미드 구조·self-attention을 그대로 가져온 <span style="color:#c0392b; font-weight:bold;">backbone 교체</span>, 항공기 특화 <span style="color:#c0392b; font-weight:bold;">Albu 데이터 증강</span>, GHMC+Smooth L1+SIoU를 조합한 <span style="color:#c0392b; font-weight:bold;">metric-space 기반 손실 함수</span>, 학습 전반부 선형·후반부 코사인 감쇠를 결합한 <span style="color:#c0392b; font-weight:bold;">동적 learning rate 스케줄</span> 5가지를 결합한 DIMD-DETR을 제안한다.</mark>

## 전체 파이프라인 (Fig. 3 기준)

```
입력 이미지 (H×W×3)
       │
       ▼
① Albu 데이터 증강 (회전/크롭/플립/스케일링/밝기·대비/노이즈/색변환/elastic/grid distortion)
       │
       ▼
② PVTv2-B3 Backbone (4-stage 피라미드, 각 stage: Patch Embedding → Transformer Encoder(Linear SRA) × L_i)
       → multi-scale feature: (H/4·W/4·C1), (H/8·W/8·C2), (H/16·W/16·C3), (H/32·W/32·C4)
       │
       ▼
③ Position Encoding + Transformer Encoder × N  → encoder memory
       │
       ▼
④ Dense Query Selection (DDQ-DETR 방식) → dense distinct query 1500개 → Content Queries
       │
       ▼
⑤ Transformer Decoder × N, 레이어 사이 BLTP로 gradient 보정
   (Layer N → BLTP → Layer N+1 → BLTP → ... )     → decoder 출력 (1500, d)
       │
       ▼
⑥ Feed Forward Network → Distinct Query Selection → 최종 (클래스, 박스)
       │
       ▼ (학습 시)
⑦ Metric-space 손실: Ω = ||λ·C||_p = 1·GHMC + 5·SmoothL1 + 2·SIoU
       │
       ▼ (학습 스케줄)
⑧ Dynamic LR: epoch 1~100 Lin-LR(α→β), epoch 100~200 Cos-LR(λ 감쇠, β로 수렴)
```

> [!info] 내 메모
> 

### ① Albu 데이터 증강
- **역할**: 항공기 이미지 특유의 스케일·외형 변화에 대한 모델의 적응력을 높여, 과적합 위험을 낮추고 다양한 조명·날씨 조건에 대한 강건성을 강화한다.
- **구현**: Albumentations(Albu) 라이브러리로 회전, 크롭, 플립, 스케일링, 밝기·대비·노이즈·색상 조정, elastic transformation, grid distortion을 적용. Elastic transformation과 grid distortion은 특히 복잡한 배경에서의 탐지 정확도를 정교화하기 위해 선택됨. 세부 파라미터는 논문에 첨부된 configuration 문서에서만 제공되며 본문에는 값이 명시되지 않음.
- **입출력 shape**: 입력 이미지 `(H, W, 3)` → 증강 후 동일 shape `(H, W, 3)` (값만 변화).

(연한 노랑 하이라이트) <mark style="background: #FFF9D6A6;">"정리"의 첫 번째 문제(항공기 크기·형태의 극단적 변화)를, 학습 데이터 자체의 스케일·형태 다양성을 인위적으로 늘려 대응한다 — Table IV에서 baseline 대비 Albu 추가 시 AP가 세 데이터셋 전반에서 개선되며(MDMF 64.7→65.5), 특히 DIOR에서 개선폭이 가장 크게 나타난다.</mark>

> [!info] 내 메모
> 

### ② PVTv2 Backbone (Pyramid Vision Transformer V2)
- **역할**: CNN 구조는 receptive field 확장에 한계가 있고, 순수 ViT는 fine-to-coarse 다중 스케일 정보를 동시에 포착하기 어렵다. PVTv2는 피라미드 구조(다중 스케일 처리)와 self-attention(전역 관계 학습)을 결합해 저해상도 소형 타겟에서도 전역·지역 feature를 함께 학습한다.
- **구현**: 4단계(stage) 피라미드 구조. 각 stage는 Patch Embedding → 여러 층의 Transformer Encoder(Linear Spatial Reduction Attention, Linear SRA)로 구성. Conv SRA는 attention 이전에 convolution으로 공간 차원 $h \times w$를 줄이는 반면, Linear SRA는 average pooling으로 고정 크기 $P \times P$($P=7$)까지 줄여 연산 비용을 더 낮춘다(Eq. 3-4: $O_{ConvSRA} = \frac{2h^2w^2c}{R^2} + hwc^2R^2$, $O_{LinerSRA} = 2hwP^2c$). PVTv2-B3 변형을 backbone으로 채택(Table III에서 ablation으로 선정).
- **입출력 shape**: 입력 이미지 `(H, W, 3)` → 4개 stage를 거쳐 `(H/4, W/4, C1)`, `(H/8, W/8, C2)`, `(H/16, W/16, C3)`, `(H/32, W/32, C4)` 다중 스케일 feature map.

```python
# 논문 §III.D, Eq.(3)-(4) 기반 의사코드, PVTv2 원 논문[25] 구조 재사용
for stage_i in range(4):
    x = patch_embedding(x)                       # 해상도 축소, 채널 확장
    for layer in transformer_encoder_layers[stage_i]:
        q = linear(x)
        kv_input = avg_pool(x, output_size=(7,7))  # Linear SRA: 고정 P×P로 축소
        k, v = linear(kv_input), linear(kv_input)
        x = multi_head_attention(q, k, v) + x       # residual
        x = feed_forward(x) + x
    multiscale_features.append(x)
```

(연한 노랑 하이라이트) <mark style="background: #FFF9D6A6;">"정리"의 두 번째 문제(낮은 해상도로 인한 특징 추출 어려움)를, 피라미드 구조로 다중 스케일 feature를 점진적으로 추출하고 self-attention으로 전역 문맥까지 반영함으로써 완화한다 — Table III에서 PVTv2-B3는 AP_S 51.6으로 Swin-B(50.3)·ConvNeXt-B(49.8)·ResNet-50(48.5) 등 CNN·타 transformer 계열을 모두 상회하며, PVTv2-B4(51.9)에만 근소하게 못 미치는 최상위권을 기록한다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> PVTv2-B4가 PVTv2-B3보다 AP가 소폭 높지만(66.4 vs 66.2) 파라미터·FLOPs가 1.5배에 달해([25]) 저자들은 연산 효율을 고려해 B3를 최종 채택했다 — 즉 backbone 교체 자체가 정확도와 연산 비용 사이의 트레이드오프를 수반하며, Table X에서 DIMD-DETR의 파라미터(63.2M)·FLOPs(243.7G)는 DDQ-DETR(42.5M/230.4G)보다 크다.

> [!info] 내 메모
> 

### ③④ Encoder + Dense Query Selection (DDQ-DETR 표준 구조)
- **역할**: PVTv2가 만든 다중 스케일 feature에 위치 인코딩을 더해 transformer encoder로 문맥을 반영한 뒤, DDQ-DETR의 dense query selection으로 decoder에 투입할 1500개의 dense하면서도 서로 구별되는(distinct) query를 고른다. 이 부분은 이 논문의 기여가 아니라 DDQ-DETR의 표준 구조를 그대로 사용한다.
- **구현**: DDQ-DETR 방식대로 encoder 출력에서 분류 신뢰도 기반으로 dense query를 선택하되, 중복도를 억제해 다양성을 확보(distinct query selection). Content Query로 decoder에 전달.
- **입출력 shape**: multi-scale feature `(H/4·W/4, ..., H/32·W/32, C)` → encoder → memory → dense query selection → `Content Queries (1500, d)`.

> [!info] 내 메모
> 

### ⑤ BLTP (Bilayer Targeted Prediction)
- **역할**: DDQ-DETR의 기존 방식인 SLTP(Single-Layer Targeted Prediction)는 decoder 레이어 사이의 gradient backpropagation을 분리(gradient separation)해, $N$번째 레이어 파라미터가 오직 현재 레이어의 loss로만 갱신되도록 한다. 이는 학습을 안정시키지만, 각 레이어가 다른 레이어의 정보를 충분히 활용하지 못해 feature 손실이 생긴다. BLTP는 $N$번째 레이어의 예측 박스가 $N$번째와 $N{+}1$번째 레이어의 loss를 모두 반영하도록 gradient 갱신 방식을 바꿔, 레이어 간 양방향 정보 교환을 가능하게 한다.
- **구현**: SLTP는 Eq.(1)만 따른다 — $\Delta b_{n+1} = \text{Layer}_{n+1}(b'_n)$, $b_n = \text{Update}(b'_{i-1}, \Delta b_i)$, 이때 $b'_{n+1}$의 gradient는 $\Delta b_{n+1}$만 반영하고 $b_n$과 $b'_n$ 사이의 gradient는 차단($\text{Detach}$)된다. BLTP는 Eq.(2)를 추가로 적용해 $b'_n = \text{Detach}(b_n)$, $b^{\text{pred}}_{n+1} = \text{Update}(b_n, \Delta b_{n+1})$로 재정의함으로써, $N{+}1$번째 레이어의 예측이 $N$번째 레이어의 박스 $b_n$과 예측 offset $\Delta b_{n+1}$을 모두 반영하게 한다. Fig. 4가 SLTP·BLTP의 gradient 흐름 차이를 도식화.
- **입출력 shape**: decoder layer $N$ 출력 `(1500, d)`(box 정보 포함) → BLTP → decoder layer $N{+}1$ 입력 `(1500, d)` (shape 불변, gradient 경로만 변경).

> [!example]- 구현 디테일 (수식)
> $$\Delta b_{n+1} = \text{Layer}_{n+1}(b'_n), \quad b_n = \text{Update}(b'_{i-1}, \Delta b_i) \tag{1}$$
> $$b'_n = \text{Detach}(b_n), \quad b^{\text{pred}}_{n+1} = \text{Update}(b_n, \Delta b_{n+1}) \tag{2}$$
> 여기서 $\text{Detach}$는 gradient 분리 전후를, $\text{Update}$는 이전 레이어 출력과 현재 예측 offset을 합치는 누적 함수를, $\text{Layer}$는 $N$번째 레이어 decoder를 나타낸다.

(연한 노랑 하이라이트) <mark style="background: #FFF9D6A6;">"정리"의 첫 번째 문제(복잡한 배경에서의 가려짐·오탐)를, decoder 레이어 사이의 gradient 상호작용을 양방향으로 복원해 해결한다 — Table II에서 BLTP가 SLTP 대비 LEVIR/DIOR/MDMF 전반에서 AP(+0.7%p)·AP50(+0.4%p)·AP75(+0.6%p)를 개선하며, 특히 더 복잡한 DIOR·MDMF에서 AP50·AP75 개선폭이 크게 나타나 복잡 배경·occlusion 대응력과 직접 연결됨을 뒷받침한다.</mark>

> [!info] 내 메모
> 

### ⑥ Feed Forward Network + Prediction Head
- **역할**: BLTP로 정제된 decoder 출력을 FFN에 통과시켜 최종 (클래스, 박스)를 예측한다. 이 부분은 DDQ-DETR의 표준 출력 구조를 그대로 사용.
- **입출력 shape**: decoder 출력 `(1500, d)` → FFN → 클래스 + 박스(1500개 후보) → distinct query selection으로 최종 후보만 출력.

> [!info] 내 메모
> 

### ⑦ Metric-Space 기반 손실 함수
- **역할**: 분류·회귀·IoU 각각의 loss 요소를 독립적으로 최적화하지 않고, 하나의 norm 기반 구조(metric space)로 묶어 가중치를 조절함으로써 모듈 간 시너지를 최적화하고 과적합을 줄인다. 새 loss 요소(예: bounding box angle loss, scale loss)를 손쉽게 추가할 수 있는 확장 가능한 구조로 설계.
- **구현**: 전체 손실 $\Omega = \|\lambda \cdot C\|_p$(Eq. 5)에서 $C$는 각 loss 요소로 구성된 벡터, $\lambda$는 요소별 가중치, $p$는 L1($p{=}1$) 또는 L2($p{=}2$) norm 차수. 분류에는 GHMC(Gradient Harmonized single-stage detector Mechanism, 난이도가 다른 샘플 간 gradient를 동적으로 균형), 회귀 정규화에는 Smooth L1(outlier에 둔감, 안정적), IoU에는 SIoU(각도·위치·형태를 함께 고려)를 채택. 최종 결합: $\Omega = \|\lambda \cdot C\|_{p=1} = 1 \cdot C_{GHMC} + 5 \cdot C_{SmoothL1} + 2 \cdot C_{SIoU}$(Eq. 6, 가중 비율 1:5:2). Table V에서 각 카테고리(분류/정규화/IoU) 후보들을 개별 비교해 GHMC(AP 65.1)·Smooth L1(AP 65.4)·SIoU(AP 65.3)를 각각 최고 성능으로 선정한 뒤, Table VI에서 세 요소를 결합했을 때 AP 66.1로 개별 요소 단독 적용보다 더 높은 시너지를 확인.
- **입출력**: 예측 (클래스, 박스) + 정답 → 스칼라 loss (별도 tensor shape 없음).

```python
# 논문 Eq.(5)-(6) 기반 의사코드
C = [GHMC_loss(cls_pred, cls_gt), SmoothL1_loss(box_pred, box_gt), SIoU_loss(box_pred, box_gt)]
lambda_ = [1, 5, 2]
Omega = sum(l * c for l, c in zip(lambda_, C))   # p=1 (L1 norm)
```

(연한 노랑 하이라이트) <mark style="background: #FFF9D6A6;">"정리"의 네 번째 문제(모듈 간 시너지 조율)를, 개별 loss 요소를 독립적으로 쓰지 않고 하나의 가중 norm 구조로 결합해 해결한다 — Table VI에서 GHMC+SIoU+Smooth L1 조합이 AP 66.1·AP_S 49.8로 두 요소만 결합했을 때(65.8/49.4)보다 우수해, "결합 자체"가 개별 최적화의 단순 합보다 큰 이득을 준다는 것을 보여준다.</mark>

> [!info] 내 메모
> 

### ⑧ 동적 Learning Rate 스케줄
- **역할**: 학습 초반에는 빠르게 좋은 파라미터 영역으로 수렴시키고, 후반에는 local optima에 갇히지 않도록 충분한 전역 탐색 여지를 남겨, 두 목표(빠른 수렴 vs 전역 탐색) 사이의 균형을 맞춘다.
- **구현**: 전체 200 epoch 중 처음 100 epoch은 Lin-LR(선형 감쇠)로 초기 학습률 $\alpha$에서 목표값 $\beta$까지 선형으로 낮춘다. 100~200 epoch은 Cos-LR(코사인 어닐링)로 학습률을 다시 $\alpha$까지 올렸다가 감쇠 파라미터 $\lambda$(Eq. 7: $\lambda = 1 + \cos(\frac{t-100}{100}\pi)$)를 이용해 $\beta$로 부드럽게 수렴시킨다(Eq. 8). $\alpha=0.0002$, $\beta=1\times10^{-5}$. Optimizer는 AdamW.
- **입출력**: epoch $t$ → 학습률 $\eta_t$ (스칼라).

```python
# 논문 Eq.(7)-(8) 기반 의사코드
alpha, beta = 0.0002, 1e-5
if t <= 100:
    eta_t = alpha - (alpha - beta) * (t / 100)          # Lin-LR
else:
    lam = 1 + cos((t - 100) / 100 * pi)                   # Eq.7, Cos-LR 감쇠 파라미터
    eta_t = beta + 0.5 * lam * (alpha - beta)              # Eq.8
```

(연한 노랑 하이라이트) <mark style="background: #FFF9D6A6;">"정리"의 다섯 번째 문제(수렴 속도와 탐색 범위의 균형)를, 전반부는 빠른 수렴에, 후반부는 재탐색에 특화된 서로 다른 감쇠 곡선을 순차 적용해 해결한다 — Table VII에서 Lin_f+Cos_l 조합이 AP50 92.4%·AP75 78.4%로 다른 조합(Multi-LR, Exp-LR)보다 우수하며, 순수 선형 감쇠 단독은 AP 0.8%p 낮아 탐색 부족을 시사한다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> Exp-LR은 후반부 공격적인 학습률 조정 때문에 미세 조정(fine-tuning)을 방해해 AP75가 1.2%p 하락 — 학습률 스케줄 설계가 잘못되면 오히려 성능을 해칠 수 있음을 저자 스스로 관찰.

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| ① Albu 증강 | (H,W,3) | (H,W,3) | 크기·형태 변화 대응 데이터 다양화 | 회전/크롭/색변환/elastic/grid distortion 등 |
| ② PVTv2-B3 Backbone | (H,W,3) | 4-scale feature (H/4~H/32 등) | 다중 스케일 global+local feature 추출 | 피라미드 + Linear SRA self-attention |
| ③④ Encoder + Dense Query Selection | multi-scale feature | Content Queries(1500,d) | 문맥 반영 + dense distinct query 선별 | DDQ-DETR 표준 구조 |
| ⑤ Decoder × N + BLTP | Content Queries(1500,d) | decoder 출력(1500,d) | 레이어 간 gradient 상호작용 강화 | Eq.(1)-(2), gradient Detach/Update 재설계 |
| ⑥ FFN + Prediction | decoder 출력(1500,d) | 클래스+박스 | 최종 검출 예측 | DDQ-DETR 표준 head |
| ⑦ Metric-space Loss | 예측+정답 | 스칼라 loss | 분류·회귀·IoU 시너지 조율 | GHMC+SmoothL1+SIoU, 1:5:2 |
| ⑧ Dynamic LR | epoch t | 학습률 η_t | 수렴 속도·탐색 범위 균형 | Lin-LR(0~100)+Cos-LR(100~200) |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table VIII·X (MDMF test)
**보는 법**: Table VIII은 ①PVTv2 ②BLTP ③Albu ④metric loss ⑤dynamic LR 5개 요소를 하나씩 추가하며 성능 변화를 보는 leave-in ablation, Table X는 완성된 DIMD-DETR을 다른 SOTA 탐지기와 나란히 비교(Param/FLOPs 포함)한다.

| 벤치마크 | 지표 | Before(DDQ-DETR baseline) | After(DIMD-DETR, 전체 요소) |
|---|---|---|---|
| MDMF | AP / AP50 / AP75 | 64.7 / 91.7 / 79.9 | 67.5(+2.8%p) / 94.9(+3.2%p) / 79.7(-0.2%p) |
| MDMF | AP_S | 48.5 | 53.1(+4.6%p) |
| MDMF | Param(M) / FLOPs(G) | 42.5 / 230.4 | 63.2 / 243.7 |

> [!note]- 세부 결과 및 Ablation
> #### Table II — BLTP vs SLTP (LEVIR/DIOR/MDMF)
> **보는 법**: 같은 모듈을 SLTP/BLTP로 바꿔 끼웠을 때 AP/AP50/AP75 변화를 데이터셋별로 비교.
> LEVIR: 55.7/82.6/61.7 → 56.4/83.5/62.4. DIOR: 68.9/92.5/80.8 → 69.3/93.8/81.6. MDMF: 64.7/91.7/77.6 → 65.3/92.4/78.2. 세 데이터셋 평균 AP+0.7%p, AP50+0.4%p, AP75+0.6%p — 복잡한 DIOR·MDMF에서 AP50·AP75 개선폭이 더 크게 나타남.
>
> #### Table III — Backbone 비교 (MDMF, AP/AP50/AP_S/AP_M/AP_L)
> **보는 법**: 여러 backbone을 동일 조건에서 교체 실험 — AP_S 열이 소형 객체 탐지력의 직접 지표.
> ResNet-50 64.7/91.7/48.5/63.1/72.3, ResNeXt101-64x4d 64.9/91.8/49.3/64.9/72.9, EfficientNetv2 64.8/91.7/49.1/64.4/72.6, MobileNetv2 64.5/91.4/48.8/63.8/72.1, ConvNeXt-B 65.4/91.8/49.8/65.1/72.8, Swin-B 65.8/92.7/50.3/66.8/73.1, **PVTV2-B3(채택) 66.2/93.3/51.6/67.5/73.3**, PVTV2-B4 66.4/93.9/51.9/67.9/73.7. PVTv2-B4가 모든 지표에서 근소하게 더 높지만 파라미터·FLOPs가 B3의 1.5배라 B3를 최종 채택.
>
> #### Table IV — Albu 증강 유무 (LEVIR/DIOR/MDMF)
> **보는 법**: Baseline과 Baseline+Albu를 나란히 비교, AP_M/AP_L까지 함께 확인.
> LEVIR: 55.7→56.2, DIOR: 68.9→69.4(AP_M 72.1→72.9), MDMF: 64.7→65.5(AP75 77.6→78.1). DIOR에서 개선폭이 가장 크게 관찰됨.
>
> #### Table V — 손실 함수 요소별 비교 (분류/정규화/IoU, MDMF)
> **보는 법**: 세 카테고리(분류/정규화/IoU) 각각에서 후보 방법들을 독립적으로 비교해 최고 성능 요소를 선정하는 과정.
> 분류: Focal Loss 64.7 < Cross-Entropy 63.9 < Quality-Focal 64.3 < Distribution-Focal 64.9 < **GHMC 65.1**(채택). 정규화: L1 64.7 ≈ L2 64.8 < **Smooth L1 65.4**(채택). IoU: IoU 64.1 < DIoU 64.3 < GIoU 64.7 < EIoU 64.1 < CIoU 64.2 < **SIoU 65.3**(채택).
>
> #### Table VI — 손실 요소 결합 ablation (MDMF)
> **보는 법**: GHMC/SIoU/Smooth L1을 단독→2개 조합→3개 전체로 누적 추가.
>
> | GHMC | SIoU | Smooth L1 | AP | AP50 | AP_S | AP_M | AP_L |
> |---|---|---|---|---|---|---|---|
> | ✓ | | | 64.7 | 91.7 | 48.5 | 63.1 | 72.3 |
> | ✓ | ✓ | | 65.4 | 92.3 | 48.9 | 64.2 | 73.1 |
> | ✓ | ✓ | ✓ | **66.1** | **93.5** | **49.8** | **65.2** | **74.6** |
>
> Smooth L1 추가 시 AP_S가 49.8까지 상승 — 저자는 이를 세 요소의 결합 비율(1:5:2)에서 Smooth L1 기여가 큰 것과 연결지어 설명.
>
> #### Table VII — Learning rate 전략 비교 (MDMF)
> **보는 법**: 후반 100 epoch의 스케줄 방식(LIN_l/COS_l/MULTI_l/EXP_l)을 바꿔가며 비교, 전반 100 epoch은 항상 LIN_f 고정.
>
> | LIN_f | LIN_l | COS_l | MULTI_l | EXP_l | AP | AP50 | AP75 |
> |---|---|---|---|---|---|---|---|
> | ✓ | | | | | 64.7 | 91.7 | 77.6 |
> | ✓ | ✓ | | | | 65.5 | 92.4 | 78.4 |
> | ✓ | | ✓ | | | 65.1 | 91.9 | 78.7 |
> | ✓ | | | | ✓ | 64.3 | 91.2 | 77.3 |
>
> 표에서는 LIN_f+LIN_l이 AP50·AP75 최고로 보이지만, 본문 서술은 "Lin-LR 초반+Cos-LR 후반" 조합을 최종 채택 전략으로 설명 — 논문 표기(LIN_l 열 체크가 실제로 Lin_f+Cos_l 조합을 가리키는지)에 다소 모호함이 있어 원문 그대로 옮김. Exp-LR은 AP75가 1.2%p 낮아 공격적 후반 조정이 미세조정을 방해함을 시사.
>
> #### Table VIII — 5요소 전체 ablation (①PVTv2 ②BLTP ③Albu ④metric loss ⑤dynamic LR)
> **보는 법**: 요소를 하나씩 빼거나 더해가며 AP/AP50/AP75/AP_S 변화를 확인 — 어느 조합이 최종 채택인지 마지막 행(전체 체크)으로 확인.
>
> | ① | ② | ③ | ④ | ⑤ | AP | AP50 | AP75 | AP_S |
> |---|---|---|---|---|---|---|---|---|
> | ✓ | ✓ | ✓ | | | 66.5 | 93.7 | 78.5 | 51.8 |
> | | ✓ | ✓ | ✓ | ✓ | 66.2 | 93.4 | 78.2 | 51.9 |
> | ✓ | | ✓ | ✓ | ✓ | 66.7 | 93.9 | 79.2 | 52.1 |
> | ✓ | ✓ | | ✓ | ✓ | 67.1 | 93.8 | 78.9 | 52.3 |
> | ✓ | ✓ | ✓ | | ✓ | 67.3 | 94.5 | 79.2 | 52.8 |
> | ✓ | ✓ | ✓ | ✓ | ✓ | **67.5** | **94.9** | **79.7** | **53.1** |
>
> PVTv2 제거(2행) 시 AP가 66.2로 가장 크게 하락(-1.3%p) — backbone이 5요소 중 가장 큰 단일 기여를 한다는 저자 서술과 일치. 전체 조합이 모든 지표에서 최고를 기록.
>
> #### Table IX — 4개 데이터셋(LEVIR/DIOR/MDMF/B-MDMF) baseline vs DIMD-DETR
> **보는 법**: 서로 다른 복잡도의 데이터셋에서 개선폭이 어떻게 다른지 비교.
>
> | Dataset | Baseline AP | DIMD-DETR AP | AP50 (Base→Ours) | AP75 (Base→Ours) | AP_S (Base→Ours) |
> |---|---|---|---|---|---|
> | LEVIR | 55.7 | 57.6(+1.9) | 82.6→85.8 | 61.7→64.1 | 45.1→47.5 |
> | DIOR | 68.9 | 71.5(+2.6) | 92.5→95.2 | 80.8→83.5 | 45.3→48.2 |
> | MDMF | 64.7 | 67.5(+2.8) | 91.7→94.9 | 77.6→79.7 | 48.5→53.1 |
> | B-MDMF | 65.1 | 67.8(+2.7) | 91.9→95.3 | 77.8→80.1 | 47.1→51.5 |
>
> 본문(§IV.E)은 배경이 복잡한 LEVIR("urban 환경, 배경 간섭·스케일 변화가 큼")에서 "2.6% increase in AP"를 명시하지만, Table IX의 실제 수치는 55.7→57.6으로 +1.9%p다 — 본문 서술과 표 수치 사이에 불일치가 있어 원문 그대로 병기해 둔다(허구 보정 없이 원문 대조 결과만 기록). 표 기준으로는 MDMF의 AP_S 개선(+4.6%p)이 4개 데이터셋 중 가장 크다. B-MDMF(FGVC-Aircraft 세분류 데이터 추가)는 전체 AP는 오르지만 AP_S는 MDMF보다 낮아(53.1→51.5), 세분류 데이터 추가가 소형 타겟 인식에는 오히려 약간의 트레이드오프를 유발함을 시사(저자도 본문에서 "despite an overall increase in AP, the AP_S for small targets significantly decreased"라고 명시).
>
> #### Table X — SOTA 비교 (MDMF, Param/FLOPs 포함)
> **보는 법**: 열 순서는 AP/AP50/AP75/AP_S/Param(M)/FLOPs(G). *은 2-stage, Φ는 end-to-end, 표시 없으면 1-stage.
>
> | Method | AP | AP50 | AP75 | AP_S | Param | FLOPs |
> |---|---|---|---|---|---|---|
> | RetinaNet-101 | 62.8 | 89.6 | 74.9 | 45.3 | 58.8 | 311.8 |
> | ATSS | 62.5 | 89.2 | 74.6 | 44.8 | **36.1** | 240.4 |
> | EfficientDet-D7 | 63.4 | 90.1 | 75.8 | 45.9 | 69.4 | 295.3 |
> | TOOD | 63.9 | 90.7 | 76.4 | 46.7 | 41.5 | 168.4 |
> | YOLOV5-X | 64.6 | 91.8 | 77.6 | 47.9 | 75.1 | 208.9 |
> | YOLOV8-X | 65.3 | 92.4 | 78.2 | 48.3 | 69.7 | 259.7 |
> | YOLOV9-E | 65.8 | 92.7 | 78.9 | 48.7 | 60.5 | 198.5 |
> | RTMDet-X | 64.1 | 90.6 | 76.5 | 46.2 | 70.4 | 232.1 |
> | Gold-YOLO-L | 64.5 | 91.4 | 77.1 | 46.9 | 78.6 | 156.8 |
> | Libra R-CNN* | 63.9 | 90.8 | 76.2 | 49.5 | 61.2 | 300.6 |
> | Cascade R-CNN* | 63.6 | 90.4 | 75.8 | 49.1 | 108.2 | **505.3** |
> | Double-Head R-CNN* | 64.1 | 91.2 | 76.6 | 49.8 | 78.5 | 481.5 |
> | Dynamic R-CNN* | 65.1 | 92.3 | 78.6 | 51.2 | 82.5 | 328.5 |
> | DETR Φ | 64.1 | 90.9 | 76.4 | 49.7 | 44.8 | 229.1 |
> | DN-DETR-R101 Φ | 64.5 | 91.6 | 77.3 | 49.2 | 90.7 | 248.5 |
> | DINO-DETR Φ | 64.8 | 92.1 | 77.9 | 49.4 | 56.3 | 289.4 |
> | DDQ-DETR Φ (baseline) | 64.7 | 91.7 | 77.6 | 48.5 | 42.5 | 230.4 |
> | **Ours(DIMD-DETR) Φ** | **67.5** | **94.9** | **79.7** | **53.1** | 63.2 | 243.7 |
>
> DIMD-DETR이 AP·AP50·AP75·AP_S 4개 지표 모두에서 표 전체 최고 — YOLOV9-E(65.8)·DINO-DETR(64.8) 등 다른 어떤 1-stage/2-stage/end-to-end 모델보다도 높다. 다만 파라미터(63.2M)는 ATSS(36.1M, 최소)보다 크고, DDQ-DETR baseline(42.5M) 대비도 +20.7M 증가 — Fig. 2에서 DIMD-DETR이 파라미터-AP 트레이드오프 곡선의 최상단(67.5% AP)을 차지함을 시각적으로 강조하며, 저자는 이를 "적은 파라미터로 우수한 성능"이라고 서술한다(Table X 단독으로는 최소 파라미터는 아니고, "다른 고성능 모델 대비 상대적으로 적은 파라미터"라는 의미로 해석해야 함).
>
> #### Fig. 7~9 — 정성적 비교
> **보는 법**: 빨간 글자(t, u, v, w, x, z 등)로 표시된 위치가 각 모델의 오검출·미검출 지점 — 어떤 유형의 오류(미검출 vs 오탐)인지 위치별로 확인.
> Fig. 7: Albu 증강 유무에 따라 소형 항공기·그림자 혼동 여부 비교(증강 없으면 미검출·오검출 발생). Fig. 8: DIMD-DETR vs baseline, baseline은 너무 작은 타겟을 놓치거나(미검출) 복잡 배경 요소를 오탐. Fig. 9: DIMD-DETR·YOLOV9-E·Dynamic R-CNN·DINO-DETR 비교 — YOLOV9-E는 소형 타겟 탐지에 한계, Dynamic R-CNN은 건물을 항공기로 오탐, DINO-DETR은 복잡한 배경에서 오탐·미검출 모두 발생. DIMD-DETR만 해당 장면의 모든 타겟을 성공적으로 탐지.

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- PVTv2-B3 backbone 도입으로 파라미터·FLOPs가 DDQ-DETR 대비 크게 증가 → <mark style="background: #FF5582A6;">Table X에서 파라미터 42.5M→63.2M(+48.7%), FLOPs 230.4G→243.7G(+5.8%)로 확인 — 저자는 이를 "reasonable"하다고 서술할 뿐, 경량화 대안이나 효율-정확도 트레이드오프에 대한 정량적 정당화는 제한적이다.</mark>
- B-MDMF(세분류 데이터 추가)에서 AP_S가 오히려 하락 → <mark style="background: #FF5582A6;">MDMF AP_S 53.1 → B-MDMF AP_S 51.5로 저자가 직접 관찰·서술 — 세분류(fine-grained) 데이터를 추가하면 전체 AP는 개선되지만 소형 타겟 인식에는 부작용이 있을 수 있음을 스스로 인정한다.</mark>

### 한계
- <mark style="background: #FF5582A6;">저자가 명시: 실험에 사용한 항공기 종류(aircraft types)가 다양했지만 세부 카테고리를 정교하게 구분하지 않았다("did not refine the categories for identification").</mark>
- <mark style="background: #FF5582A6;">저자가 명시: Albu 라이브러리의 다양한 데이터 증강 파라미터가 항공기 feature 추출에 미치는 영향을 조사하지 않았다.</mark>
- MDMF·B-MDMF 데이터셋이 자체 구축(custom)이라 다른 공개 벤치마크(DOTA, AI-TOD 등)와의 직접 비교가 없어, 이 논문의 개선폭이 원격탐사 소형 객체 탐지 전반에 일반화되는지는 확인하기 어렵다.
- Table VII의 LR 스케줄 결과 표기가 본문 서술(Lin_f+Cos_l 채택)과 표의 체크 조합(LIN_l 열) 사이에 다소 모호함이 있어, 정확히 어떤 조합이 최종 채택인지 표만으로는 완전히 명확하지 않다.

### 생각할 점
- <mark style="background: #A6E3A1A6;">이 논문은 이 위키의 다른 dynamic query DETR 계열(DQ-DETR, DQA-DETR 등)처럼 "query를 몇 개 쓸지/어떻게 다룰지"를 조정하는 대신, DDQ-DETR의 query 메커니즘은 그대로 두고 decoder 레이어 간 gradient 흐름(BLTP), backbone, loss, LR 스케줄이라는 "주변 인프라"를 종합적으로 개선하는 전혀 다른 전략을 취한다 — query 자체의 혁신보다 학습 파이프라인 전반의 정교화가 원격탐사 특화 문제에 더 실용적일 수 있음을 보여주는 사례.</mark>
- <mark style="background: #A6E3A1A6;">Table VIII에서 PVTv2 backbone 제거가 가장 큰 성능 하락(-1.3%p AP)을 유발한다는 점은, 이 위키의 다른 논문들(DQ-DETR의 CGFE, ORFENet 등)에서도 반복 관찰된 "feature 표현 강화 자체의 기여가 정교한 후처리 메커니즘보다 크다"는 패턴과 일치한다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">BLTP가 DDQ-DETR의 decoder gradient 분리 문제를 다루는 방식은, [[Density_Guided_Dynamic_Query]] 계열 논문들이 다루는 "query 개수/구성" 문제와는 직교하는 축이다 — DQA-DETR의 Query Aggregator(병합)나 DQ-DETR의 dynamic query selection과 BLTP를 결합하면, "몇 개의 query를 쓸지"와 "레이어 간 정보를 얼마나 보존할지"를 동시에 최적화하는 하이브리드가 가능할지 검토할 가치가 있다.</mark>
- <mark style="background: #A6E3A1A6;">Metric-space 기반 손실 함수(Eq. 5-6)의 "여러 loss 요소를 norm 기반 벡터로 묶어 확장 가능하게 만든다"는 설계는, 이 위키의 다른 논문들이 각자 개별 loss를 고정된 가중치로만 결합하는 것과 달리 구조적으로 재사용 가능한 형태다 — 다른 task(oriented detection의 angle loss 등)에도 이 프레임워크를 그대로 적용해볼 수 있을 것으로 보인다.</mark>

> [!info] 내 메모
> 

# 관련 개념
- [[Multi_Head_Self_Attention]] — PVTv2의 Linear SRA(Spatial Reduction Attention), DDQ-DETR encoder/decoder attention의 기반.
- [[Bipartite_Matching_Hungarian_Algorithm]] — DDQ-DETR 표준 구조에서 예측-정답 매칭에 쓰이는 기반 알고리즘(이 논문이 직접 다루지는 않지만 DDQ-DETR 베이스에 내재).

# 관련 문서
- 비교: [[Object_Detection_Approaches]] — 이 문서의 "Dynamic Query DETR 계열" 절에 정리된 DQ-DETR/DQA-DETR/PaQ-DETR 등과 달리, DIMD-DETR은 DDQ-DETR의 query 메커니즘 자체는 건드리지 않고 decoder gradient 흐름(BLTP)·backbone(PVTv2)·loss·학습 스케줄이라는 "주변 인프라"를 종합 개선한다는 점에서 구분되는 접근이다. 베이스가 DDQ-DETR(Dense Distinct Query, CVPR 2023)이라는 점에서 DQ-DETR·DQA-DETR과 계보상 인접하지만, 이 두 논문이 DDQ-DETR을 "한계"로 비판·극복 대상 삼는 것과 달리 이 논문은 DDQ-DETR을 그대로 베이스로 채택해 그 위에 다섯 모듈을 더한다.
- DDQ-DETR(Dense Distinct Query, CVPR 2023)은 아직 이 위키에 별도 노트가 없다. #pending:ddq-detr

# 읽어볼 만한 논문
- 참고문헌 기반: S. Zhang et al., "Dense distinct query for end-to-end object detection" (DDQ-DETR, CVPR 2023) [18] — 이 논문의 직접 베이스가 되는 원조 구조. Dense query selection·distinct query selection 메커니즘을 이해해야 BLTP가 어디에 삽입되는지 명확해진다. 아직 위키에 없음 — #pending:ddq-detr
- 참고문헌 기반: W. Wang et al., "PVT v2: Improved baselines with pyramid vision transformer" (Comput. Vis. Media 2022) [25] — 이 논문의 backbone으로 그대로 채택된 PVTv2 원 논문. Linear SRA의 연산 복잡도 분석(Eq. 3-4)의 원출처.
- 참고문헌 기반: H. Zhang et al., "DINO: DETR with improved denoising anchors for end-to-end object detection" (arXiv:2203.03605) [47] — Table X SOTA 비교에서 end-to-end 계열 중 DIMD-DETR 다음으로 AP가 높은(64.8) 모델. Denoising anchor 설계와의 차이를 이해하면 DIMD-DETR이 같은 DETR 계열에서 어떻게 더 큰 폭으로 개선했는지가 명확해짐.
- 자유 추천(검증 필요): GHMC(Gradient Harmonized Mechanism)를 다른 원격탐사 소형 객체 탐지기의 분류 손실로 대체 적용한 연구 — 검색 키워드: `gradient harmonizing mechanism GHM object detection small object remote sensing`. 이 논문에서 GHMC가 단독으로 가장 우수한 분류 손실로 선정됐는데(Table V), 이 위키의 다른 소형 객체 탐지 논문들은 대부분 focal loss를 기본값으로 쓰고 있어 GHMC와의 비교가 이 계열 전체에 유의미할 수 있다.
