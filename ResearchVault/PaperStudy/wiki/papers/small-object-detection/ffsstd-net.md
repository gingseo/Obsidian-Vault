---
title: "From Fuzzy Global to Clear Local: A Focus and Super-Resolution-Guided Tiny Target Detection Method for Full-Scene Images"
authors: [Yucong He, Gui Gao, Zhenghuan Xia, Dunyun He, Gang Yang, Xi Zhang, Gaosheng Li]
year: 2026
venue: "IEEE Transactions on Geoscience and Remote Sensing (TGRS)"
jcr_quartile: Q1
task: [small-object-detection]
direction: [improvement]
tags: [paper, small-object-detection, remote-sensing, full-scene-image, focus-detection, super-resolution, region-filtering]
status: read
user_read: false
added: 2026-08-05
source: "raw/small-object-detection/2026_TGRS_FFSSTD-Net.pdf"
created: 2026-08-05
---

# 한 줄 요약
<mark style="background: #FFF3A3A6;">전체 장면(full-scene) 위성 이미지를 패치로 나눈 뒤, 경량 query 메커니즘(CFD)으로 타겟이 있을 만한 패치만 걸러 배경 연산을 없애고, super-resolution 보조 브랜치(FSR)로 backbone이 고해상도 특징을 학습하도록 유도해, 정보 손실·연산 비용·샘플 불균형 세 문제를 동시에 완화하는 FFSSTD-Net.</mark>


# 문제 정의

### 기존 방법의 한계
- **정보 손실**:
  Backbone의 다중 convolution 레이어를 거치며 타이니 객체의 공간·의미 정보가 심하게 손상된다. Scale-aware·contextual modeling 전략은 backbone/detector 설계 개선에 그쳐, 약화된 feature를 보강할 보조 정보 없이는 예측 단계에서 정확한 제약을 걸기 어렵다.
- **RONI(Region of Noninterest)로 인한 추가 연산 비용**:
  Full-scene 이미지의 큰 규모와 불균일한 타겟 분포 때문에 배경 영역까지 연산해야 해 효율이 떨어지고 오탐(false alarm) 가능성도 커진다. 기존 focused detection 전략은 수작업 라벨링이나 부실하게 설계된 보조 구조에 의존해, 연산 절감이라는 목적 자체를 훼손하거나 오히려 정보 손실을 가중시킨다.
- **불균형 샘플**:
  기존 sample-oriented 전략은 데이터셋 규모 확장에만 집중해 저품질·저해상도 샘플로 인한 불균형은 방치한다. SR 기반 복원은 구조적 세부 정보 손실을 완화할 수 있지만, 모델 크기가 커서 end-to-end 프레임워크 통합이 어렵다.

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Focused detection(image clip 기반) 전략**
- 단순 tiling [24]: 서브 영역으로 분할·리스케일 — 불균일 분포에 취약하고 모든 서브 이미지를 처리해야 해 시간 소모가 큼.
- ClusDet [27], DMNet [28] 등 clustering/density map 기반: 클러스터 영역을 근사 localization — end-to-end 통합이 안 되어 클러스터링·밀도 맵 추정 품질에 정확도가 좌우됨.
- PRDet [33]: 영역 grid semantic 정보와 contextual learning 결합 — 최적화 기법과 영역 전처리에 크게 의존해 모델 복잡도·연산 비용이 증가.

**갈래 2 — Sample-oriented / SR 기반 전략**
- 데이터 증강(멀티스케일 타겟을 빈 영역에 복제) [36]: 저해상도·중복 샘플을 유입시켜 오히려 학습 데이터 품질을 저해할 수 있음.
- GAN 기반 SR [43]–[46]: 생성자-판별자 적대적 학습으로 고해상도 증강 — 가짜 텍스처·아티팩트를 만들어 탐지 헤드 정확도를 방해하고, 모델 크기 확장으로 연산 비용이 크게 증가.

