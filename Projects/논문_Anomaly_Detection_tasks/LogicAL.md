---
pm-task: true
projectId: "paperwiki-anomaly-detection"
parentId:
id: "t-logical-ywgh6l9i47"
title: "LogicAL: Towards logical anomaly synthesis for unsupervised anomaly localization"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-08-24"
due:
progress: 0
assignees: []
tags: []
customFields:
  "poo7xxb0mtck1e8q": 2024
  "0sgdhb2bmtck1e8r": "CVPRW (CVPR Workshops)"
subtaskIds: []
dependencies: []
year: 2024
venue: "CVPRW (CVPR Workshops)"
jcr_quartile: Workshop
task: [anomaly-detection]
direction: [novel-approach, improvement]
paper_tags: [paper, anomaly-detection, anomaly-synthesis, logical-anomaly, edge-manipulation, unsupervised, industrial-inspection]
source: "/Users/GyeongSeo/Workspace/논문_pdf/Anomaly_Detection/2024_CVPRW_LogicAL.pdf"
createdAt: "2026-08-24T03:44:00.000Z"
updatedAt: "2026-08-24T03:44:00.000Z"
---

Project: [[논문 읽기|논문 읽기]]
#paper #anomaly-detection #anomaly-synthesis #logical-anomaly #edge-manipulation #unsupervised #industrial-inspection

# 한 줄 요약
<mark style="background: #FFF3A3A6;">기존 이상 합성 기법들이 구조적 결함(scratch, dent 등) 위주로 편향되어 논리적 이상(부품 누락, 개수 초과, 위치 오배치 등)을 놓친다는 문제를, 이미지가 아니라 edge map을 조작(제거·교체·병합)한 뒤 edge-to-image generator로 사실적인 이상 이미지를 생성하는 LogicAL — SAM 기반 semantic 영역 선택과 임의 영역 선택을 결합해 논리적·구조적 이상을 모두 합성하고, edge reconstruction을 네트워크에 도입해 MVTecLOCO에서 AU-sPRO 69.7, MVTecAD에서 AU-ROC 98.3을 달성.</mark>

# 문제 정의

### 기존 방법의 한계
- **구조적 결함 편향(structural bias)**:
  기존 비지도 이상 합성 방법(CutPaste, Draem, OmniAL 등)은 정상 이미지의 외관을 무작위로 훼손해 이상을 만드는데, 이렇게 생성된 이상은 대부분 시각적 구조 자체가 정상과 다른 "구조적 결함"에 편향되어 있다 — 논리적 제약(부품 개수, 위치, 조합)을 위반하는 이상은 다루지 않는다.
- **논리적 이상의 특수성**:
  실제 논리적 이상(Fig. 1b)은 각 구성요소 자체는 흠 없이 정상적이지만 그 조합이 정상 데이터의 논리적·기하학적 제약을 위반한다(예: pushpin 1개 추가, hex nut 1개 추가). 구조적 이상 합성만으로 학습된 탐지기는 이런 논리적 이상에 취약하다.
- **기존 논리적 이상 합성 시도의 한계**:
  SLSG는 Draem의 Perlin noise 파라미터와 이진화 threshold를 조정해 더 집중된 이상 영역 마스크를 얻지만, 여전히 이산적인 국소 영역만 수정하고 장거리 의존성(long-range dependency)을 무시해 구조 지향적 결함 생성에 머문다. 이때 단순한 대규모 영역 교체가 만드는 가짜 경계(fake boundary)는 탐지기가 지나치게 단순한 결정 경계를 학습하게 만든다.
- **도로 장면 재합성 기법의 한계**:
  Semantic map 기반 재합성(GAN)을 쓰는 도로 이상 탐지 방법들(Synboost 등)은 정상 영역에서도 큰 차이를 유발해, 이상이 아닌 부분까지 오탐될 위험이 있다.

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — 구조적 이상 합성**
- CutPaste [15]: 국소 직사각형 영역을 잘라 임의 위치에 붙여넣기 — 단순하지만 효과적, 그러나 결함 형태가 규칙적.
- Draem [31]: binarized Perlin noise로 불규칙 이상 영역 마스크 생성, 외부 DTD 텍스처 데이터셋으로 다양성 확보.
- JNLD [33]: just noticeable distortion 기반 multi-scale 이상 생성.
- OmniAL [34]: JNLD를 panel-guided 전략으로 확장, N-클래스 통합 탐지기 학습.
- 공통 한계: 구조적 이상에 집중, 논리적 이상은 다루지 않음.

