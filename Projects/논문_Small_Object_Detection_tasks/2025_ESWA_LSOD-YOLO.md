---
pm-task: true
projectId: "paperwiki-small-object-detection"
parentId:
id: "t-lsod-yolo-bwwbsaqyex"
title: "Precision and speed: LSOD-YOLO for lightweight small object detection"
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
  "1frf59rymtcjvske": "Expert Systems With Applications"
subtaskIds: []
dependencies: []
year: 2025
venue: "Expert Systems With Applications"
jcr_quartile: Q1
task: [small-object-detection]
direction: [improvement]
paper_tags: [paper, small-object-detection, yolo, lightweight, attention-mechanism, uav]
source: "Projects/논문_pdf/Small_Object_Detection/2025_ESWA_LSOD-YOLO.pdf"
source_type: personal
createdAt: "2026-08-18T11:00:00.000Z"
updatedAt: "2026-08-31T00:00:00.000Z"
---

#paper #small-object-detection #yolo #lightweight #attention-mechanism #uav

> [!quote] 원제
> **Precision and speed: LSOD-YOLO for lightweight small object detection**
> Hezheng Wang, Jiahui Liu, Jian Zhao, Jianzhong Zhang, Dong Zhao — School of Technology / Key Lab of State Forestry Administration on Forestry Equipment and Automation / State Key Laboratory of Efficient Production of Forest Resources, Beijing Forestry University, Expert Systems With Applications 2025
> https://doi.org/10.1016/j.eswa.2025.126440

# 한 줄 요약
<mark style="background: #FFF3A3A6;">YOLOv8s의 P5 검출 헤드를 제거하고 대신 P2 헤드를 cross-layer connection과 함께 추가하는 LCOR을 중심으로, SPPFL(LSKA)·C2f-N(NAM)·Dysample을 결합해 VisDrone2019에서 파라미터를 65.5% 줄이면서도 mAP0.5를 오히려 끌어올린 경량 소형 객체 검출 모델.</mark>

> [!info] 내 메모
> 

# 정리

## 기존 방법의 한계
- **저해상도 feature의 위치 정보 손실**:
  소형 객체는 픽셀 표현이 적고 밀집 분포·복잡한 배경 속에 있어, 다운샘플링을 거친 저해상도 feature map(P4/P5)에서는 각 픽셀이 넓은 영역을 대표하게 되어 위치 정보 손실이 크고 미검출·오검출이 잦다.
- **정확도 개선 기법의 연산량 증가**:
  TPH-YOLOv5, Ji et al., FE-YOLOv5 등은 소형 객체 검출 정확도를 높이지만 대부분 파라미터 수가 크게 늘어난다.
- **P2 헤드 단순 추가의 트레이드오프**:
  소형 객체 검출력을 높이기 위해 Neck에 P2 layer(160×160)를 추가하고 검출 헤드를 하나 더 붙이는 방법(YOLOv8s-P2류)이 일반적으로 쓰이지만, 검출 헤드가 3개→4개로 늘어나면서 연산 복잡도(GFLOPs)가 크게 증가한다.

## 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — 경량화 중심**
- YOLO-S(Betti & Tucci 2023): Darknet20 기반 경량 feature extraction network — 속도는 개선되지만 backbone의 feature 추출 용량이 제한되어 정확도 하락.
- **타겟/해결**: 저해상도 feature의 위치 정보 손실(문제 ①) — 경량화는 이뤘지만 "P2 고해상도 feature를 살리면서 동시에 연산량은 늘리지 않는" 설계는 시도되지 않았다.

**갈래 2 — 정확도 중심(경량성 희생)**
- TPH-YOLOv5[Transformer prediction head], Ji et al.[multi-scale context+Soft-CIOU], FE-YOLOv5[deformable conv], FFNB[얕은/깊은 feature 병합], STC-YOLO, SO-YOLOv5[coordinate attention], Wu et al.[residual coordinate attention]: 각기 다른 방식으로 소형 객체 검출 정확도 향상.
- **타겟/해결**: 정확도 개선 기법의 연산량 증가(문제 ②) — 공통적으로 정확도는 개선되지만 파라미터·연산량이 크게 증가해 실시간/자원 제약 환경에 부적합하다.

**갈래 3 — P2 헤드 단순 추가**
- <mark style="background: #FFF3A3A6;">YOLOv8s-P2류: P2에 헤드를 그대로 추가(Fig. 2b) — 헤드가 3→4개로 늘어 연산량이 증가한다.</mark>
- **타겟/해결**: P2 헤드 단순 추가의 트레이드오프(문제 ③) — 헤드 추가의 이득(정확도)과 비용(연산량)을 분리해서 다루지 않는다.