**갭**: <mark style="background: #FFF3A3A6;">Focused detection 계열은 연산 절감이라는 목표 자체를 정확히 달성하지 못하거나(수작업 라벨/클러스터링 의존) end-to-end 통합이 안 되고, SR 기반 계열은 정보 손실을 완화하는 대신 새로운 아티팩트나 무거운 연산 비용을 대가로 치른다. "고해상도 표현 보존"과 "연산 오버헤드 절감"을 동시에, end-to-end로, 추가 라벨 없이 달성하는 통합 프레임워크는 없었다.</mark>

### 이 논문이 풀고자 하는 문제
1. 수작업 라벨링이나 별도 클러스터링 없이, 경량 네트워크만으로 타겟이 있을 만한 영역을 걸러내 배경 연산을 없애는 것
2. 모델 크기를 키우지 않으면서 backbone이 고해상도 특징을 학습하도록 유도해, 추론 시에는 SR 연산 자체를 제거하는 것
3. 위 두 모듈을 기존 backbone/detector에 구조 변경 없이 이식 가능하게 만드는 것

# 제안 방법

<mark style="background: #FFF3A3A6;">"흐릿한 전역에서 명확한 지역으로(From Fuzzy Global to Clear Local)"라는 이름처럼, 먼저 전역적 관점에서 타겟이 있을 만한 지역을 빠르게 좁히고(CFD), 그 다음 backbone이 그 지역의 특징을 더 선명하게(고해상도로) 학습하도록 유도한다(FSR). 두 모듈 모두 추론 시 연산량을 늘리지 않으면서 기존 backbone에 plug-in 가능하도록 설계했다.</mark>

### ① Convolution Focus Detection (CFD) 모듈
- 원본 이미지를 1024×1024 패치로 자르되 인접 패치와 100픽셀 겹치게 해, 패치 경계에 걸친 타겟도 온전히 한 패치에서 포착.
- Backbone의 첫 레벨 feature(`F^sf`, 저수준)와 마지막 레벨 feature(`F^df`, 고수준)를 각각 CR(3×3 conv+ReLU) 처리 후 업샘플링·concat해 두 정보를 융합.
- 융합된 feature map을 S×S(128×128) grid로 나누고, 각 grid에 타겟 중심점이 포함되는지를 이진 분류 문제로 프레임화.
- 두 개의 1×1 conv로 grid별 점수(`F^A`)를 산출하고, patch 내 최대 grid 점수가 통계 기반 적응 임계값 `Z`를 넘으면 "confirmed patch", 아니면 배경으로 간주해 이후 연산에서 배제.

> [!example]- 구현 디테일
> ```
> F_i^A = c1×1² ∘ CR ∘ Concat(CR(F_i^sf), upsample(F_i^df))
> z_j^mx = max(F_A[i,1], ..., F_A[i,s])   # patch 내 최대 grid 점수
> Z = ε * ((평균 z^mn + 평균 z^mx) / 2)²   # ε=0.25, 통계 기반 적응 임계값
> L_CFD = focal loss(α=0.75, γ=2.0)       # 배경 negative 샘플 과다 문제 대응
> ```
> Ablation 기준 grid 크기 128×128, α/γ는 grid search로 결정.

<mark style="background: #FFF9D6A6;">Full-scene 이미지는 대부분 배경(RONI)이고 타겟은 희소하게 분포하므로, 이 grid 단위 스코어링으로 배경 patch를 사전에 배제하면 별도 클러스터링·밀도 맵 추정 없이도 "문제 정의"의 RONI 연산 비용 문제를 해결하고, 배경 노이즈로 인한 오탐도 함께 줄인다.</mark>

### ② Feature Super-Resolution (FSR) 모듈
- CFD가 통과시킨 confirmed patch만 입력으로 받아, 저수준·고수준 feature를 concat한 뒤 3개의 3×3 conv와 EDSR(BN 제거된 residual 구조) + deconvolution으로 원본 이미지를 재구성하도록 학습.
- 원본 이미지 `G`와 재구성 결과 `G_s`의 L1 loss로 지도학습(`L_s = ||G - G_s||_1`) — L2 대신 L1을 쓴 이유는 HR feature의 multimodal 분포를 L2가 뭉개 blurry한 예측을 만들기 때문.
- 추론 시에는 이 브랜치 전체가 제거되어(auxiliary branch), backbone 자체가 이미 고해상도 지향적으로 학습된 특징을 내놓으므로 추가 연산이 발생하지 않음.