**갈래 2 — 논리적 이상 합성 시도**
- SLSG [29]: Draem의 Perlin noise 파라미터·threshold 조정으로 더 집중된 마스크 — 여전히 이산적 국소 영역 수정에 머묾, 이 논문이 가장 직접적으로 극복 대상으로 삼는 선행 연구.

**갈래 3 — 도로 장면 재합성 기반**
- Lis et al. [17], Xia et al. [28]: 예측된 semantic map으로부터 GAN 기반 이미지 재합성.
- Synboost [9]: 불확실성 맵으로 재합성 방법을 개선.
- 공통 한계: 도로 장면에서는 정상 영역까지 큰 차이를 유발하는 경향.

**갈래 4 — 거리/재구성 기반 탐지기**
- SPADE [6], PaDim [8], PatchCore [21]: ImageNet 사전학습 feature 기반 거리 측정 — 합성 자체보다는 탐지 알고리즘에 집중.
- S-T [3], GCAD [4]: teacher-student 구조, GCAD는 global+local branch로 논리·구조 이상 모두 대응하려 시도한 원조(MVTecLOCO 데이터셋 논문).
- SLSG [29]: 그래프 컨볼루션으로 위치 관계 추론.

**갭**: <mark style="background: #FFF3A3A6;">기존 방법들은 이미지 픽셀을 직접 조작하거나 국소 영역만 수정해 구조적 이상 생성에 유리한 반면, "사실적이면서도 논리적 제약을 위반하는" 이상을 생성하는 중간 표현(intermediate representation) 기반 접근은 없었다. Edge라는 object-agnostic한 중간 표현을 조작해 논리적·구조적 이상을 하나의 통일된 프레임워크로 생성하는 방법이 필요했다.</mark>

### 이 논문이 풀고자 하는 문제
1. 사실적(photo-realistic)이면서 논리적 제약(개수, 위치, 조합 등)을 위반하는 이상 이미지를 합성하는 것
2. 동일 프레임워크에서 구조적 이상도 함께 합성해 두 종류의 이상을 균형 있게 다루는 것
3. 합성된 이상으로 학습한 재구성 기반 탐지기의 국소화(localization) 정밀도를 개선하는 것

# 제안 방법

<mark style="background: #FFF3A3A6;">이미지를 직접 수정하는 대신 edge map이라는 object-agnostic 중간 표현을 조작(제거·교체·병합)하고, 이를 edge-to-image generator(pix2pixHD 기반)로 사실적 이미지로 변환한다 — SAM 기반 semantic 영역에서 edge를 제거하면 부품이 통째로 사라지는 논리적 이상이, 임의 영역에서 다른 이미지의 edge로 교체하면 구조적 결함이 만들어진다. 탐지 네트워크에는 edge 정보를 재구성 목표로 추가해 국소화 정밀도를 높인다.</mark>

### ① Edge Detection & Edge-to-Image Generation
- 사전학습된 PiDiNet(pixel difference convolution 기반 경량 edge detector)으로 4개 스케일의 edge map을 zero-shot 추출, 가장 세밀한 스케일을 이후 조작·학습에 사용.
- Pix2pixHD(coarse-to-fine generator + multi-scale discriminator, conditional GAN)를 정상 데이터의 (edge map, 컬러 이미지) 쌍으로 학습해 edge-to-image 변환기를 확보.
- DeepSIM의 thin-plate-spline(TPS) warp로 학습 쌍을 증강(3×3 제어점을 무작위로 이동) — 이상 edge를 입력받았을 때 생성기가 전역적으로 붕괴된(collapsed) 결과를 내는 것을 방지.

<mark style="background: #FFF9D6A6;">"문제 정의"의 첫 번째 문제(사실적이면서 논리 위반적인 이상 생성)를, 픽셀이 아니라 edge라는 object-agnostic 중간 표현을 조작 대상으로 삼음으로써 해결한다 — edge는 밝기·색상·텍스처의 급격한 변화만을 나타내므로 특정 객체 카테고리에 국한되지 않고, 이를 조작한 뒤 생성기로 변환하면 "구성요소 자체는 사실적이지만 조합이 비정상적인" 이상을 만들 수 있다.</mark>

