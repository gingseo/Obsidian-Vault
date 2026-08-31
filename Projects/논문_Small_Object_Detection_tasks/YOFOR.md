---
pm-task: true
projectId: "paperwiki-small-object-detection"
parentId:
id: "t-yofor-3b9q73o0pb"
title: "YOFOR : You only focus on object regions for tiny object detection in aerial images"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-08-24"
due:
progress: 0
assignees: []
tags: []
customFields:
  "7l8l795xmtcjvlf6": 2026
  "1frf59rymtcjvske": "Neural Networks (Elsevier)"
subtaskIds: []
dependencies: []
year: 2026
venue: "Neural Networks (Elsevier)"
jcr_quartile: Q1
task: [small-object-detection]
direction: [improvement]
paper_tags: [paper, small-object-detection, tiny-object-detection, aerial-image, coarse-to-fine, long-tailed-detection, plug-in, unsupervised]
source: "/Users/GyeongSeo/Workspace/논문_pdf/Small_Object_Detection/2026_Neural-Networks_YOFOR.pdf"
createdAt: "2026-08-24T03:24:00.000Z"
updatedAt: "2026-08-24T03:24:00.000Z"
---

Project: [[논문 읽기|논문 읽기]]
#paper #small-object-detection #tiny-object-detection #aerial-image #coarse-to-fine #long-tailed-detection #plug-in #unsupervised

# 한 줄 요약
<mark style="background: #FFF3A3A6;">Coarse detection 결과를 클러스터링해 객체 밀집 영역만 적응적으로 크롭하는 비지도 Adaptive Local Sensing Module(ALSM), recursive Gaussian filter로 서브영역 배경을 흐리게 해 객체 대비를 강화하는 Fuzzy Enhancement Module(FEM), 데이터셋의 long-tailed 클래스 분포를 분석해 소수 클래스 객체를 좌우로 조건부 복제하는 Class Balance Module(CBM) 세 비지도 모듈을 기존 detector에 plug-in으로 결합한 YOFOR — VisDrone/DOTA/AI-TOD 세 항공 벤치마크에서 YOLOv8s baseline 대비 AP를 일관되게 크게 개선(VisDrone +34.1%p, DOTA +24.5%p, AI-TOD +48.4%p).</mark>

# 문제 정의

### 기존 방법의 한계
- **Scale·분포 불균형**:
  항공 이미지는 해상도가 매우 높고 객체는 이미지 전체 면적의 약 25%에만 밀집(나머지 75%는 배경) 분포하는데, 원본 이미지를 그대로 detector에 넣으면 비효율적이고 정확도도 낮다.
- **균일 crop/tiling의 한계**:
  단순 tiling(균일 분할)이나 random crop은 객체 밀집 영역을 정확히 잡아내지 못해, crop된 서브이미지에도 여전히 많은 배경이 섞이고 복잡한 배경이 탐지를 방해한다. GLASN 같은 super-segmentation 기반 개선도 배경까지 함께 초해상도 처리해 배경 간섭을 오히려 늘릴 수 있고, 코너의 객체를 놓치기 쉽다.
- **Coarse-to-fine 파이프라인의 연산 비효율과 오탐**:
  Coarse detector가 제안한 서브영역은 대개 배경 비중이 크고 경계가 거칠어, 연산 낭비뿐 아니라 fine detector 단계에서 오탐(false alarm)·미검출(missed detection)을 유발한다.
- **Long-tailed 클래스 불균형**:
  VisDrone·DOTA·AI-TOD 같은 공개 항공 벤치마크는 클래스 간 샘플 수 불균형이 심각하다(예: VisDrone에서 pedestrian/people/car 3개 클래스가 전체 객체의 70% 이상). 기존 방법(리샘플링, reweighting, 별도 head 분리)은 각각 과적합, 최적화 난이도, 모델 복잡도 증가라는 부작용을 동반한다.

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Coarse-to-fine 서브영역 탐지**
- ClusDet(Yang et al., 2019a): 클러스터링 기반 균일 서브영역 생성, end-to-end 네트워크.
- DMNet(Li et al., 2020a): 밀도 맵으로 원본을 서브영역으로 분리.
- GLASN(Deng et al., 2020): super-segmentation 기반 서브영역 초해상도 처리 — 배경까지 함께 초해상도 처리되어 오히려 간섭이 늘어남.
- CDMNet, AMRNet, UFPMP-Det, CZDet, CRENet, AdaZoom, Focus&Detect, YOLC 등: 유사한 coarse-to-fine 계열 다수 — 공통적으로 서브영역 경계가 거칠고 배경 비중이 크다는 문제는 근본적으로 해결하지 못함.