> [!example]- 구현 디테일
> ```
> c_EDSR(F_i) = c3(c3(F_i)) ⊕ c1(F_i)          # BN 제거된 residual block
> G_FSR = c_EDSR³ ∘ c3³ ∘ Concat(CR(F_i^df), upsample(F_i^sf))
> L_total = r1*L_d + r2*L_CFD + r3*L_s          # r=(1.0, 1.0, 0.1)
> ```
> 최적 feature 조합은 C1+C4(저수준+최고수준) — DOTAv2 기준 mAP 56.75로 C1+C3(56.24), C2+C3(55.89)보다 우수.

<mark style="background: #FFF9D6A6;">학습 시에만 존재하는 보조 브랜치로 원본 이미지 복원을 강제함으로써, backbone이 업샘플링 보간이 만드는 blur 대신 실제 고해상도 구조·텍스처 정보를 latent space에 담도록 유도한다. 추론 시 브랜치를 제거하므로 "문제 정의"의 정보 손실·샘플 불균형 문제를 모델 크기 증가 없이 완화한다는 것이 SR 기반 선행 연구와의 핵심 차이다.</mark>

# 실험 결과

### 핵심 결과
| 벤치마크 | 지표 | Before(CFD·FSR 미적용, A-ConvNext) | After(FFSSTD-Net) |
|---|---|---|---|
| FAIR1M | mAP | 43.88 | 46.25 |
| DOTA-v2.0 | mAP | 53.65 | 56.75 |

> [!note]- 세부 결과 및 Ablation
> #### 모듈별 기여 (DOTA/FAIR1M/SODA)
> | CFD | FSR | mAP-FAIR1M | Fps-FAIR1M | mAP-DOTAv2 | Fps-DOTAv2 | mAP-SODA |
> |---|---|---|---|---|---|---|
> | × | × | 43.88 | 18.7 | 53.65 | 16.8 | 33.81 |
> | ✓ | × | 44.35 | 22.5 | 53.17 | 21.3 | 34.27 |
> | × | ✓ | 45.78 | 18.7 | 56.23 | 16.8 | 37.71 |
> | ✓ | ✓ | 46.25 | 22.5 | 56.75 | 21.3 | 38.20 |
>
> #### Full-scene 이미지 실측(Gaofen-2 6장)
> - CFD 적용 시 mAP 52.14→52.61, Fps(타일 처리량) 16.1→23.8로 대폭 향상.
> - Patch 제거 비율 실험: precision 약 95.94%(제거 비율 20%) 지점에서 속도·정확도 모두 개선되는 최적점.
>
> #### SOTA 비교(SODA-A, 9개 클래스)
> - FFSSTD 38.2% mAP로 RoI Transformer(36.0%), Oriented R-CNN(34.4%), Faster R-CNN(32.5%) 대비 우수.
> - Helicopter, Windmill처럼 인스턴스 수가 적은 클래스는 상대적으로 낮은 AP.
>
> #### 다른 아키텍처로의 일반화(DOTAv2/FAIR1M)
> - RetinaNet, Faster R-CNN, YOLOv3, TOOD, RTMDet, Oriented R-CNN, Mamba 등에 CFD+FSR을 이식 시 평균 mAP +10%, Fps +3 frames/s.
> - 다양한 backbone(ResNet101, CSP-DarkNet)에서도 CFD/FSR 개별 적용 시 일관된 정확도·속도 개선 확인.

# Discussion

### 이 아이디어의 잠재적 부작용
- CFD가 실제 타겟을 포함한 patch를 잘못 배제(false negative)할 위험 → <mark style="background: #FF5582A6;">논문은 "잘못 필터링된 patch는 대체로 객체 수가 매우 적어 탐지 자체가 어려웠을 patch"라고 주장하며 정당화하지만, 정량적인 recall 손실 분석은 제시하지 않는다.</mark>
- FSR이 학습에만 관여하고 추론 시 제거되므로, 학습·추론 간 feature 분포 불일치 가능성 → <mark style="background: #FF5582A6;">논문에서 별도로 검증하지 않았다.</mark>