### ② Edge Manipulation (Region Selection + Modification)
- **영역 선택**: (a) semantic 영역 선택 — SAM으로 대략적인 semantic 영역을 얻은 뒤 배경 제거·소영역 그룹화·중첩 영역 병합으로 정제한 후보 영역 지도를 구성. (b) semantic-agnostic 영역 선택 — 서로 다른 종횡비·형태의 3개 영역을 무작위로 결합.
- **Edge 수정 전략**: (1) remove — 선택 영역의 정상 edge를 제거(부품 누락 등 논리적 이상). (2) replace — 선택 영역의 정상 edge를 같은 클래스의 다른 정상 이미지에서 뽑은(증강된) edge로 교체(구조적 결함 위조). (3) merge — 선택 영역의 정상 edge와 후보 이상 edge를 결합(더 복잡한 논리+구조 혼합 이상).

> [!example]- 구현 디테일
> ```
> E_Aj = Mj + (1-Mj)*Ej                              (remove)
>      = Mj*A(Ei) + (1-Mj)*Ej                        (replace)
>      = Mj*(A(Ei)+Ej-1) + (1-Mj)*Ej                 (merge)
> ```
> `Ej`: j번째 정상 이미지의 edge map, `Ei`: 후보 이상 edge(i번째 정상 이미지에서 추출), `A`: 무작위 증강, `Mj`: 선택 영역 마스크. 생성된 이상 edge map과 이미지는 다시 TPS warping으로 추가 증강 가능. Ablation(Table 3)에서 semantic 영역 단독 사용만으로 baseline(OmniAL, 85.2/63.2) 대비 3%p 이상 개선(89.0/68.6 근방), 모든 수정 전략을 결합하면 67.0 pixel AUsPRO까지 개선.

<mark style="background: #FFF9D6A6;">"문제 정의"의 두 번째 문제(구조·논리 이상의 균형)를, remove/replace/merge라는 서로 다른 edge 조작이 각각 논리 편향·구조 편향·혼합 이상을 만들도록 설계함으로써 해결한다 — 다만 저자도 논리와 구조 이상 사이에 모호성(ambiguity)이 있어 각 전략이 항상 한쪽에만 엄격히 대응하지는 않는다고 명시한다.</mark>

### ③ Edge Reconstruction을 포함한 이상 국소화 네트워크
- OmniAL의 네트워크 구조(DCSA 블록+DiffNeck을 갖춘 reconstruction·localization 서브네트워크)를 뼈대로 삼되, reconstruction 서브네트워크가 정상 이미지·JND(just noticeable distortion) map뿐 아니라 **edge map까지 재구성**하도록 확장.
- Edge map 재구성에는 PiDiNet을 따라 annotator-robust loss[18]를 적용, 나머지 재구성(이미지·JND)에는 MSE+SSIM 손실을 함께 사용.
- Localization 서브네트워크는 edge map의 도움을 받아 재구성 오차를 계산해 이상 영역을 더 정밀하게 국소화.
- GT 마스크는 합성 이상 이미지와 (원본 정상 이미지 또는 정상 edge로 생성한 이미지) 사이 SSIM으로 계산(생성기 오차로 인해 생성 이미지와 원본이 완전히 같지 않으므로, 과민한 탐지기를 피하기 위해 SSIM 기반 완화된 GT 사용), focal loss로 불균형(정상 vs 이상 픽셀) 보정.

<mark style="background: #FFF9D6A6;">"문제 정의"의 세 번째 문제(국소화 정밀도)를, edge라는 명시적 구조 정보를 재구성 목표에 추가함으로써 해결한다 — edge는 경계 위치에 민감한 신호이므로, 이미지 재구성만으로는 놓치기 쉬운 미세한 경계 이상(부품 위치 오배치, 누락 등)을 edge 재구성 오차가 보완적으로 잡아낸다.</mark>

# 실험 결과