**갭**: <mark style="background: #FFF3A3A6;">세 갈래(경량화 중심·정확도 중심·P2 단순 추가)가 각각 정확도 또는 경량성 한쪽만 정면으로 다뤄왔고, "P2 고해상도 feature를 살리면서 동시에 연산량은 늘리지 않는" 설계는 시도되지 않았다.</mark>

## 이 논문이 풀고자 하는 문제
1. 소형 객체 검출에 유리한 고해상도 P2 feature를 검출에 활용하되, 헤드 추가로 인한 연산량 증가를 상쇄할 방법을 찾는 것.
2. 정확도(mAP0.5)와 경량성(Params/Model size/GFLOPs/FPS)을 동시에 만족하는 균형점을 찾는 것.

**갭 종합**: <mark style="background: #FFF3A3A6;">이 논문은 P2 헤드를 단순히 더하는 대신, 기여도가 낮은 P5 헤드를 제거해 얻은 여유를 P2 헤드에 재배분하는 방식으로 이 갭을 메운다.</mark>

> [!info] 내 메모
> 

# 해결 방법 요약

| | 문제 ① — 저해상도 feature의 위치 정보 손실 | 문제 ② — 정확도 개선 기법의 연산량 증가 | 문제 ③ — P2 헤드 단순 추가의 트레이드오프 |
|---|---|---|---|
| **해결 방법** | LCOR가 저기여도 P5 검출 헤드를 제거해 확보한 여유를 P2 헤드 추가에 재배분 | SPPFL·C2f-N·Dysample이 각각 저비용 attention/upsampling으로 LCOR의 부수적 손실을 보완 | Cross-layer connection으로 P5 제거로 인한 정보 손실을 P3→P2 정보 전달로 보완 |
| **예상되는 문제점** | Branch 4개를 병렬로 두는 구조는 아니지만, 헤드를 빼고 넣는 구조 자체가 P5가 담당하던 대형 객체 검출력에 영향을 줄 가능성(정량 분리 검증 없음) | 세 보완 모듈 각각의 경량성은 검증됐으나(Table 8: attention 없음 3.755M→C2f-N 3.755M), VisDrone(드론뷰) 특화 설계가 대형 객체 비중이 높은 다른 도메인에 그대로 전이될지는 불확실 | P5 제거가 대형 객체 검출력을 희생시킬 가능성(아래 Discussion 참고) |

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">저기여도 고레벨 검출 헤드(P5)를 제거해 확보한 여유를 고해상도 저레벨 검출 헤드(P2) 추가에 재배분하고, 둘 사이에 cross-layer connection을 둬 정보 손실을 보완하는 <span style="color:#c0392b; font-weight:bold;">LCOR(Lightweight Cross-layer output Reconstruction)</span>을 중심으로, attention 기반 SPP(SPPFL), attention 기반 C2f(C2f-N), 경량 upsampler(Dysample)를 결합해 이 재배분으로 인한 부수적 손실을 추가로 보완한다.</mark>

## 전체 파이프라인 (Fig. 1, Fig. 2c 기준)

```
입력 이미지 (3, 640, 640)
       │
       ▼
Backbone: CBS → CBS → C2f → CBS → C2f → CBS → C2f → CBS → C2f → SPPFL
                     │(160,160,128)  │(80,80,256)   │(40,40,512, SPPFL 통과)
       ▼
① LCOR: P5 검출 헤드 제거 + P2 헤드 추가 + cross-layer connection(P3↔P2)
       │
       ▼
② SPPFL (Neck 최상단, backbone SPPF 대체)         → (40,40,512)  [LSKA로 global 정보 보완]
       │
       ▼
Neck: Dysample(40→80) → Concat(P4) → C2f-N → Dysample(80→160) → Concat(P2/P3) → C2f-N
       │                                                                │
       │                                              ┌─────────────────┘
       ▼ top-down                                     ▼ bottom-up(Concat+C2f-N, skip connection 포함)
③ C2f-N ×4 (Neck 전체, NAM attention)              → P2'(160,160,128) / P3'(80,80,256)
       │
       ▼
Decoupled Head × 2 (P2', P3'만 사용)                → (160,160,·) / (80,80,·) 두 스케일 예측
```

> [!info] 내 메모
> 

### ① LCOR (Lightweight Cross-layer output Reconstruction)
- **역할**:
  YOLOv8 backbone은 P1~P5 5단계 다운샘플링을 거쳐 80×80/40×40/20×20 세 검출 헤드를 사용한다(Fig. 2a). 저해상도 feature map은 픽셀당 대표 영역이 넓어 위치 정밀도가 낮은 반면, 고해상도 P2(160×160)는 edge·shape 표현에 유리하다. <span style="color:#c0392b; font-weight:bold;">LCOR(Lightweight Cross-layer output Reconstruction)</span>은 P2를 살리되 헤드 추가의 연산 비용을 없애기 위한 재배분 전략이다.