**갈래 2 — Tiny object detection의 loss/label assignment**
- GWD(Yang et al., 2021a), KLD(Yang et al., 2021b): Gaussian Wasserstein/KL divergence 기반 방향성 객체 손실.
- DotD(Xu et al., 2021a), NWD(Wang et al., 2021a): IoU의 위치 민감성을 완화하는 거리 기반 지표.
- SODA-D/SODA-A(Cheng et al., 2023): 대규모 소형 객체 벤치마크 구축.
- RaFPN(Zhou & Zhu, 2024), CFINet(Yuan et al., 2023): feature relationship 모델링·coarse-to-fine + imitation learning.
- 공통 한계: 이 갈래는 손실 함수·label assignment·feature 구조 개선에 집중하며, 원본 이미지의 "어디를 볼지"(coarse-to-fine 서브영역 품질) 문제나 long-tailed 문제는 다루지 않음.

**갈래 3 — Long-tailed object detection**
- 오버샘플링/언더샘플링(Buda et al., 2018; Byrd & Lipton, 2019): 과적합 또는 head class 정보 손실 위험.
- 클래스 분리(Zhang et al., 2019): 두 서브네트워크로 분리해 일반화 능력 저해.
- Reweighting(Cui et al., 2019): 손실 함수에서 tail class 가중치 증가 — 대규모 데이터셋에서 최적화 난이도 증가.
- 공통 한계: 대부분 일반 이미지 분류/탐지에서 개발된 기법을 그대로 적용, 항공 이미지 특유의 공간적 semantic(예: 자동차를 바다 위에 복제하면 안 됨)을 고려하지 않음.

**갭**: <mark style="background: #FFF3A3A6;">Coarse-to-fine 계열은 서브영역 localization의 정확도(배경 비중)를 다루지 않고, tiny object detection 계열은 loss/label assignment에 집중해 "어디를 볼지"는 다루지 않으며, long-tailed 계열은 항공 이미지의 공간적 semantic을 무시한 채 일반적인 리샘플링만 적용한다. 이 세 문제(scale/분포 불균형, 서브영역 품질, long-tailed)를 하나의 비지도 파이프라인에서 동시에 다루는 방법은 없었다.</mark>

### 이 논문이 풀고자 하는 문제
1. Coarse detection 결과로부터 객체가 실제로 밀집된 서브영역을 배경 비중을 최소화하며 적응적으로 localization하는 것
2. Localization된 서브영역의 배경 간섭을 억제해 객체 feature를 상대적으로 강화하는 것
3. Long-tailed 클래스 불균형을 항공 이미지의 공간적 semantic을 훼손하지 않으면서 완화하는 것
4. 위 세 가지를 별도 학습 없이(비지도) 기존 detector에 plug-in으로 결합 가능하게 만드는 것

# 제안 방법

<mark style="background: #FFF3A3A6;">원본 이미지에 대해 먼저 낮은 threshold로 coarse detection을 수행해 대략적인 객체 위치를 얻은 뒤, ALSM이 이 coarse box들을 IoU 기반 병합과 K-means 클러스터링으로 묶어 배경 비중이 최소화된 서브영역을 적응적으로 잘라내고, FEM이 그 서브영역에 recursive Gaussian filter로 배경을 흐리게 해 객체 대비를 강화하며, CBM이 tail 클래스 객체를 공간적 semantic을 고려해 좌우로 조건부 복제한다 — 세 모듈 모두 별도 학습 없이 기존 detector 출력 위에서 동작하는 비지도 후처리/전처리다.</mark>

