---
pm-task: true
projectId: "paperwiki-reading-unified"
parentId:
id: "t-fanet-aufqv4u9nn"
title: "FANet: Frequency-Aware Attention-Based Tiny-Object Detection in Remote Sensing Images"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-07-01"
due:
progress: 0
assignees: []
tags: []
subtaskIds: []
dependencies: []
year: 2025
venue: "Remote Sensing (MDPI)"
jcr_quartile: null
task: [small-object-detection]
direction: [improvement]
paper_tags: [paper, small-object-detection, remote-sensing, frequency-domain, attention, two-stage-detector, class-imbalance]
source: "Projects/논문 읽기_pdf/Small_Object_Detection/2025_RemoteSensing_FANet.pdf"
createdAt: "2026-08-18T11:00:00.000Z"
updatedAt: "2026-08-18T11:00:00.000Z"
---

Project: [[논문 읽기|논문 읽기]]
#paper #small-object-detection #remote-sensing #frequency-domain #attention #two-stage-detector #class-imbalance

# 한 줄 요약
<mark style="background: #FFF3A3A6;">Faster R-CNN 기반 tiny-object detector의 FPN(P2)과 RoI head에 각각 주파수 영역(2D-DFT/2D-DCT) 기반 plug-and-play attention 모듈(MSFFEM, CAREM)을 추가해 spatial 특징의 약점을 보완하고, few-shot 카테고리 불균형은 다중 방향 flip 증강(SAS)으로 완화해 AI-TOD에서 24.8% AP를 달성한 원격탐사 tiny-object detection 논문.</mark>


# 문제 정의

### 기존 방법의 한계
- **약한 spatial 특징**:
  원격탐사 tiny object(16×16px 미만)는 픽셀 수가 극히 적어 저대비·저해상도이고, bounding box regression의 작은 오차도 IoU를 크게 흔들 만큼 위치 오차에 민감하다.
- **큰 intra-class variation**:
  조명·구름·촬영 고도·각도에 따라 같은 카테고리라도 외형이 크게 달라진다(예: AI-TOD의 선박이 촬영 조건에 따라 확연히 다르게 보임, Figure 1).
- **심한 class imbalance**:
  카테고리 간 인스턴스 수 편차가 극단적이다 — AI-TOD trainval에서 vehicle이 88.22%를 차지하는 반면 windmill·swimming pool은 각각 0.1% 미만이라, few-shot 카테고리의 특징이 실제로 학습되는지조차 의심스럽다.
- **범용 검출기·공간 특징 강화 기법의 한계**:
  Faster R-CNN, SSD, 초기 YOLO 등은 tiny object 전용 설계가 아니다. FPN, PANet [16], DetectoRS [17], BAFNet [18], FSANet [19] 같은 공간 특징 강화 기법이나 context modeling 기반 기법(MENet [25], CFENet [26], FFCA-YOLO [27], DQ-DETR [28])도 여전히 spatial-domain 정보에만 의존하므로, tiny object의 근본적으로 약한 feature 문제 자체는 해결하지 못한다.

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Spatial-domain feature 강화 (§2.1)**
- 멀티스케일 융합: PANet [16], DetectoRS [17], BAFNet [18], FSANet [19] — 타일 분할 전처리 자체가 tiny-object 특징을 손상시키는 문제는 대부분 간과.
- Context/attention 기반: non-local [22], DETR 계열 [23,24], MENet [25], CFENet [26], FFCA-YOLO [27], DQ-DETR [28] — 여전히 spatial-domain feature 의존, 내재적으로 약한 특징 자체는 그대로.
- Sample imbalance/few-shot: super-resolution, GAN 합성 [32], SVDDD [33](diffusion), UniFusOD [34](멀티모달) — 연산 복잡도 증가, 일반화 부족.
- Label assignment 개선: NWD [35], RFLA [36] — positive sample 품질·recall 개선하나 "특징 자체가 약하다"는 근본 문제는 미해결.

