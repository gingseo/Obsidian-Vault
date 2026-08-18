---
title: "RS-TOD: Tiny object detection model in Remote Sensing Imagery"
authors: [Rakhi Nautiyal, Maroti Deshmukh]
year: 2025
venue: "Remote Sensing Applications: Society and Environment (Elsevier)"
jcr_quartile: null
task: [small-object-detection]
direction: [improvement]
tags: [paper, small-object-detection, remote-sensing, yolo, attention-module, detection-head]
status: in-progress
added: 2026-07-01
source: "PaperStudy/Raw/Small_Object_Detection/2025_RSASE_RS-TOD.pdf"
created: 2026-08-04
---

#paper #small-object-detection #remote-sensing #yolo #attention-module #detection-head

# 한 줄 요약
<mark style="background: #FFF3A3A6;">YOLOv8n에 channel+spatial attention 모듈 RSAM을 각 detection head 앞에 삽입하고, shallow feature를 활용하는 160×160 해상도의 tiny-object 전용 detection head를 추가해, 원격탐사 영상(SODA-A/AI-TOD/TinyPerson)에서 tiny object detection 성능을 baseline 대비 큰 폭으로 끌어올린 경량 모델.</mark>


# 문제 정의

### 기존 방법의 한계
- **정보 소실**:
  RSI(Remote Sensing Imagery)는 밀집·불규칙 분포한 tiny object가 많고 배경이 장면 대부분을 차지한다. CNN의 연속적 다운샘플링으로 최상위(topmost) feature map에서 객체가 원본 대비 1/16 크기로 축소되어, 이미 몇 픽셀 안 되는 tiny object의 신호가 거의 지워진다.
- **Scale/orientation 다양성**:
  같은 장면에 배와 항구처럼 스케일 차이가 큰 객체가 공존하고, 차량처럼 객체 방향이 랜덤하게 분포한다. 고정 receptive field는 이런 스케일 편차에 대응하지 못한다.
- **복잡한 배경과 클래스 혼동**:
  배경이 장면 대부분을 차지해 객체 영역이 압도(overshadow)되고, 밀집·유사 클래스 객체 간 혼동이 발생한다.
- **YOLOv8 자체의 한계**:
  YOLOv8은 SOTA one-stage detector이지만, 위 요인들 때문에 tiny object에 대해서는 여전히 누락(omission)과 오검출이 남아 있다.

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Attention/feature 강화 기반**
- FFCA-YOLO(Zhang et al., 2024): class imbalance 대응에 초점, RSAM처럼 spatial+channel을 체계적으로 묶지는 않음.
- CSDP-YOLO(Fan et al., 2024): context-aware attention 도입.
- SCDNet·SDSDet(Liu et al., 2024): contextual information과 정교한 학습 전략 결합.
- SESA-Net(Ma et al., 2024): sample assessment 알고리즘으로 작은 객체에 집중.
- SEB-YOLO(Hui et al., 2024b), PLNet-PR(Ni et al., 2024): multi-scale feature fusion·attention을 빈번히 사용하나 전용 head 설계는 없음.
- MFCANet(Jiang et al., 2024), CTAM(Lang et al., 2024): dual-branch·decoupled head·transformer 모듈 결합.
- KCFS-YOLO(Tian et al., 2023), RSI-YOLO(Li et al., 2023b): attention·전용 detection head·anchor box 최적화를 함께 사용.

**갈래 2 — 경량화 및 손실 함수 개선**
- LE-YOLO(Yue et al., 2024), STF-YOLO(Hui et al., 2024a): edge computing 겨냥 경량 설계로 mAP 유지·연산량 감소.
- YOLO-DCTI(Min et al., 2023), MCS-YOLO(Ji et al., 2023): Soft-CIOU, IoU-T 등 loss 개선으로 localization 정교화.

**갭**: <mark style="background: #FFF3A3A6;">대부분 attention/feature 강화 또는 loss 개선 중 한 축에만 집중하며, "얕은 feature map을 tiny object 전용 경로로 분리"와 "channel+spatial attention을 매 head 앞에 체계적으로 배치"를 함께 다루는 연구는 드물다.</mark> RSAM이 직접 비교하는 SE Block(channel만, spatial 무시)과 CBAM(channel+spatial이지만 2-level pooling·skip connection 없음) 모두 원격탐사 특유의 밀집·복잡 배경을 겨냥해 설계되지 않았다.