### ① Adaptive Local Sensing Module (ALSM)
- 먼저 원본 데이터셋으로 학습된(낮은 threshold로 완화한) detector를 원본 이미지에 적용해 coarse box 집합을 얻음.
- **개별 박스 처리**: IoU>0가 되는 coarse box끼리 연결해 1차 서브영역을 형성, 이후 K-means로 재클러스터링, 형성된 서브영역은 일정 비율만큼 확대(코너·경계의 객체 누락 방지).
- **서브영역 재처리(Algorithm 1)**: 너무 많은 미세 서브영역이 생기는 것을 막기 위해, 단일 객체 박스 `b_i`를 기준으로 나머지 서브영역까지의 유클리드 거리를 계산해, `b_i`와 가장 가까운 서브영역 `b_r`까지의 거리가 `b_r` 내부 다른 박스 간 최대 거리 이상이면 병합, 아니면 병합하지 않음 — 이 과정을 모든 단일 박스에 대해 반복.
- 최종 서브영역이 지나치게 작으면 확대하거나 가장 가까운 서브영역과 병합.

<mark style="background: #FFF9D6A6;">"문제 정의"의 첫 번째 문제(균일 crop/tiling이 배경을 과도하게 포함)를, 원본 이미지 전체를 미리 보지 않고 coarse detection이 실제로 찾은 객체 위치만을 기준으로 클러스터링해 크롭 영역을 정함으로써 해결한다 — 배경이 75%인 항공 이미지에서 이 적응적 접근은 균일 분할 대비 서브영역당 배경 비중을 구조적으로 낮춘다.</mark>

### ② Fuzzy Enhancement Module (FEM)
- ALSM이 만든 서브영역에 recursive Gaussian filter(RGF, Young & van Vliet 1995)를 적용해 배경을 흐리게 만듦 — 표준 Gaussian 커널과 달리 윈도우 크기와 무관하게 이미지 크기에만 비례하는 연산 복잡도를 가져 고해상도 항공 이미지에서도 빠름.
- RGF는 다항식으로 Gaussian 함수를 근사한 뒤 s-domain·z-domain 변환을 거쳐, forward/backward 두 방향의 1차원 IIR(무한 임펄스 응답) 필터로 분해해 재귀적으로 적용.
- Fuzzy 처리된(배경이 흐려진) 이미지와 원본 서브영역의 coarse detection 결과를 결합해 최종 강화된 서브영역을 얻고, 이를 detector에 다시 입력해 정제된(refined) 검출 결과를 얻음.

> [!example]- 구현 디테일
> ```
> H_L(z) = A2 / (b0 - b1z^-1 - b2z^-2 - b3z^-3)   # forward filter
> H_R(z) = A2 / (b0 - b1z^1 - b2z^2 - b3z^3)       # backward filter
> w[n] = B·in[n] + (b1w[n-1]+b2w[n-2]+b3w[n-3])/b0   # forward pass
> out[n] = B·w[n] + (b1out[n+1]+b2out[n+2]+b3out[n+3])/b0  # backward pass
> B = 1 - (b1+b2+b3)/b0
> ```
> 필터 반복 횟수 ablation(Table 8): 0→18회까지 늘릴수록 AP 지속 개선(29.2→34.4)이나 개선폭은 점차 완만해짐 — 특정 수준 이상에서는 배경 억제 효과가 포화.

<mark style="background: #FFF9D6A6;">"문제 정의"의 세 번째 문제(coarse-to-fine 파이프라인의 오탐·미검출)를, 배경을 아예 없애는 대신 흐리게(fuzzy) 만들어 상대적으로 객체 영역의 대비를 높임으로써 완화한다 — GLASN류 초해상도 접근이 배경까지 함께 선명하게 만들어 간섭을 오히려 늘리는 것과 반대로, 이 모듈은 배경 정보를 의도적으로 저하시켜 detector의 주의를 객체로 유도한다.</mark>

### ③ Class Balance Module (CBM)
- 데이터셋 전체를 분석해 클래스별 객체 수 비율을 계산하고, 비율이 7% 미만인 클래스를 tail class로 식별(데이터셋마다 재분석 필요).
- Tail class 객체 `C`에 대해, 좌우 인접 객체도 tail class인지 확인 — 그런 경우에만 `C`를 좌우로 복제(이때 tail class 박스 너비의 1/4을 복제 거리로 사용).
- 이미지 경계를 벗어나는 경우 복제하지 않음.
- 무작위 위치에 복제하는 random copy-paste(RCP)와 달리, 원래 객체 주변의 공간적 문맥(방향·거리)을 그대로 유지한 채 근접 복제.