**갈래 2 — Frequency-domain feature 강화 (§2.2)**
- DFT/wavelet 고전 접근 [37]: 고주파가 경계·디테일과 밀접함을 밝힘.
- CNN+Fourier 결합 [38–41]: camouflaged object detection, segmentation 등에 적용됐으나 범용 목적, tiny object 특화 아님.
- 학습형 주파수 필터/spectral attention: Zhu et al. [42], SpectFormer [43], HS-FPN [44] — 방향은 유사하나 원격탐사 미특화, feature map·RoI 레벨을 동시에 다루지 않음.

**갭**: <mark style="background: #FFF3A3A6;">spatial-domain 갈래는 "특징이 약하다"는 근본 원인을 건드리지 못했고, frequency-domain 갈래는 범용 목적이거나 feature map/RoI 중 한 지점에만 적용되어, 원격탐사 tiny object에 특화된 multi-scale·multi-stage 주파수 강화를 시도한 연구가 없었다.</mark> 이 논문은 feature map 레벨(MSFFEM)과 RoI 레벨(CAREM)에 동시에 주파수 강화를 적용하고, 카테고리별 주파수 특성 차이를 정량 분석해 sample imbalance 대응(SAS)까지 하나의 프레임워크로 묶는다.

### 이 논문이 풀고자 하는 문제
1. Spatial 특징만으로는 표현이 부족한 tiny object의 contour/texture를 주파수 영역 정보로 보완해 배경 노이즈를 억제하고 판별력을 높인다.
2. RoI 단위에서도 고주파 응답을 활용해 위치 추정·분류 정확도를 추가로 개선한다.
3. 카테고리 간 극심한 샘플 불균형(few-shot 카테고리)으로 인한 검출 성능 저하를 완화한다.

# 제안 방법

<mark style="background: #FFF3A3A6;">주파수 영역 변환(2D-DFT/2D-DCT)은 정보 손실 없는 가역 변환이므로, 고주파(경계·텍스처)와 저주파(배경)가 자연스럽게 분리되는 성질을 이용해 학습 가능한 주파수 가중치로 특정 대역을 선택적으로 강조/억제한다 — feature map 레벨(MSFFEM)과 RoI 레벨(CAREM) 두 곳에 plug-and-play로 삽입.</mark>

### ① MSFFEM (Multi-Scale Frequency Feature Enhancement Module)
- tiny object가 주로 매핑되는 P2 feature map만을 대상으로 함(P3~P6은 그대로).
- Feature map을 patch로 분할(PRM) → 각 patch에 2D-DFT 적용 → 채널 방향 융합 후 sigmoid로 적응형 주파수 가중치 W(u,v) 학습 → 2D-IDFT로 복원(IPRM) → 잔차 연결.
- 서로 다른 patch size의 여러 FFEM 분기를 concat → channel shuffle → group conv로 융합해 다양한 객체 스케일에 동시 대응.
- Patch size는 데이터셋별로 조정(AI-TOD [50,100], VisDrone2019 [(24,32),(48,64)], DOTA-v1.5 [64,128]).

> [!example]- 구현 디테일
> ```
> W(u,v) = σ( Σ_i Conv1x1(F(u,v))[i] )
> F̂(u,v) = F(u,v) · W(u,v)
> FFEM(P, ps) = P + P̂   (P̂: IDFT 복원 후 IPRM 결과)
> ```
> 수용영역 계산 근거: ResNet 기준 2~32px 크기 객체는 P2에서 약 20~50px 수용영역을 가지므로, patch가 너무 크면 배경 노이즈에 묻히고 너무 작으면 객체가 잘려 aliasing이 생긴다.

<mark style="background: #FFF9D6A6;">tiny object는 patch 단위 FFT 스펙트럼에서 다른 영역과 뚜렷이 구분되는 고유한 주파수 응답을 보인다(Figure 3). 학습 가능한 주파수 가중치는 이 차이를 직접 이용해 객체 대역만 선택적으로 증폭하므로, spatial-domain 강화보다 더 명시적으로 "약한 spatial 특징" 문제를 겨냥한다. 서로 다른 patch size 분기 병렬 융합은 intra-class variation에도 대응한다.</mark>

