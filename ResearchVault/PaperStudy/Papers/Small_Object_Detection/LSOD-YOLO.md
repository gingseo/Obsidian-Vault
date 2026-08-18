---
title: "Precision and speed: LSOD-YOLO for lightweight small object detection"
authors: [Hezheng Wang, Jiahui Liu, Jian Zhao, Jianzhong Zhang, Dong Zhao]
year: 2025
venue: "Expert Systems With Applications"
jcr_quartile: null
task: [small-object-detection]
direction: [improvement]
tags: [paper, small-object-detection, yolo, lightweight, attention-mechanism, uav]
status: in-progress
added: 2026-07-01
source: "PaperStudy/Raw/Small_Object_Detection/2025_ESWA_LSOD-YOLO.pdf"
created: 2026-08-04
---

#paper #small-object-detection #yolo #lightweight #attention-mechanism #uav

# 한 줄 요약
<mark style="background: #FFF3A3A6;">YOLOv8s의 P5 검출 헤드를 제거하고 대신 P2 헤드를 cross-layer connection과 함께 추가하는 LCOR을 중심으로, SPPFL(LSKA)·C2f-N(NAM)·Dysample을 결합해 VisDrone2019에서 파라미터를 65.5% 줄이면서도 mAP0.5를 오히려 끌어올린 경량 소형 객체 검출 모델.</mark>


# 문제 정의

### 기존 방법의 한계
- **저해상도 feature의 위치 정보 손실**:
  소형 객체는 픽셀 표현이 적고 밀집 분포·복잡한 배경 속에 있어, 다운샘플링을 거친 저해상도 feature map(P4/P5)에서는 각 픽셀이 넓은 영역을 대표하게 되어 위치 정보 손실이 크고 미검출·오검출이 잦다.
- **정확도 개선 기법들의 연산량 증가**:
  TPH-YOLOv5(Transformer prediction head), Ji et al.의 multi-scale context + Soft-CIOU, FE-YOLOv5(deformable conv) 등은 소형 객체 검출 정확도를 높이지만 대부분 파라미터 수가 크게 늘어난다.
- **경량화 기법들의 정확도 손실**:
  YOLO-S(Betti & Tucci, 2023)처럼 Darknet20 기반 경량 backbone을 쓰면 속도는 빨라지지만 backbone의 feature 추출 능력이 제한되어 정확도가 떨어진다.
- **P2 헤드 단순 추가의 트레이드오프**:
  소형 객체 검출력을 높이기 위해 Neck에 P2 layer(160×160)를 추가하고 검출 헤드를 하나 더 붙이는 방법(YOLOv8s-P2류)이 일반적으로 쓰이지만, 검출 헤드가 3개→4개로 늘어나면서 연산 복잡도(GFLOPs)가 크게 증가한다.

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — 정확도 중심의 개선(경량성 희생)**
- TPH-YOLOv5(Zhu et al., 2021): Transformer prediction head로 전역 문맥 강화 — 파라미터 증가.
- Ji et al. (2023): multi-scale contextual information + Soft-CIOU loss — 파라미터 증가.
- FE-YOLOv5(Wang et al., 2023): deformable convolution으로 수용영역 적응적 확장 — 파라미터 증가.
- FFNB(Wang et al., 2023, YOLOv8 기반): 얕은/깊은 feature 병합 — 파라미터 증가.
- STC-YOLO(Lai et al., 2023), SO-YOLOv5(Xuan et al., 2023, coordinate attention), Wu et al.(2023, residual coordinate attention): 같은 갈래, 같은 한계.
- 공통 한계: 정확도는 개선되지만 대부분 파라미터·연산량이 크게 증가해 실시간/자원 제약 환경에 부적합.

**갈래 2 — 경량화 중심의 backbone 설계**
- YOLO-S(Betti & Tucci, 2023): Darknet20 기반 경량 feature extraction network.
- 공통 한계: 검출 속도는 개선되지만 backbone의 feature 추출 용량이 제한되어 정확도가 떨어짐.