- **구현**:
  기존 방법(YOLOv8s-P2)은 P2에 헤드를 그대로 추가(Fig. 2b) — 헤드가 3→4개로 늘어 연산량이 증가한다. LCOR는 대신 (a) semantic 정보 위주지만 소형 객체 민감도가 낮은 **P5 검출층을 제거**, (b) **P2 검출 헤드를 추가**, (c) 얕은 layer(P2/P3)와 깊은 layer 사이에 **cross-layer connection**(skip connection)을 둬 두 정보를 함께 보존한다(Fig. 2c). 결과적으로 최종 검출 헤드는 160×160(P2)과 80×80(P3) 두 스케일만 사용한다.
- **입출력 shape**:
  Backbone 5-scale feature `P1~P5` → 검출에 쓰이는 feature는 `P2'(160,160,·)`, `P3'(80,80,·)` 두 스케일만(기존 3~4스케일 대비 축소).

<mark style="background: #FFF9D6A6;">P5 제거와 P2 추가는 "헤드 하나를 빼고 하나를 더하는" 상쇄 구조라 P2 단순 추가 방식이 동반하던 연산량 증가를 원천적으로 없애고, P5가 담당하던 semantic 정보 손실은 cross-layer connection으로 P3 정보를 P2에 전달해 보완한다 — 두 목표(정확도 유지·경량화)가 같은 설계 변경 안에서 함께 달성된다. Table 6 ablation에서 LCOR 제거 시 파라미터가 3.8M→12.3M로 3배 이상 급증하고 mAP0.5도 37.0%→35.8%로 가장 크게 하락하는 것이 LCOR가 경량화의 절대적 핵심임을 뒷받침한다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> P5는 저해상도지만 수용영역이 가장 넓어 일반적으로 대형 객체에 유리한 레벨이다. 이를 제거하면 대형 객체 검출력에 부정적 영향이 있을 수 있으나, 논문은 P5 제거만의 순수 영향을 다른 요인(P2 추가 등)과 분리해 검증하지 않는다(아래 Discussion 참고).

> [!info] 내 메모
> 

### ② SPPFL (SPPF + [[Large_Separable_Kernel_Attention]])
- **역할**:
  LCOR로 계층이 단순화되며 고수준 semantic 정보가 줄어들 수 있다는 우려에 대응해, backbone 최상단의 SPPF(multi-scale pooling으로 feature 통합) 출력에 attention을 결합해 헤드나 레이어 추가 없이 semantic/global 정보를 보완하는 모듈이다.
- **구현**:
  기존 SPPF는 3회 연속 maxpool + concat으로 multi-scale feature를 통합한다(Fig. 3a). SPPFL은 이 출력에 [[Large_Separable_Kernel_Attention]](LSKA, Lau et al. 2024)를 결합한다(Fig. 3b) — 수평/수직 1D depthwise conv + depthwise-dilated conv로 넓은 수용영역을 확보한 뒤, 1×1 conv로 attention map을 만들고 원본 feature와 Hadamard product로 정제한다.
- **입출력 shape**:
  backbone 최종 feature `(512, 40, 40)` → SPPF multi-scale pooling → LSKA attention → `(512, 40, 40)`(shape 불변, semantic 정보가 attention으로 재분배된 값).

```python
# LSKA 상세 구현은 Architecture Design 참고
sppf_out = SPPF(backbone_feat)              # 3x maxpool + concat
sppfl_out = LSKA(sppf_out)                  # 수평/수직 1D conv + dilated conv + 1x1 conv attention
```

<mark style="background: #FFF9D6A6;">LCOR로 단순화되며 손실될 수 있는 semantic/global 정보를, 헤드나 레이어를 추가하지 않고도 attention 가중치 재분배만으로 보완한다 — LSKA의 분해된 1D convolution 구조 덕분에 넓은 수용영역을 확보하면서도 연산량 증가가 크지 않다.</mark>

> [!info] 내 메모
> 

### ③ C2f-N (C2f + [[Normalization_Based_Attention_Module]])
- **역할**:
  Neck의 저해상도 feature map에서 복잡한 배경 속 비중요 feature를 억제해, C2f 모듈의 소형 객체 feature 추출 능력을 강화하는 모듈이다.
- **구현**:
  [[Normalization_Based_Attention_Module]](NAM, Liu et al. 2021)을 C2f 모듈의 residual branch 출력에 삽입한 구조(Fig. 6). NAM은 CBAM 구조에서 착안하되, Batch Normalization의 scaling factor `γ`(channel)와 pixel normalization scaling factor `λ`(spatial)를 그대로 attention 가중치로 재활용하고, sparsity 정규화 항으로 비중요 feature를 억제한다.