<mark style="background: #FFF9D6A6;">"문제 정의"의 네 번째 문제(long-tailed 클래스 불균형)를, "자동차를 바다 위에 복제"하는 식의 semantic 모순을 피하면서 해결한다 — 원 객체 주변에 이미 존재하는 문맥(도로 위 차량 근처에 복제된 차량도 도로 위)을 그대로 상속하므로, 무작위 위치 복제(RCP) 대비 semantic 위반 없이 tail class 샘플 수만 늘릴 수 있다는 것이 핵심 차별점(Table 10에서 CBM이 RCP를 일관되게 상회함으로써 뒷받침).</mark>

# 실험 결과

### 핵심 결과 (YOLOv8s baseline, 세 모듈 순차 적용, Table 2/3/4)
| 벤치마크 | 지표 | Before(YOLOv8s) | After(YOFOR, ALSM+FEM+CBM) |
|---|---|---|---|
| VisDrone val | AP | 19.1 | 52.8 (+33.7%p) |
| DOTA | AP | 29.2 | 53.7 (+24.5%p) |
| AI-TOD val | AP | 21.8 | 70.2 (+48.4%p) |

> [!note]- 세부 결과 및 Ablation
> #### 모듈별 순차 기여 — VisDrone(Table 2, YOLOv8s)
> | ALSM | FEM | CBM | AP | AP0.5 | AP_s | AP_m | AP_l |
> |---|---|---|---|---|---|---|---|
> | - | - | - | 19.1 | 32.4 | 11.3 | 29.0 | 40.7 |
> | ✓ | - | - | 38.1 | 58.8 | 27.7 | 48.3 | 61.2 |
> | ✓ | ✓ | - | 46.9 | 64.5 | 34.3 | 65.4 | 71.7(추정, 표에서 확인) |
> | ✓ | ✓ | ✓ | **52.8** | **69.1** | **43.8** | **65.1** | **79.8** |
> - ALSM 단독으로 AP가 19.1→38.1(약 2배)로 가장 큰 단일 기여 — 저자는 "coarse detection 결과를 국소적으로 재정제하는 것 자체"가 핵심 이득이라고 해석.
> - CBM 추가 시 AP_s가 34.3(추정)→43.8로 크게 개선 — tail class 중 소형 객체 비중이 높아 long-tailed 완화가 소형 객체 성능에도 직결됨을 시사.
>
> #### SOTA 비교 — VisDrone(Table 11, 발췌)
> | 방법 | AP | AP50 | AP75 | AP_s | AP_m | AP_l |
> |---|---|---|---|---|---|---|
> | ClusDet | 26.7 | 50.6 | 24.7 | 17.6 | 38.9 | 51.4 |
> | UFPMP-Det | 37.6 | 59.1 | 40.1 | 9.2 | 28.7 | – |
> | Focus&Detect | 42.0 | 66.1 | 44.6 | 32.0 | 47.9 | 54.5 |
> | YOLC | 39.6 | 63.7 | 41.6 | 32.3 | 13.3 | 21.8(원문 표기 그대로) |
> | Faster-RCNN* (YOFOR 적용) | 43.5 | 61.5 | 46.7 | 34.0 | 55.7 | 67.4 |
> | **YOFOR*(YOLOv5s 기반)** | **58.2** | **72.3** | **62.4** | **50.3** | **69.8** | **79.8** |
> | YOLOv10s*(YOFOR 적용) | 57.1 | 71.8 | 62.0 | 47.2 | 68.7 | 78.4 |
> - Faster-RCNN·YOLOv5s·YOLOv10s 세 가지 서로 다른 baseline 모두에 YOFOR를 적용해 일관된 대폭 개선을 확인 — 특정 detector에 국한되지 않는 범용성 주장의 근거.
>
> #### SOTA 비교 — AI-TOD(Table 12, anchor-free 발췌)
> | 방법 | AP | AP0.5 | AP_vt | AP_t |
> |---|---|---|---|---|
> | YOLOv8 | 24.5 | 55.0 | 17.9 | 27.6 |
> | FFCA-YOLO | 24.7 | 59.5 | 16.8 | 25.7 |
> | YOFOR | 31.6 | 58.1 | 30.4 | 9.1(원문 표기) |
> | **YOFOR*(적용 후)** | **31.6** | **58.1** | **30.4** | **29.9(추정)** |
> | Faster-RCNN* | 37.8 | 67.6 | 37.4 | 21.5 |
>
> #### ALSM vs 다른 클러스터링 방법(Table 5, VisDrone, Faster-RCNN)
> | 방법 | AP | AP0.5 | AP_s |
> |---|---|---|---|
> | 없음(baseline) | 18.1 | 30.7 | 9.6 |
> | HC(계층적 클러스터링) | 13.6 | 30.2 | 4.2(표기 그대로, 오히려 하락) |
> | SARSA(Deng et al., 2020) | 24.9 | 43.5 | 17.7 |
> | UFP(Huang et al., 2022) | 26.2 | 50.5 | 16.3 |
> | **ALSM(제안)** | **30.1** | **50.5** | **20.4** |
>
> #### FEM vs super-resolution(Table 6, VisDrone, Faster-RCNN+ALSM)
> ALSM만 30.1 AP → ALSM+SRN(초해상도) 30.6(+0.5, 미미) → ALSM+FEM(제안) **46.0(+15.9)** — 저자는 이를 "SRN은 저해상도 이미지 개선 시 배경 간섭도 함께 증폭시키는 반면 FEM은 배경만 선택적으로 억제하기 때문"이라고 해석.
>
> #### 필터 종류 비교(Table 7, DOTA)
> Gaussian/JunZhi/ZhongZhi/RGF 네 필터 모두 baseline 대비 개선(29.1~29.8 AP)되어 필터 종류에 무관하게 "배경을 흐리게 한다"는 아이디어 자체가 유효함을 뒷받침 — RGF는 그중에서도 창 크기 무관 속도 이점 때문에 채택.
>
> #### CBM 거리 파라미터(Table 9, AI-TOD, YOLOv8s)
> 복제 거리 0(복제 없음) 25.4 AP → 0.5배 49.0 → **1배(채택) 51.1** → 2배 46.3 — 1배가 sweet spot, 과도한 거리는 오히려 성능 저하.