### 이 논문이 풀고자 하는 문제
1. YOLOv8의 feature representation을 원격탐사 영상의 복잡한 배경·중첩 객체 상황에 맞게 강화하는 attention 모듈을 설계하는 것
2. Shallow feature map을 활용하는 전용 검출 경로를 마련해, 다운샘플링 과정에서 소실되는 tiny object의 디테일을 detection head 단계까지 보존하는 것

# 제안 방법

<mark style="background: #FFF3A3A6;">YOLOv8n을 기반으로 (1) 3-level channel pooling과 skip connection을 갖춘 RSAM을 4개 detection head 각각의 입력 앞에 배치해 feature representation을 강화하고, (2) backbone의 얕은 C2f feature를 neck의 업샘플된 feature와 결합한 160×160 해상도의 tiny-object 전용 detection head를 새로 추가한다.</mark>

### ① 160×160 Tiny-scale Detection Head
- YOLOv8의 기존 3개 head(80×80, 40×40, 20×20)는 모두 backbone 하위 레벨(B4 이후) feature만 사용.
- RS-TOD는 neck의 B16(C2f 이후)에 upsampling을 추가하고, backbone 최상위 해상도 첫 C2f(B3, 160×160×128)와 concat(B17) 후 추가 C2f(B18)로 정제.
- 정제된 feature는 RSAM(B33)을 거쳐 새 head(B34, 160×160)로 입력.
- B18까지 과도한 다운샘플링을 거치지 않아 tiny object 디테일이 보존되며, 저자들은 학습 시 IoU 상승 → 정확한 bbox 생성에 기여한다고 설명.

<mark style="background: #FFF9D6A6;">깊은 feature는 의미 정보는 풍부하나 해상도가 낮고, 얕은 feature는 반대다. 기존 YOLOv8은 가장 얕은 검출 경로도 80×80에서 멈춰 다운샘플링으로 소실된 tiny object 신호를 살릴 방법이 없었는데, 160×160 head는 backbone 최상위 해상도 feature(B3)를 직접 끌어와 이 소실을 구조적으로 우회한다.</mark>

### ② RSAM (Remote Sensing Attention Module)
- 입력 feature map `I(h, w, c)`에 채널 크기 1, c/4, c/2 세 레벨로 AvgPool을 병렬 적용해 pyramid pooled feature 획득.
- 이를 concat 후 1×1 conv로 채널 1로 축소해 attention map 생성, BatchNorm+Sigmoid로 reweight map을 얻어 원본 `I`에 element-wise 곱.
- 4개 detection head 각각의 입력 앞(B27/B29/B31/B33)에 배치.
- SE Block(channel만) · CBAM(channel+spatial, 2-level pooling, skip connection 없음)과 명시적으로 대비되는 설계.

> [!example]- 구현 디테일
> ```
> M(I) = σ( f_1x1[ X1(I); X2(I); X3(I) ] )
> X = AvgPool(I)
> ```
> 손실 함수는 YOLOv8 표준 구성 그대로: Bbox loss(IoU+MSE), Cls loss(cross-entropy/softmax), Obj loss(BCE), Dfl loss(focal 계열, 파라미터 α).
>
> 학습 설정: Epoch 300, batch size 16, optimizer SGD(초기/최종 lr 0.01, momentum 0.937, weight decay 0.0005), Mosaic augmentation, 입력 해상도 640×640 vs 1280×1280 비교(1280이 세 데이터셋 모두 우세).

<mark style="background: #FFF9D6A6;">단순 channel attention(SE Block)만으로는 "어디를 봐야 하는지"를 특정하지 못하고, 2-level pooling(CBAM)만으로는 세밀한 채널 상호작용이 손실된다. RSAM은 3-level pooling으로 촘촘한 채널 정보를 유지하면서 spatial attention까지 적용해 배경 대비 객체 영역에 가중치를 집중시키고, skip connection으로 gradient 흐름을 유지한다 — "배경에 압도되어 객체 영역이 무시된다"는 문제의 원인에 직접 대응한다.</mark>

# 실험 결과

### 핵심 결과