- **입출력 shape**:
  C2f 모듈 입출력과 동일 — Neck 각 레벨 feature `(C, H, W)` → `(C, H, W)`(shape 불변, NAM으로 정제).

<mark style="background: #FFF9D6A6;">이미 계산되어 있는 BN scaling factor를 재사용해 attention을 만들기 때문에 추가 파라미터·연산 비용이 거의 없다(Table 8: attention 없음 3.755M → C2f-N 3.755M로 사실상 불변). 복잡한 배경에서 비중요 feature를 억제해 별도의 무거운 attention 모듈 없이도 정확도를 끌어올려 "정확도 개선을 경량성 손실 없이" 달성한다 — Table 8에서 C2f-CBAM(36.6)·C2f-CA(36.7)·C2f-SA(36.7)·C2f-SE(36.9)보다 C2f-N(37.0)이 최소 파라미터로 최고 성능을 기록한 것이 이를 뒷받침한다.</mark>

> [!info] 내 메모
> 

### ④ [[Dysample_Dynamic_Upsampling]]
- **역할**:
  기존 nearest-neighbor 업샘플링은 픽셀 공간 위치만으로 kernel을 결정해 content 정보를 활용하지 못하고, 소형 객체에서 pixel distortion·정보 손실을 유발한다. Dysample은 이를 대체해 content-aware하면서도 저비용인 업샘플링을 제공한다.
- **구현**:
  [[Dysample_Dynamic_Upsampling]](Liu et al. 2023)은 CARAFE류와 달리 dynamic convolution이나 별도 sub-network 없이, "linear layer + pixel shuffle"만으로 샘플링 위치 offset을 동적 생성한다(Fig. 7). Neck의 top-down 경로에서 기존 nearest-neighbor 업샘플링을 대체.
- **입출력 shape**:
  `(c, h, w)` → `(c, sh, sw)`(s=업샘플 배율, 예: 2배).

<mark style="background: #FFF9D6A6;">동적 sub-network나 dynamic convolution을 쓰는 CARAFE 대비 훨씬 적은 연산으로 content-aware upsampling 효과를 얻어, upsampling 품질(정확도)과 연산량(경량성) 사이의 트레이드오프 자체를 개선한다 — Table 7에서 nearest-neighbor(mAP0.5 36.7, GFLOPs 33.8) · CARAFE(36.9, 35.8) 대비 Dysample(37.0, 33.9)이 정확도·연산량·FPS(93) 모두에서 우위를 보인 것이 이를 뒷받침한다.</mark>

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| ① LCOR | Backbone 5-scale (P1~P5) | 검출용 P2'(160,160,·), P3'(80,80,·) | P5 제거+P2 추가 재배분, 경량화 핵심 | 헤드 상쇄 구조 + cross-layer connection |
| ② SPPFL | Backbone 최상단 (512,40,40) | (512,40,40) | semantic/global 정보 보완 | SPPF + [[Large_Separable_Kernel_Attention]] |
| ③ C2f-N ×4 | Neck 각 레벨 (C,H,W) | 동일 (C,H,W) | 비중요 feature 억제 | C2f + [[Normalization_Based_Attention_Module]] |
| ④ Dysample | (c,h,w) | (c,sh,sw) | content-aware 저비용 업샘플링 | linear layer + pixel shuffle + grid sample |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 (VisDrone2019, Table 2)

| 모델 | mAP0.5 | Params(M) | Model Size(MB) |
|---|---|---|---|
| YOLOv8s | 34.5% | 11.0 | 22.5 |
| YOLOv8s-P2 | 36.9% | 10.9 | 21.7 |
| LSOD-YOLO | 37.0% | 3.8 (−65.5%) | 7.6 (−66.2%) |