### 한계
- <mark style="background: #FF5582A6;">CFD 모듈이 대규모 라벨링된 데이터셋에 강하게 의존한다 — attention을 가이드하려면 사전 라벨된 관심 영역이 필요해, 라벨이 부족하거나 공간적으로 편향된 시나리오에서 적용성이 제한된다고 저자가 명시(Discussion).</mark>
- <mark style="background: #FF5582A6;">FSR의 업샘플링 전략이 여전히 개선 여지가 있다 — 극도로 밀집하거나 저대비인 영역에서 재구성 아티팩트 가능성을 저자가 인정하며, transformer 기반 계층적 복원이나 diffusion 기반 개선을 향후 과제로 제시.</mark>
- <mark style="background: #FF5582A6;">대형/소형 차량(l-vehicle/s-vehicle) 클래스 간 혼동이 여전히 존재 — 크기 정의의 모호성과 학습 샘플의 불완전한 분할에서 기인한다고 분석.</mark>

### 생각할 점
- <mark style="background: #A6E3A1A6;">CFD의 "저해상도 예측으로 고해상도 연산 위치를 좁힌다"는 구조는 [[QueryDet]]의 Cascade Sparse Query와 문제의식이 매우 유사하다 — 둘 다 coarse-to-fine 방식으로 배경 연산을 없애지만, QueryDet은 feature pyramid 레벨 간 sparse convolution을, FFSSTD-Net은 patch 단위 grid 스코어링을 쓴다는 점에서 구현 층위가 다르다.</mark>
- <mark style="background: #A6E3A1A6;">FSR의 "학습 시에만 존재하는 auxiliary reconstruction branch"는 [[sr-tod]]의 self-reconstruction difference map과 목적이 다르다 — SR-TOD는 재구성 오차 자체를 attention prior로 쓰는 반면, FFSSTD-Net은 재구성 학습이 backbone feature 품질을 간접적으로 끌어올리는 정규화 역할만 한다(추론 시 오차 맵을 쓰지 않음).</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">CFD의 patch 필터링과 [[QueryDet]]의 CSQ를 결합하면, patch 단위 1차 필터링 후 남은 patch 내부에서 다시 sparse query로 연산을 좁히는 2단계 가속이 가능할 것으로 보임.</mark>
- <mark style="background: #A6E3A1A6;">[[RS-TOD]], [[FANet]]처럼 원격탐사 도메인에서 attention 기반 feature 강화를 다루는 논문들과 달리 이 논문은 "연산 비용 절감"에 초점을 맞춘 유일한 원격탐사 소형 객체 탐지 논문 — MOC의 "아직 못 채운 빈틈"(경량화 계열은 [[LSOD-YOLO]] 하나뿐)을 보완하는 위치.</mark>

# 관련 개념
- (없음 — CFD/FSR은 이 논문 안에서만 의미 있는 구현 디테일로 판단, 별도 concept 문서로 만들지 않음)

# 관련 문서
- 비교: [[small-object-detection-approaches]]

# 읽어볼 만한 논문
- 참고문헌 기반: G. Cheng et al., "Towards large-scale small object detection: Survey and benchmarks" (IEEE TPAMI 2023) [14] — 이 논문이 참조하는 4갈래 분류(scale-aware/contextual/focused/sample-oriented) 체계의 원 서베이. [[sr-tod]] 노트에서도 동일 서베이가 이미 5갈래 분류로 인용되어 교차 확인 가치가 있음.
- 참고문헌 기반: C. Xu, J. Wang, W. Yang, H. Yu, L. Yu, G.-S. Xia, "RFLA: Gaussian receptive field based label assignment for tiny object detection" (ECCV 2022) [38] — 이미 `wiki/reading-list.md`에 [[Unc-SOD]] 출처로 등재된 논문과 동일. Full-scene 원격탐사 타이니 객체 탐지에서도 반복적으로 인용되는 것으로 보아 우선순위가 높음.
- 자유 추천(검증 필요): Focus-and-Detect 계열의 최신 후속 연구(2025~2026) — 이 논문이 인용한 Koyun et al., "Focus-and-detect" (2022)의 최신 확장판 존재 여부. 검색 키워드: `focus and detect small object aerial images 2025 2026`