### ② CAREM (Channel Attention-based RoI Enhancement Module)
- MSFFEM이 feature map 전체에 적용되는 것과 달리, RoI-Align된 개별 RoI feature(7×7)에 적용.
- 2D-DCT 기반 Gaussian 가중 고주파 필터(σ=2)로 고주파 응답 x_high 추출.
- x_high를 GMP/GAP로 채널 벡터 압축 → 1×1 conv 2개 합산 후 sigmoid → 채널 가중치 W_c(squeeze-and-excitation [46] 유사) → 원본 RoI feature에 곱해 채널 선택적 강화.
- 이진 0-1 필터보다 Gaussian 필터가 저주파 정보를 일부 남겨 텍스처 손실을 줄이므로 우수함을 ablation으로 확인.

> [!example]- 구현 디테일
> ```
> x_high = IDCT( z · DCT(x) ),   z(u,v) = exp(-d_f² / 2σ²)
> x̂ = x · W_c
> ```
> RoI Gaussian 필터 σ=2. 손실 함수는 `L_total = L_rpn_cls + λL_rpn_reg + γ(L_head_cls + λL_head_reg)`, λ=γ=1, RPN·head 분류는 cross-entropy·회귀는 L1 — Faster R-CNN vanilla loss 구성 그대로이며 추가 auxiliary loss 없음(모듈이 feature enhancement이지 loss 항이 아니기 때문).

<mark style="background: #FFF9D6A6;">tiny object는 RoI 안에서 차지하는 비중이 feature map 전체에서보다도 더 작다. CAREM은 RoI 안에 이미 섞여 들어온 배경/저주파 성분을 한 번 더 걸러내 "어떤 채널이 tiny object를 잘 대변하는가"를 학습함으로써, 검출 파이프라인 마지막 단계에서 위치 추정·분류 정확도를 직접 겨냥한다. MSFFEM 단독(AP50 +0.6%p)보다 CAREM 단독(+1.2%p)의 효과가 더 크고 둘을 합치면 +2.0%p로 상호 보완적(Table 9).</mark>

### ③ SAS (Sample Augmentation Strategy)
- Few-shot 카테고리는 인스턴스 수에 반비례해 수평/수직/대각 flip을 최대 8배까지 적용(airplane/bridge ×4, storage tank ×2, swimming pool/windmill ×8).
- 지배적 카테고리(vehicle)는 이미지를 무작위로 일부 제거해 중복 샘플 축소.
- 근거: 카테고리별 patch에 2D Hann window → 2D-DFT → 로그 파워 스펙트럼 방사 평균 R_log(k)로 카테고리 간 주파수 분포가 실제로 다름을 정량 확인(Figure 6).

<mark style="background: #FFF9D6A6;">class imbalance는 단순히 "샘플이 적다"가 아니라 "지배 카테고리에 학습이 쏠려 few-shot 카테고리 특징이 애초에 제대로 학습되는지 알 수 없다"는 문제였다. R_log(k) 분석으로 카테고리 간 주파수 특성이 실제로 다름을 확인했기에, vehicle 샘플을 줄여도(369k→163k) 다른 카테고리 성능 저하 없이 개선이 가능함을 사전에 근거 있게 예상했고 실제로도 확인됐다(Table 8).</mark>

# 실험 결과

### 핵심 결과

| 벤치마크 | 지표 | Before | After |
|---|---|---|---|
| AI-TOD (전체, SAS 포함) | AP / AP50 | 20.6 / 50.4 | 24.8 / 58.1 |
| AI-TOD (SAS 제외) | AP / AP50 | 20.6 / 50.4 | 21.4 / 52.4 |