> [!note]- 세부 결과 및 Ablation
> #### 설정
> - **주 데이터셋**: VisDrone2019(드론 항공 영상, train 6,471 / val 548 / test 1,610장, 10 class, 약 54만 인스턴스). 32×32px 미만 객체가 전체의 44.70%(tiny 12.05% + small 32.65%). Pedestrian/People 클래스는 각각 64.59%/77.44%가 소형·극소형.
> - **일반화 검증 데이터셋**: TinyPerson, LEVIR-Ship, UAVDT
> - **플랫폼**: RTX 4090, PyTorch 2.2.1, CUDA 12.1.1, 입력 640×640, 300 epoch, batch 16, lr 0.01, momentum 0.937, weight decay 0.0005, SGD
>
> #### Baseline 대비 전체 지표 (Table 2)
> | 모델 | P(%) | R(%) | mAP0.5(%) | Params(M) | FPS | Model Size(MB) | GFLOPs |
> |---|---|---|---|---|---|---|---|
> | YOLOv8s | 45.2 | 36.3 | 34.5 | 11.0 | 90 | 22.5 | 28.8 |
> | YOLOv8s-P2 | 48.4 | 38.4 | 36.9 | 10.9 | 92 | 21.7 | 39.7 |
> | LSOD-YOLO | 48.4 | 38.2 | 37.0 | 3.8 | 93 | 7.6 | 33.9 |
>
> YOLOv8s-P2 대비: GFLOPs −5.8, 파라미터 −7.1M, 모델 크기 −14.1MB이면서 mAP0.5는 오히려 +0.1%p — "고해상도 헤드 추가"의 정확도 이득은 유지하면서 비용만 제거한 형태. YOLOv8s 대비로는 P +3.2%p, R +1.9%p, mAP0.5 +2.5%p.
>
> #### 클래스별 mAP0.5 비교 (Table 3)
> | 모델 | Pedestrian | People | Bicycle | Car | Van | Truck | Tricycle | Awning-tricycle | Bus | Motor |
> |---|---|---|---|---|---|---|---|---|---|---|
> | YOLOv8s | 33.6 | 21.6 | 11.4 | 75.3 | 39.4 | 37.9 | 18.7 | 18.8 | 55.7 | 33.2 |
> | YOLOv8s-P2 | 35.7 | 23.3 | 12.2 | 76.5 | 40.2 | 39.7 | 21.1 | 22.4 | 59.6 | 35.8 |
> | LSOD-YOLO | 35.8 | 23.4 | 12.4 | 76.6 | 41.5 | 40.1 | 21.7 | 22.1 | 59.7 | 36.1 |
>
> LSOD-YOLO는 대표적인 소형 객체 클래스(Pedestrian, People)에서 최고 성능이면서, Bus·Truck 같은 상대적으로 큰 객체에서도 YOLOv8s-P2와 대등하거나 근소하게 우위 — P5 제거가 대형 객체 성능을 크게 희생시키지는 않은 것으로 보이나, 크기별 세부 breakdown은 논문에 없음.
>
> #### 다른 경량/SOTA 검출기 대비 (Table 4)
> | 모델 | mAP0.5(%) | Params(M) | FPS | Model Size(MB) | GFLOPs |
> |---|---|---|---|---|---|
> | SSD | 22.1 | 24.5 | 66 | 95.2 | 3.1 |
> | YOLOv3-Tiny | 20.1 | 12.2 | 165 | 24.4 | 19.1 |
> | YOLOv5s | 27.5 | 7.0 | 78 | 14.4 | 16.5 |
> | YOLOv5m | 30.3 | 20.9 | 89 | 42.2 | 49.0 |
> | YOLOv6s | 31.3 | 16.4 | 125 | 32.8 | 44.0 |
> | YOLOv7-tiny | 29.5 | 6.0 | 68 | 12.3 | 13.1 |
> | YOLOv9-c | 39.7 | 50.7 | 33 | 102.9 | 236.7 |
> | YOLOv10s | 32.3 | 8.0 | 105 | 15.7 | 24.5 |
> | LSOD-YOLO | 37.0 | 3.8 | 93 | 7.6 | 33.9 |
>
> YOLOv9-c만 mAP0.5가 더 높지만 파라미터 13배(50.7M), 모델 크기 14배(102.9MB), FPS 33에 불과. LSOD-YOLO는 파라미터·모델크기·FPS 균형에서 비교 모델 대부분을 상회.
>
> #### 클래스별 mAP0.5 전체 비교 (Table 5)
> LSOD-YOLO가 Pedestrian(35.8)·People(23.4) 등 소형 객체 비중이 높은 클래스에서 SSD/YOLOv3-Tiny/YOLOv5계열/YOLOv6s/YOLOv7-tiny/YOLOv9-c/YOLOv10 대비 대체로 최고 또는 최상위권. YOLOv9-c가 Van(46.3)·Bus(66.5) 등 일부 대형 객체 클래스에서는 더 높음.
>
> #### Ablation — LCOR (Table 6)
> | 모델 | mAP0.5(%) | Params(M) | GFLOPs |
> |---|---|---|---|
> | YOLOv8s | 34.5 | 11.0 | 28.8 |
> | YOLOv8s-P2 | 36.9 | 10.9 | 39.7 |
> | LSOD-YOLO(cross-layer connection 제외) | 36.5 | 3.7 | 33.6 |
> | LSOD-YOLO | 37.0 | 3.8 | 33.9 |
>
> cross-layer connection 하나만 추가해도 GFLOPs +0.3 대비 mAP0.5 +0.5%p — 연산 비용 대비 이득이 큼.
>
> #### Ablation — Dysample (Table 7)
> | 방식 | mAP0.5(%) | Params(M) | FPS | Model Size(MB) | GFLOPs |
> |---|---|---|---|---|---|
> | nearest-neighbor | 36.7 | 3.8 | 90 | 7.8 | 33.8 |
> | CARAFE | 36.9 | 3.9 | 83 | 7.8 | 35.8 |
> | Dysample | 37.0 | 3.8 | 93 | 7.6 | 33.9 |
>
> #### Ablation — C2f attention 모듈 비교 (Table 8)
> | 모듈 | P(%) | R(%) | mAP0.5(%) | Params(M) | GFLOPs |
> |---|---|---|---|---|---|
> | attention 없음 | 47.8 | 37.8 | 36.5 | 3.755 | 33.8 |
> | C2f-CBAM | 47.4 | 38.3 | 36.6 | 3.762 | 33.9 |
> | C2f-CA | 48.5 | 37.7 | 36.7 | 3.762 | 33.9 |
> | C2f-SA | 47.9 | 37.8 | 36.7 | 3.756 | 33.9 |
> | C2f-SE | 47.8 | 38.1 | 36.9 | 3.758 | 33.9 |
> | C2f-N | 48.4 | 38.2 | 37.0 | 3.755 | 33.9 |
>
> C2f-N이 최소 파라미터로 최고 mAP0.5·R 달성 (P는 C2f-CA가 0.1%p 더 높음). CBAM이 SE·SA보다 소폭 낮은 것은 저자가 "복합 attention이 이 task에 반드시 더 유리하지는 않음"을 시사한다고 해석.
>
> #### 전체 Ablation (Table 9, ✓=포함 −=제외)
> | 구성 | LCOR | SPPFL | Dysample | C2f-N | mAP0.5(%) | Params(M) | FPS | Model Size(MB) | GFLOPs |
> |---|---|---|---|---|---|---|---|---|---|
> | LSOD-YOLO(전체) | ✓ | ✓ | ✓ | ✓ | 37.0 | 3.8 | 93 | 7.6 | 33.9 |
> | 1: LCOR 제외 | − | ✓ | ✓ | ✓ | 35.8 | 12.3 | 94 | 24.7 | 29.7 |
> | 2: SPPFL 제외 | ✓ | − | ✓ | ✓ | 36.2 | 3.5 | 91 | 7.0 | 33.0 |
> | 3: Dysample 제외 | ✓ | ✓ | − | ✓ | 36.7 | 3.7 | 90 | 7.5 | 33.8 |
> | 4: C2f-N 제외 | ✓ | ✓ | ✓ | − | 36.6 | 3.8 | 89 | 7.5 | 33.8 |
> | 5: LCOR+SPPFL 제외 | − | − | ✓ | ✓ | 36.0 | 11.2 | 90 | 22.6 | 28.9 |
> | 6: LCOR+Dysample 제외 | − | ✓ | − | ✓ | 35.8 | 12.2 | 95 | 24.7 | 29.7 |
> | 7: Dysample+C2f-N 제외 | ✓ | ✓ | − | − | 35.6 | 12.3 | 98 | 24.7 | 28.8 |
> | YOLOv8s(baseline) | − | − | − | − | 34.5 | 11.2 | 90 | 22.5 | 28.8 |
>
> - **LCOR가 경량화의 절대적 핵심**: 제외 시 파라미터 3.8M→12.3M로 3배 이상 급증, mAP0.5도 37.0%→35.8%로 가장 크게 하락.
> - **SPPFL 제외는 파라미터엔 거의 영향 없이(3.5M) mAP0.5만 36.2%로 하락** — 순수하게 정확도 보완 역할.
> - **LCOR+Dysample 동시 제외(구성 6)**는 LCOR 단독 제외(구성 1)와 거의 같은 mAP0.5(35.8% vs 35.8%) — LCOR의 영향력이 압도적.
> - **Dysample+C2f-N 동시 제외(구성 7)**는 mAP0.5 35.6%로, 각각 단독 제외(36.7%, 36.6%)보다 더 크게 하락 — 두 모듈 간 synergy 효과.
>
> #### 일반화 실험 (Table 10)
> | 데이터셋 | 모델 | P(%) | R(%) | mAP0.5(%) | mAP0.5:0.95(%) |
> |---|---|---|---|---|---|
> | TinyPerson | YOLOv5s | 45.9 | 27.6 | 26.6 | 10.1 |
> | TinyPerson | YOLOv8s | 46.0 | 25.5 | 26.1 | 11.7 |
> | TinyPerson | YOLOv8s-P2 | 49.6 | 33.5 | 34.3 | 14.9 |
> | TinyPerson | LSOD-YOLO | 51.0 | 34.7 | 35.2 | 15.6 |
> | LEVIR-Ship | YOLOv8s | 82.9 | 71.0 | 78.3 | 30.6 |
> | LEVIR-Ship | YOLOv8s-P2 | 82.4 | 69.8 | 76.9 | 29.9 |
> | LEVIR-Ship | LSOD-YOLO | 83.0 | 73.2 | 79.5 | 31.3 |
> | UAVDT | YOLOv8s | 30.3 | 42.3 | 33.6 | 20.3 |
> | UAVDT | YOLOv8s-P2 | 42.1 | 33.9 | 33.6 | 20.3 |
> | UAVDT | LSOD-YOLO | 48.1 | 37.9 | 37.1 | 22.1 |
>
> TinyPerson·LEVIR-Ship·UAVDT 세 데이터셋 모두에서 LSOD-YOLO가 YOLOv5s, YOLOv8s, YOLOv8s-P2, YOLOv7-tiny를 상회(TinyPerson mAP0.5 기준 YOLOv5s 대비 +8.6%p, YOLOv8s 대비 +9.1%p, YOLOv8s-P2 대비 +0.9%p) — 드론뷰(VisDrone) 특화 설계가 사람 탐지·선박 탐지·다른 드론뷰에도 일관되게 전이됨을 시사.
>
> #### Fig. 10 — Grad-CAM 시각화
> **보는 법**: attention 모듈별로 class activation map을 비교 — 밝을수록 모델이 그 위치에 주목. C2f-N·C2f-SE가 복잡한 장면(도로 교통)에서 다른 attention보다 target 영역에 더 집중된 활성화를 보임.
>
> #### Fig. 8, Fig. 9 — 정성적 비교 및 confusion matrix
> 밀집·다중스케일·야간·가림(occlusion) 네 시나리오 모두에서 LSOD-YOLO가 YOLOv8s·YOLOv8s-P2 대비 더 많은 소형 객체를 정확히 검출. Confusion matrix(Fig. 9)에서 LSOD-YOLO의 대표 클래스 "Pedestrian" 예측 정확도가 YOLOv8s(0.29)·YOLOv8s-P2(0.31) 대비 0.33으로 가장 높고, 오분류 값도 전반적으로 낮음.

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- **P5 제거가 대형 객체 검출을 희생시킬 가능성**:
  P5는 저해상도지만 수용영역이 가장 넓어 일반적으로 대형 객체에 유리한 레벨. <mark style="background: #FF5582A6;">논문은 이를 직접 검증하지 않는다 — Table 3에서 "Bus"(대형 객체 비중 80.10%)가 55.7→59.7로 개선되긴 하나 P2 추가 등 다른 요인과 뒤섞여 P5 제거만의 순수 영향을 분리하지 않으며, Table 9 ablation도 크기별 breakdown이 없다.</mark>