| 벤치마크 | 지표 | Before(YOLOv8) | After(RS-TOD) |
|---|---|---|---|
| SODA-A | mAP50 | 52.81% | 60.10% (+7.29%p) |
| AI-TOD | mAP50 | 48.50% | 59.84% (+11.34%p) |

> [!note]- 세부 결과 및 Ablation
> #### Baseline 대비 개선 전체
> | 벤치마크 | 지표 | Before(YOLOv8) | After(RS-TOD) | 비고 |
> |---|---|---|---|---|
> | SODA-A | mAP50 | 52.81% | 60.10% | +7.29%p (baseline: YOLOv8n) |
> | SODA-A | mAP50-95 | 14.30% | 20.30% | +6.00%p |
> | AI-TOD | mAP50 | 48.50% | 59.84% | +11.34%p (baseline: YOLOv8s) |
> | AI-TOD | mAP50-95 | 22.37% | 28.40% | +6.03%p |
> | TinyPerson | mAP50 | 40.52% | 47.60% | +7.08%p (baseline: YOLOv8s) |
> | TinyPerson | mAP50-95 | 13.40% | 16.80% | +3.40%p |
>
> 설정: SODA-A(2,513장, 6class, 872,069 instance), AI-TOD(28,036장, 8class, 700,621 instance), TinyPerson(1,610장, 1class person, 72,651 instance). 비교 대상은 YOLOv5n/v6n/NAS/v9t/v10n/11n, baseline YOLOv8(n 또는 s), 데이터셋별 최신 문헌(TBNet, EL-YOLO, DetectoRS w/ SR-TOD, Faster RCNN-FPN 등). 아래 수치는 별도 표기 없으면 1280×1280 기준.
>
> #### SODA-A 클래스별 성능 (mAP50, Table 6)
> | 클래스 | YOLOv8n baseline | RS-TOD | 비고 |
> |---|---|---|---|
> | Small Vehicle | 20.67 | **49.20** | |
> | Large Vehicle | **42.69** | 39.50 | 유일하게 baseline이 우세 (mAP50-95도 30.16 vs 25.84) |
> | Ship | 20.40 | **42.70** | |
> | Storage Tank | 35.21 | **48.80** | |
> | Container | 29.30 | **69.52** | |
> | Swimming Pool | 72.30 | **77.40** | |
> | ALL | 52.81 | **60.10** | |
>
> #### AI-TOD 클래스별 성능 (mAP50, Table 7)
> | 클래스 | YOLOv8s baseline | RS-TOD | 비고 |
> |---|---|---|---|
> | Airplane | 24.20 | **30.00** | |
> | Bridge | 27.68 | **32.00** | |
> | Storage Tank | 39.00 | **41.34** | |
> | Ship | 40.84 | **58.96** | |
> | Swimming Pool | 38.13 | **42.00** | |
> | Vehicle | **27.66** | 26.70 | baseline 근소 우세 — 형태/스케일 다양성, Storage Tank와 class 유사성 |
> | Person | 11.27 | **14.18** | |
> | Windmill | 9.60 | 13.40 | YOLO-NAS가 최고(16.34) — 극심한 class imbalance(train 176 / test 67) |
> | ALL | 48.50 | **59.84** | |
>
> #### 기존 문헌 대비 비교
> | 벤치마크 | 비교 모델 | 지표 | 비교 모델 | RS-TOD | 비고 |
> |---|---|---|---|---|---|
> | AI-TOD | TBNet [Li et al. 2025] | mAP50 | 59.00 | **59.84** | +0.84%p |
> | AI-TOD | EL-YOLO-s [Xue et al. 2024] | mAP50-95 | 26.30 | **28.40** | +2.10%p |
> | AI-TOD | DetectoRS w/ SR-TOD [Cao et al. 2025] | mAP50 / mAP50-95 | 54.60 / 24.00 | **59.84 / 28.40** | RS-TOD가 mAP50-95에서도 상회 |
> | TinyPerson | Faster RCNN-FPN [Lin et al. 2017] | mAP50 | 43.55 | **47.60** | +4.05%p |
> | TinyPerson | Faster RCNN-FPN | mAP50-95 | 5.35 | **16.80** | +11.45%p (원문 표기상 "TOD-YOLOv7 대비 +7.30%p" 서술도 있으나 Table 11에 TOD-YOLOv7 행은 없어 Faster RCNN-FPN 대비 수치로 반영) |
>
> #### 해상도 비교 (Table 12)
> | 데이터셋 | 지표 | 640×640 | 1280×1280 |
> |---|---|---|---|
> | SODA-A | mAP50 | 53.78 | **60.10** |
> | SODA-A | mAP50-95 | 15.80 | **20.30** |
> | AI-TOD | mAP50 | 50.24 | **59.84** |
> | AI-TOD | mAP50-95 | 23.74 | **28.40** |
> | TinyPerson | mAP50 | 41.89 | **47.60** |
> | TinyPerson | mAP50-95 | 13.96 | **16.80** |
>
> #### 연산 비용 (Table 13)
> | 모델 | #Param(M) | FPS | GFLOPs(B) | 모델 크기(MB) |
> |---|---|---|---|---|
> | Baseline YOLOv8 | 3.81 | **60** | 8.7 | **12.2** |
> | RS-TOD | **2.93** | 45 | **7.8** | 12.9 |
>
> 파라미터·GFLOPs는 감소했지만(레이어 수 225→168) FPS는 60→45로 25% 하락, 모델 크기는 0.7MB 증가 — attention 모듈이 경량(1×1 conv 위주)이라 파라미터 총량은 줄었으나, FPS 하락은 head 개수 증가로 인한 추론 단계 연산 분기 증가로 추정(논문이 이 괴리를 직접 설명하지는 않음).