**갭**: <mark style="background: #FFF3A3A6;">두 갈래가 각각 정확도 또는 경량성 한쪽만 정면으로 다뤄왔고, "P2 고해상도 feature를 살리면서 동시에 연산량은 늘리지 않는" 설계는 시도되지 않았다.</mark> 이 논문은 P2 헤드를 단순히 더하는 대신, 기여도가 낮은 P5 헤드를 제거해 얻은 여유를 P2 헤드에 재배분하는 방식으로 이 갭을 메운다.

### 이 논문이 풀고자 하는 문제
1. 소형 객체 검출에 유리한 고해상도 P2 feature를 검출에 활용하되, 헤드 추가로 인한 연산량 증가를 상쇄할 방법을 찾는 것
2. 정확도(mAP0.5)와 경량성(Params/Model size/GFLOPs/FPS)을 동시에 만족하는 균형점을 찾는 것 — 즉 위 두 갈래 중 하나를 포기하지 않는 것

# 제안 방법

<mark style="background: #FFF3A3A6;">핵심 아이디어: 저기여도 고레벨 검출 헤드(P5)를 제거해 확보한 여유를 고해상도 저레벨 검출 헤드(P2) 추가에 재배분하고, 둘 사이에 cross-layer connection을 둬 정보 손실을 보완한다(LCOR) — 여기에 attention 기반 SPP(SPPFL), attention 기반 C2f(C2f-N), 경량 upsampler(Dysample)를 결합해 이 재배분으로 인한 부수적 손실을 추가로 보완한다.</mark>

### ① LCOR (Lightweight Cross-layer output Reconstruction)
- YOLOv8 backbone은 P1~P5 5단계 다운샘플링을 거쳐 80×80/40×40/20×20 세 검출 헤드를 사용(Fig. 2a). 저해상도 feature map은 픽셀당 대표 영역이 넓어 위치 정밀도가 낮은 반면, 고해상도 P2(160×160)는 edge·shape 표현에 유리.
- 기존 방법(YOLOv8s-P2)은 P2에 헤드를 그대로 추가(Fig. 2b) — 헤드가 3→4개로 늘어 연산량 증가.
- LCOR는 대신 (a) semantic 정보 위주지만 소형 객체 민감도가 낮은 **P5 검출층을 제거**, (b) **P2 검출 헤드를 추가**, (c) 얕은 layer(P2/P3)와 깊은 layer 사이에 **cross-layer connection**(skip connection)을 둬 두 정보를 함께 보존(Fig. 2c).
- 결과적으로 최종 검출 헤드는 160×160(P2)과 80×80(P3) 두 스케일만 사용.

<mark style="background: #FFF9D6A6;">P5 제거와 P2 추가는 "헤드 하나를 빼고 하나를 더하는" 상쇄 구조라 P2 단순 추가 방식이 동반하던 연산량 증가를 원천적으로 없애고, P5가 담당하던 semantic 정보 손실은 cross-layer connection으로 P3 정보를 P2에 전달해 보완한다 — 두 목표(정확도 유지·경량화)가 같은 설계 변경 안에서 함께 달성된다.</mark>

### ② SPPFL (SPPF + Large Separable Kernel Attention)
- 기존 SPPF는 3회 연속 maxpool + concat으로 multi-scale feature 통합(Fig. 3a).
- LCOR로 계층이 단순화되며 고수준 semantic 정보가 줄어들 수 있다는 우려에 대응해, SPPF 출력에 LSKA(Lau et al., 2024)를 결합한 SPPFL 제안(Fig. 3b).
- LSKA: 수평/수직 1D depthwise conv + depthwise-dilated conv로 넓은 receptive field를 확보한 뒤, 1×1 conv로 attention map 생성, 원본 feature와 Hadamard product.

> [!example]- 구현 디테일
> ```
> Z̄C = Σ W(2d-1)×1 * (Σ W1×(2d-1) * FC)      … 수평/수직 depthwise conv
> ZC = Σ W(k/d)×1 * (Σ W1×(k/d) * Z̄C)         … depthwise-dilated conv
> AC = W1×1 * ZC                              … attention map
> F̄C = AC ⊗ FC                                … Hadamard product로 feature 정제
> ```