- **VisDrone 특화 설계가 다른 도메인에 그대로 전이될지 불확실**:
  LCOR의 P5 제거는 "드론 항공뷰는 대형 객체 비중이 낮다"는 이 데이터셋 특성에 최적화된 것일 수 있다. <mark style="background: #FF5582A6;">일반화 검증에 쓰인 TinyPerson/LEVIR-Ship/UAVDT도 모두 소형 객체 비중이 높은 항공/감시 도메인이라, 대형 객체가 흔한 COCO류 벤치마크에서의 검증은 없다.</mark>

### 한계
- <mark style="background: #FF5582A6;">저조도(poor lighting) 조건에서의 검출 성능 개선이 아직 부족함을 저자들이 명시 — 향후 image enhancement, adaptive learning strategy 도입을 계획으로 남김.</mark>
- <mark style="background: #FF5582A6;">불규칙한 객체 분포(irregular target distributions)에 대응하는 동적 anchor box 전략은 아직 없음 — 향후 과제로 명시.</mark>
- <mark style="background: #FF5582A6;">LCOR 자체의 한계로 논문이 인정: hierarchy simplification으로 고수준 semantic 정보가 줄어들 수 있음(SPPFL로 보완하나 완전히 해소됐는지 별도 검증 없음).</mark>
- 저자들도 실시간 적용성을 위해 model pruning, quantization 등 추가 경량화가 필요하며 모바일 기기 최적화가 향후 목표라고 명시 — 이 논문의 경량화도 종착점이 아니라 중간 단계.