# Discussion

### 이 아이디어의 잠재적 부작용
- 추론 경로가 길어질 위험(head 4개+RSAM 4개 추가) → 실측 결과 파라미터·GFLOPs는 오히려 감소했지만 <mark style="background: #FF5582A6;">FPS는 60→45로 25% 하락, 논문은 GFLOPs 감소와 FPS 하락이 동시 발생하는 이유를 설명하지 않는다.</mark>
- RSAM이 원격탐사 특유 조건(복잡 배경, 중첩 객체)을 겨냥해 설계됨 → <mark style="background: #FF5582A6;">원격탐사 세 데이터셋에만 검증, COCO 등 일반 자연 영상 일반화 실험은 없음. 저자도 "다른 attention과 결합해 일반화 개선 가능"이라 언급해 한계를 사실상 인정.</mark>

### 한계
- <mark style="background: #FF5582A6;">AI-TOD Windmill 클래스(train 176/test 67, 극심한 class imbalance)에서 RS-TOD(13.40)가 YOLO-NAS(16.34)보다 낮음</mark> — oversampling으로 개선 여지 언급만 하고 실제 적용 안 함.
- <mark style="background: #FF5582A6;">AI-TOD Vehicle, SODA-A Large Vehicle에서 baseline이 근소 우세</mark> — 형태/스케일 다양성과 유사 클래스 혼동이 원인으로 지목되나 RSAM/추가 head가 이를 해결하도록 설계되지는 않음.
- <mark style="background: #FF5582A6;">저자 명시 한계: 성능에 가장 큰 영향을 미치는 요인은 데이터셋 자체(class imbalance, 유사성, 라벨링 품질, 중첩, 노이즈)이며 전처리로 일부 완화 가능하나 근본 해결은 안 됨.</mark>
- RSAM의 3-level pooling에 대한 ablation(레벨 수 변화 비교)이 논문에 없음 — Discussion에서 제안만 하고 실험 근거는 없음.