# Discussion

### 이 아이디어의 잠재적 부작용
- ALSM의 coarse detection 자체가 실패하면(threshold를 낮춰도 객체를 아예 놓치면) 이후 파이프라인 전체가 그 영역을 볼 기회조차 얻지 못함 → <mark style="background: #FF5582A6;">논문은 coarse detector의 recall 실패가 파이프라인에 미치는 영향을 별도로 정량화하지 않는다 — "낮은 threshold로 완화"한다고만 서술할 뿐 그 threshold의 구체적 값이나 실패율은 보고되지 않는다.</mark>
- CBM의 tail class 판정 기준(비율 7% 미만)이 고정 임계값이라 데이터셋마다 재분석이 필요 → <mark style="background: #FF5582A6;">저자도 "tail 카테고리 식별은 데이터셋별로 구체적으로 분석되어야 한다"고 명시 — 완전 자동화된 것이 아니라 데이터셋 전환 시 수작업 재설정이 필요함을 인정.</mark>

### 한계
- <mark style="background: #FF5582A6;">저자가 Conclusion에서 명시: 현재는 원본 이미지·서브영역 레벨에서 enhancement를 적용하는데, 향후 feature map 레벨에서 직접 강화하는 방향을 향후 과제로 남김 — 현재 설계가 feature 수준 최적화까지는 이르지 못했다는 것을 스스로 인정.</mark>
- <mark style="background: #FF5582A6;">Table 11/12에서 "YOFOR"과 "YOFOR*(적용 후)"라는 두 표기가 혼재하고 AP_t 등 일부 수치가 본문 서술과 완전히 일치하지 않는 부분이 있어(예: Table 12의 YOFOR AP_t 9.1 vs Table 4의 AI-TOD 결과와의 정합성), 표 해석에 다소 모호함이 남는다 — 원문 표를 직접 재확인할 필요.</mark>
- ALSM의 K-means 클러스터링에서 클러스터 수 K를 어떻게 정하는지 구체적 기준(고정값인지 적응적인지)이 본문에 명확히 서술되지 않음.
- CBM은 "좌우"로만 복제하며 상하 방향은 다루지 않는데, 이 방향 선택의 근거(항공뷰 특유의 도로/구조물 배치 방향성 가정 등)가 명시적으로 설명되지 않는다.