### 생각할 점
- <mark style="background: #A6E3A1A6;">"저기여도 헤드 제거 + 고기여도 헤드 추가"라는 LCOR의 상쇄 설계는 YOLO의 detection head 구조를 넘어, encoder-decoder 구조를 가진 다른 dense prediction 태스크(segmentation, keypoint detection)에도 일반화될 수 있어 보인다.</mark> 다만 "기여도"를 사전에 어떻게 측정할지(이 논문은 정성적 판단에 의존)는 별도 방법론이 필요.
- P5를 제거하는 대신 경량화(채널 수 축소, depthwise separable conv 등)하는 대안도 가능 — 완전 제거보다 정보 손실이 적을 수 있지만 연산량 절감 폭은 작아지는 트레이드오프 예상.

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">[[Small_Object_Detection_Approaches]] 비교 문서에서 이 논문은 "아키텍처 경량화" 축의 유일한 사례로 분류되어 있다. [[2026_TIP_Unc-SOD|Unc-SOD]](label assignment 축), feature 강화 계열([[2024_ECCV_SR-TOD|SR-TOD]], [[2025_RemoteSensing_FANet|FANet]], [[2025_CVPR_Feature_Info_Driven_Gaussian|Feature_Info_Driven_Gaussian]]) 등은 모두 정확도 개선에 집중하며 파라미터/FLOPs 증가를 감수하는데, 이들의 핵심 모듈(uncertainty branch, self-reconstruction head 등)을 LCOR처럼 "저기여 부분 제거로 상쇄"하는 방식으로 경량화할 수 있는지 검토해볼 가치가 있다.</mark>
- Ablation에서 확인된 SPPFL-LCOR, Dysample-C2f-N 간 synergy 효과(구성 6, 7)는 "어떤 모듈 조합이 상호 보완적인지" 사전에 예측할 수 있는 원리가 있는지 궁금증을 남긴다 — 현재는 실험으로만 확인되었을 뿐 설계 원리로 제시되지 않는다.