> [!note]- 세부 결과 및 Ablation
> #### 설정
> - 데이터셋: AI-TOD [6](8클래스, 평균 객체 12.8px, 800×800), VisDrone2019 [48](UAV, 10클래스), DOTA-v1.5 [3](항공영상, 16클래스, sliding window crop)
> - 지표: COCO 스타일 AP/AP50/AP75, AI-TOD 크기별 AP(APvt[2,8px]/APt[8,16px]/APs[16,32px]/APm[32px+])
> - Baseline: Faster R-CNN w/ RFLA [36](ResNet-50) — RFLA가 이미 tiny object에 유리한 label assignment를 제공해 공정 비교 위해 채택
> - 학습: MMDetection, ResNet-50, SGD(momentum 0.9, wd 1e-4), batch 2, 12 epoch, 단일 RTX 4090
>
> #### 벤치마크별 개선
> | 벤치마크 | 지표 | Before | After | 비고 |
> |---|---|---|---|---|
> | AI-TOD (SAS 제외) | AP/AP50/AP75 | 20.6/50.4/12.9 | 21.4/52.4/13.7 | 모델 크기·FLOPs 거의 동일, FPS 42.5→38.7(-9%) |
> | AI-TOD (전체) | AP/AP50/AP75 | 20.6/50.4/12.9 | 24.8/58.1/17.5 | APvt 7.0→10.7, APt 20.8→24.9, APs 25.7→31.6 — test set 최고(2위 MENet 20.4%, RFLA 20.6%) |
> | VisDrone2019 (SAS 미적용) | AP/AP50/APt | 21.1/41.6/7.0 | 21.7/43.0/7.5 | 2위 DetectoRS(AP 21.6) 상회 |
> | DOTA-v1.5 (SAS 미적용) | AP/AP50/APt | 40.0/67.5/13.3 | 40.5/68.5/14.2 | |
>
> #### 다른 프레임워크로의 일반화 (Table 2, AI-TOD test)
> | Method | AP | AP50 | AP75 | FPS |
> |---|---|---|---|---|
> | Faster R-CNN [7] → +Ours | 11.1→12.1 | 26.3→27.9 | 7.6→8.7 | 45.9→37.3 |
> | NWD-RKA [35] → +Ours | 18.8→20.1 | 47.5→50.1 | 11.1→12.0 | 40.7→38.9 |
> | RFLA [36] → +Ours | 20.6→21.4 | 50.4→52.4 | 12.9→13.4 | 42.5→38.7 |
> | Cascade R-CNN [52] → +Ours | 13.7→13.9 | 30.6→31.3 | 10.0→10.5 | 33.7→27.2 |
>
> 파라미터/연산량 증가는 무시할 수준이나 FPS는 프레임워크별 9~19% 감소(대부분 CAREM의 RPN 의존 RoI 처리에서 발생, MSFFEM 자체 영향은 미미).
>
> #### Ablation — 모듈 조합 (Table 9, AI-TOD test)
> | MSFFEM | CAREM | SAS | AP | AP50 | AP75 | APvt |
> |---|---|---|---|---|---|---|
> | - | - | - | 20.6 | 50.4 | 12.9 | 7.0 |
> | ✓ | - | - | 21.0 | 51.0 | 13.5 | 8.9 |
> | - | ✓ | - | 21.4 | 51.6 | 13.8 | 8.0 |
> | ✓ | ✓ | - | 21.4 | 52.4 | 13.7 | 8.2 |
> | - | - | ✓ | 24.4 | 58.1 | 16.7 | 9.0 |
> | ✓ | ✓ | ✓ | 24.8 | 58.1 | 17.5 | 10.7 |
>
> #### 세부 발견
> - MSFFEM patch size(Table 4): patch 10/25는 APvt(8.8%/9.1%)에 유리, patch 100은 APm(32.9%, 최고)에 유리 — 작을수록 극소 객체, 클수록 중간 크기에 유리. Patch 50이 전체 AP(21.2%)·AP50(51.8%) 최적.
> - CAREM σ 영향(Table 7): σ=2에서 AP 21.4%(최고), σ=0.2는 APvt는 더 높지만(8.3%) AP50은 낮음(52.2% vs 51.6%) — trade-off 존재.
> - SAS 세부(Table 8): few-shot 카테고리(airplane/bridge/swimming pool/windmill) 각각 +8.3%p/+5.8%p/+10.9%p/+4.8%p AP. Vehicle 인스턴스 369k→163k로 줄여도 다른 카테고리 저하 없이 vehicle 자체만 24.6%→21.3%(중복 제거 효과, 정보 손실 없음 시사). 이미 샘플 충분한 카테고리(storage tank·ship·person)는 개선폭 1.2% 미만.
> - FFEM을 P3에도 적용하면 오히려 AP 소폭 하락(21.2%→20.9%, Table 5) — 대부분 tiny object가 P2에만 매핑되어 다중 레벨 적용이 항상 이득은 아님.
> - CAREM에서 필터 없이 채널 attention만 적용해도 AP+0.3%p(Table 6) — 채널 attention과 고주파 필터 효과가 서로 독립적으로 기여.
> - 0-1 이진 필터보다 Gaussian 필터가 우수(AP 21.2%→21.4%) — 저주파 정보를 완전히 버리지 않는 것이 유리.
> - VisDrone2019/DOTA-v1.5에서도 MSFFEM+CAREM 조합이 각 단독보다 우수(Table 10) — AI-TOD 국한 효과 아님.