### 핵심 결과 (MVTecLOCO test, 이상 합성 있는 방법 대비)
| 벤치마크 | 지표 | Before(SLSG, 종합 2위) | After(LogicAL) |
|---|---|---|---|
| MVTecLOCO | Mean 이미지 AU-ROC / 픽셀 AU-sPRO | 90.3 / 67.3 | 88.5 / 69.7 |
| MVTecAD | Mean 이미지 AU-ROC / 픽셀 AU-ROC | 98.5(SLSG) / 97.8(OmniAL+) | 98.8 / 98.3 |

> [!note]- 세부 결과 및 Ablation
> #### MVTecLOCO 카테고리별(Table 1)
> | 카테고리 | GCAD | SLSG | OmniAL | LogicAL |
> |---|---|---|---|---|
> | BreakfastBox | 83.9/50.2 | 88.9/**65.9** | 75.9/46.5 | 85.4/46.8 |
> | JuiceBottle | 99.4/91.0 | 99.1/82.0 | **99.5**/87.7 | 98.5/**91.3** |
> | Pushpins | 86.2/73.9 | **95.5**/74.4 | 79.6/59.6 | 87.4/**81.3** |
> | ScrewBag | 63.2/55.8 | 79.4/47.2 | **83.1**/53.2 | 82.0/52.3 |
> | SConnector | 89.3/79.8 | 88.5/66.9 | 88.1/69.1 | **89.0**/**76.3** |
> - Pushpins·SConnector에서 pixel AU-sPRO 최고(81.3/76.3), JuiceBottle에서도 최고(91.3) — "extra component"류 논리적 이상이 많은 카테고리에서 특히 강점.
>
> #### 논리 vs 구조 이상 분리 비교(Table 2, MVTecLOCO)
> | 방법 | Logical(I/P) | Structural(I/P) | Mean(I/P) |
> |---|---|---|---|
> | S-T | 66.4/49.7 | 88.3/75.6 | 77.3/62.6 |
> | SPADE | 70.9/53.6 | 66.8/36.8 | 68.9/45.1 |
> | GCAD | 86.0/71.1 | 80.6/69.2 | 83.3/70.1 |
> | SLSG | 89.6/- | 91.4/- | 90.3/67.3 |
> | **LogicAL** | 84.6/68.8 | **93.6/81.5** | 88.5/**69.7** |
> - **구조적 이상**에서 압도적 우위(93.6/81.5, 2위 SLSG 91.4 대비 image +2.2%p, pixel은 GCAD 69.2 대비 +12.3%p) — 저자의 논리 편향 개선 주장과 달리, 실제로는 구조적 이상 국소화에서 가장 큰 개선을 보임.
> - **논리적 이상**에서는 SLSG(89.6)·GCAD(86.0)보다 image AU-ROC가 낮음(84.6) — "논리 이상 합성을 채워 넣는다"는 목표를 완전히 달성했다고 보긴 어려운 결과.
>
> #### MVTecAD(Table 4, OmniAL+ = 저자의 합성 이상으로 재학습한 OmniAL)
> LogicAL image/pixel AU-ROC 98.8/98.3로 전체 최고. OmniAL(97.0/97.8)→OmniAL+(98.3/97.8)로 저자의 합성 이상만 적용해도 image AU-ROC 1%p 개선, 여기에 edge reconstruction까지 추가한 LogicAL이 추가로 0.5%p 개선.
>
> #### VisA(Table 5), MADsim(Table 6) 일반화
> - VisA: LogicAL mean 95.3/97.4/47.2(I-AUC/P-AUC/P-AP)로 OmniAL+(95.2/97.5/49.2) 대비 유사~근소 우위, JNLD·Draem 대비는 명확히 우위.
> - MADsim(3D LEGO, pose 다양성): LogicAL 67.6/86.0으로 CutPaste(59.3)·Draem(58.0) 등 다른 합성 기반 방법 대비 pixel AU-ROC 우위, 다만 PAD(자세 정렬 사용, 90.9/97.8)에는 크게 못 미침 — 저자는 "자세 정렬 없이" 달성한 성능이라는 점을 강조.
>
> #### Ablation(Table 3, MVTecLOCO)
> | 영역선택 | Edge 수정 | TPS | I | P |
> |---|---|---|---|---|
> | (baseline, OmniAL) | | | 85.2 | 63.2 |
> | Arbitrary | Replace | - | 87.6 | 67.4 |
> | Semantic | Remove | - | 89.0 | 68.6 |
> | Semantic | Replace | - | 89.1 | 68.1 |
> | Semantic | Merge | - | 88.5 | 67.8 |
> | Semantic | Remove+Replace | - | 88.7 | 67.9 |
> | Semantic | Remove+Replace+Merge | - | 88.5 | 67.0 |
> | Semantic+Arbitrary | 전체 | - | 88.5 | **69.7**(최고 P) |
> | Semantic+Arbitrary | 전체 | ✓ | **89.2**(최고 I) | 67.2 |
> - Semantic 단독 영역 선택이 baseline 대비 가장 큰 초기 개선(+3.8/+5.4). Arbitrary 영역 추가는 학습 샘플 다양성을 높여 +2.7%p 기여(저자 서술).
> - TPS warping 추가는 image AUROC +0.7%p이지만 pixel AUsPRO는 오히려 -2.5%p — 이미지 레벨과 픽셀 레벨 지표에 상반된 영향.

# Discussion

### 이 아이디어의 잠재적 부작용
- TPS warping이 image AU-ROC는 개선하지만 pixel AU-sPRO는 오히려 저하 → <mark style="background: #FF5582A6;">논문은 이 상반된 효과의 원인을 분석하지 않고 수치만 보고한다 — warping이 국소화 정밀도에 구체적으로 어떤 부작용을 미치는지 설명이 없다.</mark>
- Edge-to-image generator 자체의 오차로 인해 생성된 정상 이미지가 원본과 완전히 같지 않음 → <mark style="background: #FF5582A6;">저자도 이를 인지해 SSIM 기반 완화된 GT를 쓰지만, 이는 근본 원인(생성기 오차) 자체를 해결하는 것이 아니라 그 영향을 우회하는 방편이다 — 생성기 품질이 낮은 카테고리에서는 이 완화가 충분하지 않을 수 있다.</mark>

### 한계
- <mark style="background: #FF5582A6;">MVTecLOCO의 논리적 이상(Logical) 지표에서 SLSG(89.6)·GCAD(86.0)보다 낮은 image AU-ROC(84.6)를 기록 — "논리적 이상 합성 개선"이 이 논문의 핵심 주장인데, 정작 논리적 이상 자체의 image-level 탐지에서는 최고 성능이 아니다. 저자는 이 결과를 별도로 논의하지 않는다.</mark>
- <mark style="background: #FF5582A6;">MADsim에서 자세 정렬(pose alignment)을 활용하는 PAD(90.9/97.8)에 크게 못 미침(67.6/86.0) — "정렬 없이" 달성했다는 점을 강조할 뿐, 정렬 기법과 결합 시의 잠재력은 다루지 않는다.</mark>
- SAM 기반 semantic 영역 선택에 의존하는데, SAM이 처음부터 과분할(over-segmentation)되는 문제를 후처리(배경 제거, 소영역 그룹화, 중첩 병합)로 완화한다고만 서술할 뿐, 이 후처리 규칙 자체의 민감도나 실패 사례는 분석되지 않는다.
- Edge-to-image generator(pix2pixHD)를 카테고리(데이터셋)마다 별도로 300 epoch 학습해야 하므로, 새로운 제품 카테고리에 적용하려면 매번 추가 학습 비용이 발생 — 이 오버헤드가 실제 산업 적용에 미치는 영향은 논의되지 않는다.

### 생각할 점
- <mark style="background: #A6E3A1A6;">Table 2에서 LogicAL이 실제로는 구조적 이상(93.6/81.5)에서 가장 큰 개선을 보이고 논리적 이상에서는 오히려 SLSG·GCAD보다 낮다는 점은, "논리적 이상 합성을 개선한다"는 논문 제목의 핵심 주장과 실제 정량 결과 사이에 흥미로운 긴장 관계를 만든다 — remove 전략(논리 이상 담당)이 replace 전략(구조 이상 담당)만큼 효과적이지 않을 가능성을 시사한다.</mark>
- <mark style="background: #A6E3A1A6;">이 논문의 "edge를 중간 표현으로 조작해 이미지를 생성한다"는 전략은, 이 위키의 [[ORFENet]]·[[BAFNet]]·[[CoLR-Det]]이 다루는 "학습시에만 존재하는 auxiliary reconstruction branch"와 달리, edge 조작 자체가 데이터 증강(합성 이상 생성)의 도구로 쓰인다는 점에서 다른 층위의 활용이다 — "재구성"이 아니라 "생성"에 edge라는 중간 표현을 쓴다는 점이 특징적.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">이 논문의 semantic region selection(SAM)+edge manipulation 조합은, small object detection 분야의 [[CDATOD-Diff]]가 CLIP의 semantic 정보를 diffusion 조건으로 쓰는 것과 유사하게 "foundation model의 semantic 이해를 저수준 조작(여기서는 edge, CDATOD-Diff는 label assignment)에 결합"하는 패턴을 공유한다 — 서로 다른 task(anomaly synthesis vs label assignment)에서 유사한 상위 전략이 재발견되고 있다는 점이 흥미롭다.</mark>
- <mark style="background: #A6E3A1A6;">Edge reconstruction을 auxiliary loss로 추가하는 것이 국소화 정밀도를 높인다는 이 논문의 결과는, 이 위키의 [[BAFNet]] Boundary-Aware Branch(Laplacian pyramid 기반 경계 GT로 supervision)와 정확히 같은 아이디어를 anomaly detection과 salient/small object detection이라는 서로 다른 task에서 독립적으로 재발견한 사례다 — "경계 정보를 명시적 학습 목표로 추가하면 국소화가 개선된다"는 원리가 여러 dense prediction task에서 일반적으로 성립할 가능성을 시사한다.</mark>

# 관련 개념
- [[Edge_Controlled_Anomaly_Synthesis]] — 이 논문의 핵심 기여. 이미지 픽셀이 아니라 edge map을 중간 표현으로 조작(제거·교체·병합)한 뒤 edge-to-image generator로 사실적 이상 이미지를 생성하는 기법.

# 관련 문서
- 비교: (아직 없음 — 이 위키에서 anomaly detection은 [[ReContrast]]에 이어 두 번째 논문. ReContrast는 dual-encoder contrastive reconstruction으로 탐지 아키텍처 자체를 다루는 반면 LogicAL은 이상 합성(데이터 생성) 방법론이라 직접 비교축이 명확하지 않음. 세 번째 논문이 추가되면 비교 문서 신설 검토)

# 읽어볼 만한 논문
- 참고문헌 기반: P. Bergmann, K. Batzner, M. Fauser, D. Sattlegger, C. Steger, "Beyond dents and scratches: Logical constraints in unsupervised anomaly detection and localization" (Int. J. Comput. Vis. 2022) [4] — MVTecLOCO 데이터셋과 GCAD(global+local branch) 원조 논문. 논리적 이상이라는 문제 정의 자체가 시작된 곳으로 배경 이해에 필수.
- 참고문헌 기반: V. Zavrtanik, M. Kristan, D. Skocaj, "DRAEM - A discriminatively trained reconstruction embedding for surface anomaly detection" (ICCV 2021) [31] — 이 논문의 anomaly localization 네트워크가 계보를 잇는 재구성 기반 탐지 원조. 합성 이상의 품질이 성능에 미치는 영향을 처음 체계적으로 보인 연구.
- 참고문헌 기반: Y. Vinker, E. Horwitz, N. Zabari, Y. Hoshen, "Image shape manipulation from a single augmented training sample" (DeepSIM, ICCV 2021) [24] — 이 논문의 TPS 증강 전략이 직접 따르는 원조. Edge-to-image generator가 out-of-distribution 입력에서도 안정적으로 동작하게 만드는 핵심 배경 기법.
- 자유 추천(검증 필요): Object-agnostic 중간 표현(edge, depth, semantic map 등)을 활용한 다른 이상 합성/데이터 증강 연구 — 검색 키워드: `intermediate representation edge depth semantic anomaly synthesis data augmentation 2025`. LogicAL의 edge 기반 접근이 다른 중간 표현(예: depth map)으로 확장된 후속 연구가 있는지 확인할 가치가 있음.

---
**보안 참고**: PDF 전체를 확인했으며, 프롬프트 인젝션이나 지시문처럼 보이는 텍스트는 발견되지 않았다.