### 생각할 점
- <mark style="background: #A6E3A1A6;">RSAM의 attention map 계산 자체(3-level AvgPool + 1×1 conv + Sigmoid)는 도메인 특정적이지 않다 — "복잡한 배경·밀집 객체" 조건은 드론뷰·감시 카메라·의료 영상에도 있어, 원격탐사 특화인지 일반적 attention 개선인지는 다른 도메인 실험 없이 판단하기 어렵다.</mark>
- 160×160 head가 backbone 얕은 feature(B3)를 그대로 끌어오는 방식은 [[SR-TOD]]가 difference map으로 "정보 손실 큰 영역"을 간접적으로 찾는 방식과 대조적 — RS-TOD는 위치를 특정하지 않고 얕은 해상도 경로를 통째로 추가하는 반면, SR-TOD는 reconstruction 난이도 신호로 위치를 특정한 뒤 P2에 강화를 가한다. 결합 가능성 있음.

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">RS-TOD의 RSAM(feature 강화)과 [[Unc-SOD]]의 label assignment 축(instance-level uncertainty 기반 동적 sampling)은 서로 다른 지점에 개입하므로 직교적 — RSAM으로 강화된 feature 위에서 uncertainty 기반 sampling을 적용하면 개선이 누적될 가능성([[Small_Object_Detection_Approaches]]에서도 두 논문이 다른 축으로 분류).</mark>
- <mark style="background: #A6E3A1A6;">RS-TOD Table 8에서 직접 비교되는 DetectoRS w/ SR-TOD([[SR-TOD]])는 mAP50-95(24.00)에서 RS-TOD(28.40)에 뒤짐 — 같은 AI-TOD 벤치마크에서 이미 간접 비교되고 있으므로, difference map 기반 강화와 RSAM+전용 head를 같은 백본에서 직접 결합하는 실험이 자연스러운 다음 단계.</mark>

# 관련 개념
- [[Remote_Sensing_Attention_Module]] — 이 논문이 새로 제안하는 channel+spatial attention 모듈(RSAM). 각 detection head 앞에 배치되어 feature representation을 강화하는 핵심 기여.

# 관련 문서
- 주의(혼동 방지): 이름이 유사한 [[SR-TOD]](difference-map 기반 tiny object detection, 2024 ECCV)와는 저자·방법론이 전혀 무관한 별개 논문이다 — RS-TOD는 YOLOv8 기반 attention+head 확장, SR-TOD는 self-reconstruction difference map 기반 feature 강화. 다만 RS-TOD의 Table 8에서 "DetectoRS w/ SR-TOD"가 AI-TOD 비교 대상 중 하나로 실제로 인용되므로, 두 논문은 이름만 비슷한 게 아니라 같은 벤치마크(AI-TOD)에서 실제로 비교되는 관계이기도 하다.
- 비교: [[Small_Object_Detection_Approaches]] — feature 강화(attention) + 헤드 추가 축, 원격탐사 특화 계열로 분류

# 읽어볼 만한 논문
- 참고문헌 기반: Z. Li, Y. Wang, D. Xu, Y. Gao, T. Zhao, "TBNet: A texture and boundary-aware network for small weak object detection in remote-sensing imagery" (Pattern Recognition, 2025) — RS-TOD가 AI-TOD에서 mAP50 기준 직접 비교하는 가장 근접한 경쟁 모델(59.00 vs 59.84). texture/boundary를 명시적으로 다루는 접근이라 RSAM의 spatial attention과 대비하며 읽을 만하다.
- 참고문헌 기반: B. Cao, H. Yao, P. Zhu, Q. Hu, "Visible and Clear: Finding Tiny Objects in Difference Map" (ECCV, 2025 — 논문 내 인용 표기는 Cao et al., 2025) — 이미 위키에 [[SR-TOD]]로 등록된 논문. RS-TOD의 Table 8에서 AI-TOD 비교 대상("DetectoRS w/ SR-TOD")으로 직접 인용되어, RS-TOD와 실측 비교가 가능한 가장 가까운 대안적 feature 강화 접근.
- 참고문헌 기반: C. Xu, J. Wang, W. Yang, H. Yu, L. Yu, G.-S. Xia, "Detecting tiny objects in aerial images: A normalized Wasserstein distance and a new benchmark" (ISPRS J. Photogramm. Remote Sens., 2022) — RS-TOD가 Table 8에서 "RetinaNet w/ NWD-RKA"로 비교하는 label-assignment 계열 대표 논문. RSAM 같은 feature 강화 축과는 다른 축(assignment metric)에서 tiny object 문제를 다뤄, [[Unc-SOD]] 노트에서 이미 추천된 RFLA와 함께 label-assignment 계열 배경 이해에 도움.
- 자유 추천(검증 필요): CBAM(Convolutional Block Attention Module)의 원 논문 — 검색 키워드: `CBAM convolutional block attention module ECCV 2018`. RS-TOD 본문이 RSAM을 SE Block, CBAM과 직접 비교하며 설계 근거로 삼고 있어, RSAM의 3-level pooling·skip connection이 실제로 CBAM 대비 어떤 구조적 차이인지 원 논문으로 확인할 가치가 있다.