# Discussion

### 이 아이디어의 잠재적 부작용
- 초극소 객체(2~8px)는 정보 자체가 거의 없어 주파수 필터링 효과가 제한적일 위험 → <mark style="background: #FF5582A6;">논문도 인정 — patch size 100 설정에서 APvt가 baseline(7.0%)보다 낮아지고(7.6%), σ=2 CAREM의 APvt 개선폭도 APm 대비 크지 않음. 전체 조합에서 APvt가 크게 개선되는 것(7.0%→10.7%)은 주파수 필터링보다 SAS의 샘플 증가 기여가 더 컸을 가능성이 있으나, 논문은 두 요인의 기여도를 분리하지 않는다.</mark>
- 학습된 주파수 가중치가 특정 데이터셋 통계에 최적화되어 도메인 전이 시 부적합할 위험 → <mark style="background: #FF5582A6;">저자도 "contextual pattern이 도메인 간 크게 바뀌면 효과가 달라질 수 있다"고 명시(§6.2)하며 domain adaptation을 향후 과제로 남겨 미해결.</mark>

### 한계
- <mark style="background: #FF5582A6;">Very tiny object는 객체 영역 내 고유 정보 자체가 부족해 주변 문맥에 의존할 수밖에 없다 — 주파수 강화는 경계를 부각시킬 뿐 없던 정보를 만들어낼 수는 없음(저자 명시, §6.2).</mark>
- <mark style="background: #FF5582A6;">극소 스케일에서는 bounding box 위치의 미세한 차이도 IoU에 큰 영향을 줘 annotation·평가 자체가 불안정할 수 있음(저자 지적, §5.2.1) — FANet만의 한계는 아니나 수치 해석 시 감안 필요.</mark>
- <mark style="background: #FF5582A6;">정적 이미지에 국한 — 영상 시퀀스의 시간적 정보는 미활용(저자가 향후 과제로 명시).</mark>
- CAREM 도입으로 추론 속도 9~19% 저하(FPS 42.5→38.7) — 실시간 애플리케이션에서는 트레이드오프 고려 필요.

### 생각할 점
- <mark style="background: #A6E3A1A6;">MSFFEM/CAREM은 "고주파=경계·텍스처, 저주파=배경"이라는 가정에 기반한다. 이 가정이 깨지는 상황(고주파 텍스처를 가진 도심·암반 배경, 또는 안개·저조도로 고주파 성분이 거의 없는 영상)에서는 역효과가 날 수 있으나 논문은 이런 실패 조건을 별도 분석하지 않는다 — 배포 전 검증 필요.</mark>
- SAS의 R_log(k) 분석은 카테고리 간 주파수 특성이 다름을 보였을 뿐, 카테고리별 최적 증강 배수까지 최적화하지는 않는다 — 현재는 인스턴스 수 반비례 휴리스틱(×1~×8)인데, R_log(k) 유사도 자체를 증강 강도에 반영하는 것도 가능해 보인다.

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">[[Frequency_Domain_Feature_Attention]]으로 분류된 MSFFEM/CAREM은 [[Small_Object_Detection_Approaches]] 비교표 기준 "feature 강화(주파수 영역)" 축에 속한다. [[Unc-SOD]]의 instance-level uncertainty 기반 sampling(label assignment 축)과는 직교적 개선이므로, RFLA에 FANet을 얹었을 때 이미 상호 보완적 이득이 확인된 것처럼(Table 2, RFLA AP50 50.4→52.4) 두 축을 결합하면 추가 이득 여지가 있다.</mark>
- <mark style="background: #A6E3A1A6;">[[SR-TOD]]의 self-reconstruction difference map은 정보 손실이 큰 영역을 공간적으로 찾는 방식인데, FANet의 R_log(k) 같은 주파수 통계와 결합하면 "공간적으로 어디에" + "주파수적으로 어떤 대역에" 정보가 몰려 있는지 동시에 활용하는 하이브리드 강화가 가능할 것으로 보인다.</mark>