### 생각할 점
- <mark style="background: #A6E3A1A6;">FEM이 "배경을 지우는 대신 흐리게 만든다"는 선택은, 이 위키의 [[SR-TOD]]·[[ORFENet]]이 "정보 손실이 큰 영역(=객체 가능성이 높은 영역)을 강조"하는 것과 반대 방향에서 같은 목표(객체-배경 대비 강화)에 도달한다 — 강조(highlight) 대신 억제(suppress)라는 전략의 차이가 실제로 어느 쪽이 더 안정적인지 두 접근을 직접 비교하면 흥미로울 것으로 보인다.</mark>
- <mark style="background: #A6E3A1A6;">CBM의 "공간적 semantic을 보존하는 복제"라는 아이디어는 이 위키에서 long-tailed 문제를 다룬 첫 사례로, [[DQ-DETR]] 등 dynamic query DETR 계열이 다루는 "밀도 불균형"(이미지 간 인스턴스 수 차이)과는 다른 층위의 불균형(클래스 간 샘플 수 차이)을 다룬다는 점에서 상호 보완적이다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">ALSM의 coarse-to-fine 서브영역 localization은 이 위키의 [[QueryDet]](feature pyramid 레벨 간 sparse query)·[[FFSSTDNet]](patch 단위 CFD 필터링)과 "어디를 볼지 좁힌다"는 문제의식을 공유하지만, 이 논문은 픽셀/이미지 공간에서 직접 클러스터링한다는 점에서 구현 층위가 다르다 — 세 방법의 연산 비용·recall 손실 트레이드오프를 정량 비교하면 어떤 층위(feature vs 이미지)에서 가속하는 것이 더 유리한지 밝힐 수 있을 것으로 보인다.</mark>
- <mark style="background: #A6E3A1A6;">CBM처럼 항공 이미지의 공간적 semantic을 보존하는 augmentation 전략을, 이 위키의 dynamic query DETR 계열이 다루는 "이미지 간 밀도 불균형" 문제에도 적용해 볼 수 있다 — 희소 이미지에 tail class 객체를 semantic 보존 복제로 추가하면, query 배정의 밀도 추정 정확도(예: [[DQP-DETR]]의 ADPG)에도 도움이 될 가능성이 있다.</mark>

# 관련 개념
- [[Class_Balanced_Spatial_Copy_Paste]] — 이 논문의 CBM이 제안한 핵심 기여. Tail class 객체를 원 객체 주변의 공간적 문맥(방향·거리)을 보존한 채 조건부로 좌우 복제해, semantic 모순 없이 long-tailed 클래스 불균형을 완화하는 기법.

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]] — 기존 비교표의 어느 축에도 완전히 속하지 않는 독특한 위치(coarse-to-fine 서브영역 localization + 배경 억제 + long-tailed 완화 3종 결합). 다음 comparison 갱신 시 별도 행으로 추가 검토.

# 읽어볼 만한 논문
- 참고문헌 기반: C. Yang, Z. Huang, N. Wang, "QueryDet: Cascaded sparse query for accelerating high-resolution small object detection" (CVPR 2022) — 이미 위키에 추가됨: [[QueryDet]]. "어디를 볼지 좁힌다"는 문제의식을 feature pyramid 레벨에서 다루는 대조군으로 비교 가치가 높음.
- 참고문헌 기반: F. Yang, H. Fan, P. Chu, E. Blasch, H. Ling, "Clustered object detection in aerial images" (ClusDet, ICCV 2019a) — ALSM이 계승·개선하는 클러스터링 기반 coarse-to-fine의 대표 선행 연구.
- 참고문헌 기반: I. T. Young, L. J. Van Vliet, "Recursive implementation of the Gaussian filter" (Signal Processing, 1995) — FEM이 채택한 RGF의 원조 논문. 윈도우 크기 무관 연산 복잡도의 수학적 배경 이해에 필수.
- 자유 추천(검증 필요): Feature map 레벨에서 직접 배경 억제/객체 강조를 수행하는 attention 기반 연구 — 검색 키워드: `feature-level background suppression attention aerial small object detection`. 저자가 Conclusion에서 향후 과제로 명시한 "feature map 레벨 enhancement" 방향과 직접 연결되는 검색.

---
**보안 참고**: PDF 전체를 확인했으며, 프롬프트 인젝션이나 지시문처럼 보이는 텍스트는 발견되지 않았다.