<mark style="background: #FFF9D6A6;">LCOR로 단순화되며 손실될 수 있는 semantic/global 정보를, 헤드나 레이어를 추가하지 않고도 attention 가중치 재분배만으로 보완한다.</mark>

### ③ C2f-N (C2f + NAM)
- NAM(Normalization-based Attention Module, Liu et al., 2021)은 CBAM 구조에서 착안하되, BN의 scaling factor를 그대로 channel/spatial 중요도로 활용.
- Channel attention은 BN scaling factor γ, spatial attention은 pixel normalization scaling factor λ로 가중치 산출, sparsity 정규화 항으로 비중요 feature 억제.
- C2f-N은 C2f 모듈의 residual branch 출력에 NAM을 삽입한 구조(Fig. 6), Neck의 저해상도 feature map에 적용.

> [!example]- 구현 디테일
> Channel attention: BN scaling factor γ 정규화(Eq. 5~6). Spatial attention: pixel normalization scaling factor λ(Eq. 7). 손실 함수에 γ, λ sparsity 정규화 항 추가(Eq. 8). sparsity 계수 p는 원 논문(Liu et al., 2021) 설정을 그대로 따름 — 이 논문에서 별도 튜닝값 명시 없음.

<mark style="background: #FFF9D6A6;">이미 계산되어 있는 BN scaling factor를 재사용해 attention을 만들기 때문에 추가 파라미터·연산 비용이 거의 없다(Table 8: 3.755M→3.755M로 사실상 불변). 복잡한 배경에서 비중요 feature를 억제해 별도의 무거운 attention 모듈 없이도 정확도를 끌어올려 "정확도 개선을 경량성 손실 없이" 달성한다.</mark>

### ④ Dysample
- 기존 nearest-neighbor upsampling은 픽셀 공간 위치만으로 kernel을 결정해 content 정보를 활용 못하고, 소형 객체에서 pixel distortion·정보 손실 유발.
- Dysample(Liu et al., 2023)은 CARAFE류와 달리 dynamic convolution·별도 sub-network 없이 "linear layer + pixel shuffle"만으로 샘플링 위치 offset을 동적 생성(Fig. 7).

> [!example]- 구현 디테일
> c×h×w feature map → linear layer(출력 채널 2s²) → pixel shuffle로 2×sh×sw offset 생성 → 원본 sampling grid에 더해 c×sh×sw upsampled feature map 생성.
>
> 학습 설정: 입력 640×640, 300 epoch, batch size 16, 초기 학습률 0.01, momentum 0.937, weight decay 0.0005, optimizer SGD. Loss는 YOLOv8 기본(CIoU + DFL + BCE) 그대로 사용, 별도 변경 없음.