# 관련 개념
- [[Frequency_Domain_Feature_Attention]] — 이 논문의 핵심 기여인 MSFFEM/CAREM이 사용하는 주파수 영역(DFT/DCT) 기반 적응형 attention 기법. 이 개념 문서의 "등장 논문"에 FANet이 이미 등재되어 있음.

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]] — feature 강화 계열 중 "주파수 영역" 축으로 분류되며, label assignment 축인 [[Unc-SOD]], self-reconstruction 축인 [[SR-TOD]] 등과 대비됨.

# 읽어볼 만한 논문
- 참고문헌 기반: Z. Shi et al., "HS-FPN: High Frequency and Spatial Perception FPN for Tiny Object Detection" [44] (AAAI 2025) — FANet의 CAREM이 필터 설계(0-1 필터 비교)를 참고한 논문으로, 고주파 응답을 attention으로 활용하는 가장 직접적인 선행 연구. MSFFEM/CAREM과의 차이(feature map+RoI 이중 적용 vs 단일 지점)를 이해하는 데 필수적.
- 참고문헌 기반: C. Xu et al., "RFLA: Gaussian receptive field based label assignment for tiny object detection" [36] (ECCV 2022) — 이 논문의 baseline이자 비교 대상. Label assignment 축(gap 서술의 갈래 1)의 대표 논문으로, FANet의 feature 강화 축과 직교적인 접근이라 결합 가능성을 검토할 때 먼저 읽을 만함. reading-list에 이미 등재되어 있음([[Unc-SOD]] 경유).
- 참고문헌 기반: B.N. Patro et al., "SpectFormer: Frequency and Attention is what you need in a Vision Transformer" [43] (arXiv 2023) — self-attention 자체를 spectral attention으로 대체하는 접근. FANet은 CNN 기반 R-CNN 구조에 주파수 모듈을 plug-in하는 방식인데, Transformer 백본에서 주파수 정보를 원천적으로 다루는 방식과 비교하면 "어디에 주파수 정보를 넣을지"에 대한 설계 스펙트럼을 넓게 볼 수 있다.
- 참고문헌 기반: B. Cao et al., "Visible and Clear: Finding Tiny Objects in Difference Map" (SR-TOD) [64] (ECCV 2024) — 이미 [[SR-TOD]]로 위키에 등재된 논문. FANet과 같은 RFLA baseline 위에서 비교되며(Table 11, AP 20.6%로 동률), self-reconstruction 기반 difference map이라는 전혀 다른 신호로 "약한 feature 보강"이라는 같은 문제를 푼다 — Discussion의 후속 연구 아이디어(공간+주파수 하이브리드)를 검토할 때 재독 가치가 있음.
- 자유 추천(검증 필요): 원격탐사·항공 이미지에서 반복적 텍스처(건물 격자, 농경지 패턴 등)가 주파수 도메인 필터링에 미치는 영향을 다루는 연구 — 검색 키워드: `periodic texture frequency domain false positive remote sensing object detection`. Discussion에서 제기한 "고주파=경계라는 가정이 깨지는 상황"에 대한 실증 연구가 있는지 확인할 때 유용할 것으로 예상.