> [!info] 내 메모
> 

# 관련 개념
- [[Lightweight_Cross_Layer_Output_Reconstruction]] — 이 논문의 핵심 기여인 LCOR 모듈. 저기여도 고레벨 검출층 제거와 cross-layer connection 추가를 결합해 경량화와 소형 객체 검출력 향상을 동시에 달성하는 기법.
- [[Large_Separable_Kernel_Attention]] — SPPFL 모듈의 attention 메커니즘(LSKA). 큰 커널을 수평·수직 1D depthwise conv로 분해해 저비용으로 넓은 수용영역을 확보. Architecture Design으로 신규 작성.
- [[Normalization_Based_Attention_Module]] — C2f-N 모듈의 attention 메커니즘(NAM). BN scaling factor를 채널·공간 중요도로 재활용하는 초저비용 attention. Architecture Design으로 신규 작성.
- [[Dysample_Dynamic_Upsampling]] — Neck의 경량 동적 업샘플러. Linear layer+pixel shuffle만으로 content-aware 샘플링 위치를 생성. Architecture Design으로 신규 작성.

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]] — 8편 중 "아키텍처 경량화" 축을 정면으로 다루는 유일한 논문으로 분류. 파라미터/FLOPs/FPS를 명시적으로 보고하는 것도 이 논문이 유일함.

# 읽어볼 만한 논문
- 참고문헌 기반: K. W. Lau, L.-M. Po, Y. A. U. Rehman, "Large separable kernel attention: rethinking the large kernel attention design in CNN" (Expert Systems with Applications, 2024) — SPPFL이 채택한 LSKA의 원조 논문. SPPFL의 attention 메커니즘을 제대로 이해하려면 먼저 읽을 필요가 있음.
- 참고문헌 기반: Y. Liu, Z. Shao, Y. Teng, N. Hoffmann, "NAM: Normalization-based attention module" (arXiv:2111.12419, 2021) — C2f-N이 채택한 NAM의 원조 논문. BN scaling factor를 attention으로 재활용하는 아이디어가 왜 저비용인지 배경을 이해하는 데 필요.
- 참고문헌 기반: W. Liu, H. Lu, H. Fu, Z. Cao, "Learning to upsample by learning to sample" (ICCV 2023) — Dysample의 원조 논문. "linear layer + pixel shuffle"만으로 content-aware upsampling을 저비용으로 구현하는 설계를 직접 다룸.
- 참고문헌 기반: X. Zhu, S. Lyu, X. Wang, Q. Zhao, "TPH-YOLOv5: Improved YOLOv5 based on transformer prediction head for object detection on drone-captured scenarios" (ICCV Workshops 2021) — 이 논문이 "정확도는 높이지만 파라미터가 크게 증가하는" 갈래의 대표 사례로 인용한 논문. LSOD-YOLO가 피하고자 한 트레이드오프를 실제로 보여주는 대조군으로 읽을 만함.
- 자유 추천(검증 필요): YOLO 계열 검출 헤드에 structured pruning/channel pruning을 적용한 경량화 연구 — 검색 키워드: `YOLO detection head structured pruning small object real-time`. 이 논문이 Conclusion에서 향후 과제로 남긴 "model pruning/quantization을 통한 추가 경량화"와 직접 연결되는 방향이라, LCOR와 결합 가능성을 검토할 때 참고할 만함.