<mark style="background: #FFF9D6A6;">동적 sub-network나 dynamic convolution을 쓰는 CARAFE 대비 훨씬 적은 연산으로 content-aware upsampling 효과를 얻어, upsampling 품질(정확도)과 연산량(경량성) 사이의 트레이드오프 자체를 개선한다.</mark>

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
> - **플랫폼**: RTX 4090, PyTorch 2.2.1, CUDA 12.1.1
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
> #### Ablation — LCOR (Table 6)
> | 모델 | mAP0.5(%) | Params(M) | GFLOPs |
> |---|---|---|---|
> | YOLOv8s-P2 | 36.9 | 10.9 | 39.7 |
> | LSOD-YOLO(cross-layer connection 제외) | 36.5 | 3.7 | 33.6 |
> | LSOD-YOLO | 37.0 | 3.8 | 33.9 |
>
> cross-layer connection 하나만 추가해도 GFLOPs +0.3 대비 mAP0.5 +0.5%p — 연산 비용 대비 이득이 큼.
>
> #### Ablation — Dysample (Table 7)
> | 방식 | mAP0.5(%) | Params(M) | FPS | GFLOPs |
> |---|---|---|---|---|
> | nearest-neighbor | 36.7 | 3.8 | 90 | 33.8 |
> | CARAFE | 36.9 | 3.9 | 83 | 35.8 |
> | Dysample | 37.0 | 3.8 | 93 | 33.9 |
>
> #### Ablation — C2f attention 모듈 비교 (Table 8)
> | 모듈 | mAP0.5(%) | Params(M) |
> |---|---|---|
> | attention 없음 | 36.5 | 3.755 |
> | C2f-CBAM | 36.6 | 3.762 |
> | C2f-CA | 36.7 | 3.762 |
> | C2f-SA | 36.7 | 3.756 |
> | C2f-SE | 36.9 | 3.758 |
> | C2f-N | 37.0 | 3.755 |
>
> C2f-N이 최소 파라미터로 최고 mAP0.5·R 달성 (P는 C2f-CA가 0.1%p 더 높음).
>
> #### 전체 Ablation (Table 9, ✓=포함 −=제외)
> | 구성 | LCOR | SPPFL | Dysample | C2f-N | mAP0.5(%) | Params(M) |
> |---|---|---|---|---|---|---|
> | LSOD-YOLO(전체) | ✓ | ✓ | ✓ | ✓ | 37.0 | 3.8 |
> | 1: LCOR 제외 | − | ✓ | ✓ | ✓ | 35.8 | 12.3 |
> | 2: SPPFL 제외 | ✓ | − | ✓ | ✓ | 36.2 | 3.5 |
> | 3: Dysample 제외 | ✓ | ✓ | − | ✓ | 36.7 | 3.7 |
> | 4: C2f-N 제외 | ✓ | ✓ | ✓ | − | 36.6 | 3.8 |
> | 5: LCOR+SPPFL 제외 | − | − | ✓ | ✓ | 36.0 | 11.2 |
> | 6: LCOR+Dysample 제외 | − | ✓ | − | ✓ | 35.8 | 12.2 |
> | 7: Dysample+C2f-N 제외 | ✓ | ✓ | − | − | 35.6 | 12.3 |
> | YOLOv8s(baseline) | − | − | − | − | 34.5 | 11.2 |
>
> - **LCOR가 경량화의 절대적 핵심**: 제외 시 파라미터 3.8M→12.3M로 3배 이상 급증, mAP0.5도 37.0%→35.8%로 가장 크게 하락.
> - **SPPFL 제외는 파라미터엔 거의 영향 없이(3.5M) mAP0.5만 36.2%로 하락** — 순수하게 정확도 보완 역할.
> - **LCOR+Dysample 동시 제외(구성 6)**는 LCOR 단독 제외(구성 1)와 거의 같은 결과(35.8% vs 35.8%) — LCOR의 영향력이 압도적.
> - **Dysample+C2f-N 동시 제외(구성 7)**는 mAP0.5 35.6%로, 각각 단독 제외(36.7%, 36.6%)보다 더 크게 하락 — 두 모듈 간 synergy 효과.
>
> #### 일반화 실험 (Table 10)
> | 데이터셋 | 모델 | mAP0.5(%) | mAP0.5:0.95(%) |
> |---|---|---|---|
> | TinyPerson | YOLOv8s | 26.1 | 11.7 |
> | TinyPerson | YOLOv8s-P2 | 34.3 | 14.9 |
> | TinyPerson | LSOD-YOLO | 35.2 | 15.6 |
> | LEVIR-Ship | YOLOv8s | 78.3 | 30.6 |
> | LEVIR-Ship | YOLOv8s-P2 | 76.9 | 29.9 |
> | LEVIR-Ship | LSOD-YOLO | 79.5 | 31.3 |
> | UAVDT | YOLOv8s | 35.2 | 21.9 |
> | UAVDT | YOLOv8s-P2 | 33.6 | 20.3 |
> | UAVDT | LSOD-YOLO | 37.1 | 22.1 |
>
> TinyPerson·LEVIR-Ship·UAVDT 세 데이터셋 모두에서 LSOD-YOLO가 YOLOv8s, YOLOv8s-P2, YOLOv5s/m, YOLOv7-tiny를 상회 — 드론뷰(VisDrone) 특화 설계가 사람 탐지·선박 탐지·다른 드론뷰에도 일관되게 전이됨을 시사.

# Discussion

### 이 아이디어의 잠재적 부작용
- **P5 제거가 대형 객체 검출을 희생시킬 가능성**:
  P5는 저해상도지만 수용영역이 가장 넓어 일반적으로 대형 객체에 유리한 레벨. <mark style="background: #FF5582A6;">논문은 이를 직접 검증하지 않는다 — Table 3/5에서 "Bus"(대형 객체 비중 80.10%)가 55.7%→59.7%로 개선되긴 하나 P2 추가 등 다른 요인과 뒤섞여 P5 제거만의 순수 영향을 분리하지 않으며, Table 9 ablation도 크기별 breakdown이 없다.</mark>
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
- <mark style="background: #A6E3A1A6;">[[Small_Object_Detection_Approaches]] 비교 문서에서 이 논문은 "아키텍처 경량화" 축의 유일한 사례로 분류되어 있다. [[Unc-SOD]](label assignment 축), feature 강화 계열([[SR-TOD]], [[FANet]], [[Feature_Info_Driven_Gaussian]]) 등은 모두 정확도 개선에 집중하며 파라미터/FLOPs 증가를 감수하는데, 이들의 핵심 모듈(uncertainty branch, self-reconstruction head 등)을 LCOR처럼 "저기여 부분 제거로 상쇄"하는 방식으로 경량화할 수 있는지 검토해볼 가치가 있다.</mark>
- Ablation에서 확인된 SPPFL-LCOR, Dysample-C2f-N 간 synergy 효과(구성 6, 7)는 "어떤 모듈 조합이 상호 보완적인지" 사전에 예측할 수 있는 원리가 있는지 궁금증을 남긴다 — 현재는 실험으로만 확인되었을 뿐 설계 원리로 제시되지 않는다.

# 관련 개념
- [[Lightweight_Cross_Layer_Output_Reconstruction]] — 이 논문의 핵심 기여인 LCOR 모듈. 저기여도 고레벨 검출층 제거와 cross-layer connection 추가를 결합해 경량화와 소형 객체 검출력 향상을 동시에 달성하는 기법. 이 위키에서 유일하게 재사용 사례가 아직 없는(이 논문 1편에서만 등장) concept.

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]] — 8편 중 "아키텍처 경량화" 축을 정면으로 다루는 유일한 논문으로 분류. 파라미터/FLOPs/FPS를 명시적으로 보고하는 것도 이 논문이 유일함.

# 읽어볼 만한 논문
- 참고문헌 기반: K. W. Lau, L.-M. Po, Y. A. U. Rehman, "Large separable kernel attention: rethinking the large kernel attention design in CNN" (Expert Systems with Applications, 2024) — SPPFL이 채택한 LSKA의 원조 논문. SPPFL의 attention 메커니즘을 제대로 이해하려면 먼저 읽을 필요가 있음.
- 참고문헌 기반: Y. Liu, Z. Shao, Y. Teng, N. Hoffmann, "NAM: Normalization-based attention module" (arXiv:2111.12419, 2021) — C2f-N이 채택한 NAM의 원조 논문. BN scaling factor를 attention으로 재활용하는 아이디어가 왜 저비용인지 배경을 이해하는 데 필요.
- 참고문헌 기반: W. Liu, H. Lu, H. Fu, Z. Cao, "Learning to upsample by learning to sample" (ICCV 2023) — Dysample의 원조 논문. "linear layer + pixel shuffle"만으로 content-aware upsampling을 저비용으로 구현하는 설계를 직접 다룸.
- 참고문헌 기반: X. Zhu, S. Lyu, X. Wang, Q. Zhao, "TPH-YOLOv5: Improved YOLOv5 based on transformer prediction head for object detection on drone-captured scenarios" (ICCV Workshops 2021) — 이 논문이 "정확도는 높이지만 파라미터가 크게 증가하는" 갈래의 대표 사례로 인용한 논문. LSOD-YOLO가 피하고자 한 트레이드오프를 실제로 보여주는 대조군으로 읽을 만함.
- 자유 추천(검증 필요): YOLO 계열 검출 헤드에 structured pruning/channel pruning을 적용한 경량화 연구 — 검색 키워드: `YOLO detection head structured pruning small object real-time`. 이 논문이 Conclusion에서 향후 과제로 남긴 "model pruning/quantization을 통한 추가 경량화"와 직접 연결되는 방향이라, LCOR와 결합 가능성을 검토할 때 참고할 만함.
