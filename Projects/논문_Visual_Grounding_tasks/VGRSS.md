---
pm-task: true
projectId: "paperwiki-visual-grounding"
parentId:
id: "t-vgrss-q83oxmdh87"
title: "VGRSS: Datasets and Models for Visual Grounding in Remote Sensing Ship Images"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-08-24"
due:
progress: 0
assignees: []
tags: []
customFields:
  "bm1wp6i4mtck1e92": 2025
  "xy2qmm1smtck1e93": "IEEE Transactions on Geoscience and Remote Sensing (TGRS)"
subtaskIds: []
dependencies: []
year: 2025
venue: "IEEE Transactions on Geoscience and Remote Sensing (TGRS)"
jcr_quartile: Q1
task: [visual-grounding]
direction: [novel-approach, foundational]
paper_tags: [paper, visual-grounding, remote-sensing, ship-detection, multimodal, transformer, dataset, sar]
source: "/Users/GyeongSeo/Workspace/논문_pdf/Visual_Grounding/2025_TGRS_VGRSS.pdf"
createdAt: "2026-08-24T03:48:00.000Z"
updatedAt: "2026-08-24T03:48:00.000Z"
---

Project: [[논문 읽기|논문 읽기]]
#paper #visual-grounding #remote-sensing #ship-detection #multimodal #transformer #dataset #sar

# 한 줄 요약
<mark style="background: #FFF3A3A6;">원격탐사 선박 영상에서 자연어 표현으로 특정 선박을 위치시키는 새로운 과제 VGRSS(Visual Grounding for Remote Sensing Ship images)를 정의하고, 자동화된 속성 추출+템플릿 기반 표현 생성으로 두 대규모 벤치마크(광학 RSSVG, SAR SARVG)를 구축한 뒤, 언어 정보로 시각 feature를 융합 이전에 미리 강화하는 LVFE 모듈과 차원 압축 없이 시각-언어 feature를 쌓아 융합하는 VLF 모듈, EIoU 손실을 결합한 Transformer 기반 모델로 자연 이미지 SOTA VG 방법들을 세 데이터셋 모두에서 능가한 논문.</mark>

# 문제 정의

### 기존 방법의 한계
- **원격탐사 선박 영상 특유의 어려움에 대한 데이터 부재**:
  Visual grounding(VG) 연구는 자연 이미지에서 상당히 발전했지만, 원격탐사 도메인 특히 선박(ship) 영상에 특화된 대규모 데이터셋이 없다. 선박 영상은 복잡한 배경, 소형 타겟, 복잡한 공간 관계를 특징으로 하며, 기존 원격탐사 VG 연구(예: RSVG)조차 더 큰 타겟(경기장, 대형 건물 등)에 집중해 선박처럼 작고 클래스 간 유사도가 높은 객체에는 성능이 제한적이다.
- **모달리티 상호작용이 융합 모듈에만 국한**:
  기존 VG 방법 대부분은 시각-언어 상호작용을 fusion 모듈에서만 수행해, 텍스트 정보를 충분히 활용하지 못한다 — 언어 정보가 시각 feature 추출 과정 자체에 관여하지 않는다.
- **융합 시 공간 정보 손실**:
  전통적 방법들은 융합 시 시각 feature의 차원을 압축하는데, 이 과정에서 객체 위치 판별에 중요한 공간 정보가 손실될 수 있다.
- **자연 이미지 VG 방법의 원격탐사 전이 실패**:
  Transformer 기반 방법이 다양한 비전 과제를 지배해왔지만, 자연 이미지에서 개발된 VG 방법을 원격탐사 이미지에 그대로 적용하면 만족스럽지 못한 결과만 얻는다 — 도메인 갭이 존재.

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — One-stage 자연 이미지 VG**
- FAOA [27]: YOLOv3에 텍스트 임베딩을 통합, 여러 해상도에서 시각·텍스트·공간 feature 융합 — feature 융합 방식의 한계로 성능 불충분.
- LBYL-Net [28]: landmark feature convolution module로 객체 간 상대적 공간 관계를 전역 문맥으로 포착 — 복잡한 장면에서 성능 저하.
- 기타(Yang et al. [26] ReSC, Huang et al. [28], Liao et al. [29] 등): 쿼리를 동적으로 분해하거나 단계별 언어 가이드 feature 학습 도입.
- 공통 한계: 자연 이미지 설계라 원격탐사 도메인에 그대로 적용 시 성능 열세.

**갈래 2 — Two-stage VG**
- 사전학습 detector(예: Mask R-CNN)로 region proposal 생성 후 언어 쿼리로 최적 영역 선택·정제.
- MAttNet [32]: 주체·위치·관계형 속성을 모듈화해 grounding 정확도 향상.
- 공통 한계: 자연 이미지에 사전학습된 detector의 시각 feature가 원격탐사 도메인과 호환되지 않을 수 있어, pregenerated proposal의 품질이 성능 병목이 됨.

**갈래 3 — Transformer 기반 VG**
- TransVG [41]: BERT+visual Transformer+multimodal fusion Transformer로 구성된 최초의 end-to-end Transformer VG 네트워크.
- VLTVG [44]: language-guided visual feature aggregation+multilevel cross-modal decoder — 텍스트 설명과 관련된 영역에 focus를 강화하지만 multi-granularity modal 정보를 간과해 성능이 불충분.
- QRNet [45]: language query-aware dynamic attention 메커니즘.
- 공통 한계: 대부분 자연 이미지용으로 설계되어 원격탐사 이미지에는 부적합.

**갈래 4 — 원격탐사 특화 VG**
- Sun et al. [5](GeoVG, RSVG 데이터셋): 지리공간 관계 학습과 적응적 영역 attention으로 정확도 개선 — 그러나 어선·소형 여객선 같은 소형 원격탐사 타겟의 인식 정확도는 여전히 제한적.
- Zhan et al. [4](MGVLF, DIOR-RSVG 데이터셋): multi-granularity textual embedding으로 스케일 변화·복잡 배경 대응 — 대형 원격탐사 타겟(경기장, 대형 건물)에 주로 최적화되어 선박 같은 소형 타겟에는 성능이 제한적.

**갭**: <mark style="background: #FFF3A3A6;">기존 원격탐사 VG 연구는 대형 타겟 위주로 설계되어 선박처럼 작고 클래스 간 유사도가 높으며 클래스 내 다양성이 큰 객체를 다루는 대규모 벤치마크·모델이 없었다. 또한 기존 방법들은 언어 정보를 융합 단계에서만 활용하고 융합 시 공간 정보를 압축해, 소형 타겟의 정밀한 위치 판별에 불리했다.</mark>

### 이 논문이 풀고자 하는 문제
1. 원격탐사 선박 영상에 특화된 대규모 visual grounding 벤치마크(광학+SAR)를 구축하는 것
2. 언어 정보를 융합 이전 단계부터 시각 feature 강화에 활용하는 것
3. 융합 시 시각 feature의 공간 정보를 압축 없이 보존하는 것
4. 위 설계로 소형·클래스 간 유사도가 높은 선박 타겟에 대한 grounding 정확도를 개선하는 것

# 제안 방법

<mark style="background: #FFF3A3A6;">CNN backbone과 BERT로 시각·언어 feature를 각각 추출한 뒤, LVFE(Language-guided Visual Feature Enhancement) 모듈이 multihead self-attention으로 언어 정보를 시각 feature에 미리 주입해 강화하고, 이 강화된 시각 feature를 원본 언어 feature와 채널 방향으로 concat(차원 압축 없이 공간 정보 보존)한 뒤, VLF(Visual-Linguistic Fusion) 모듈이 학습 가능한 토큰을 쌓아 이 결합 feature 위에서 multihead self-attention으로 반복 정제해 최종 bounding box를 회귀한다.</mark>

### ① 자동화된 표현 생성으로 RSSVG/SARVG 데이터셋 구축
- 기존 4개 detection 데이터셋(FAIR1M, CGWX, SAR-Ship-Dataset, DIOR-RSVG)의 선박 섹션을 기반으로, (1) 데이터 필터링(비선박 카테고리 제거, 선박 비율 5% 미만 이미지 제거, 극소 영역 제거, 오류 라벨 제거) → (2) 속성 추출(크기, 절대/상대 위치, 상대 크기, 색상, 기타 속성을 픽셀 면적·HSV 색공간·좌표 기반으로 자동 산출) → (3) 표현 생성(고유성 검사로 각 선박을 식별하는 최소 속성 집합 선택, 템플릿 기반 문장 생성) 3단계 파이프라인으로 구성.
- RSSVG(광학, FAIR1M/CGWX/DIOR-RSVG 유래): 25,237쌍, 11,157장. SARVG(SAR, SAR-Ship-Dataset 유래): 54,429쌍, 43,798장.

<mark style="background: #FFF9D6A6;">"문제 정의"의 첫 번째 문제(선박 특화 대규모 벤치마크 부재)를, 사람이 일일이 라벨링하는 대신 기존 detection 데이터셋의 박스 정보로부터 속성을 자동 계산하고 템플릿으로 표현을 생성함으로써 대규모로 해결한다 — 고유성 검사를 통해 생성된 표현이 실제로 해당 선박만을 유일하게 지칭하도록 보장하는 것이 데이터 품질의 핵심 장치.</mark>

### ② Language-guided Visual Feature Enhancement (LVFE)
- 고레벨 시각 임베딩 `F_v`(256×400)와 고레벨 언어 임베딩 `F_t`(768×20, BERT 출력)를 선형 투영으로 동일 차원 공간(256×20)에 정렬.
- `F_v`를 query로, `F_t`를 key·value로 하는 multihead self-attention을 3회 반복 적용해, 매번 residual 연결로 원본 `F_v`에 업데이트를 누적 — 언어 정보가 점진적으로 시각 feature에 스며들도록 설계.
- 언어로 강화된 시각 feature `F_v`(256×400)를 얻은 뒤, 원본 언어 feature `F_t`를 확장(차원 변환·복제·재구성)해 채널 방향으로 concat, 최종 결합 feature `F_vt`(512×400) 생성.

> [!example]- 구현 디테일
> ```
> q = Linear(Fv);  k,v = Linear(Ft)                    # 채널 차원 정렬 후
> Fv' = MHAttn(q, k, v);  Fv = Fv + Linear(Fv')          # residual, 3회 반복
> Fvt = Concat(Fv, Expand(Ft))                           # 채널 방향 결합, 512×400
> ```
> LVFE 레이어 수 ablation(Table III): 3층→4층 시 정확도 83.15%→84.20%(+1.05%p), 5층에서는 83.47%로 소폭 하락 — 4층이 sweet spot.

<mark style="background: #FFF9D6A6;">"문제 정의"의 두 번째 문제(언어 정보가 융합 단계에서만 활용됨)를, 융합 이전에 이미 언어 가이드로 시각 feature 자체를 사전 강화함으로써 해결한다 — Table II ablation에서 LVFE 단독 추가만으로 정확도 57.12%→59.27%(+2.15%p)가 개선되어, "융합 전 언어 가이드 강화"라는 설계가 실제로 유효함을 확인.</mark>

### ③ Visual-Linguistic Fusion (VLF)
- 무작위 초기화된 학습 가능한 토큰 `T_q`(512×1)를 query로, 결합 feature `F_vt`를 key·value로 하는 multihead self-attention을 반복(3층 stack, residual 연결) 수행 — `T_q`가 `F_vt`의 모든 위치와 연결을 형성하며 시각·언어 문맥 정보를 점진적으로 집약.
- 최종 `T_q`는 시각·언어 문맥의 통합 표현이 되어, MLP 기반 prediction head(2-layer, ReLU+linear)에 입력되어 4D bounding box 좌표를 직접 회귀.

<mark style="background: #FFF9D6A6;">"문제 정의"의 세 번째 문제(융합 시 공간 정보 압축 손실)를, 전통적 방법처럼 시각 feature 차원을 줄이는 대신 채널 방향으로 쌓아(stacking) 공간 차원을 그대로 유지한 채 self-attention으로 융합함으로써 해결한다 — Table II에서 VLF 단독 추가는 +3.57%p로 LVFE 단독(+2.15%p)보다 더 큰 개별 기여를 보여, 공간 정보 보존이 특히 소형 선박 위치 판별에 중요함을 시사.</mark>

### 손실 함수: Enhanced IoU (EIoU)
- 기존 smooth L1+GIoU 손실에 더해, 예측-GT 박스의 대각선·너비·높이 차이까지 반영하는 EIoU 항을 추가.

> [!example]- 구현 디테일
> ```
> L_EIoU = IoU(b,b̂) - (ρ²/c² + ρw²/cw² + ρh²/ch²)
> L = L_smooth-L1 + λ·L_GIoU + λ·L_EIoU     (λ=1)
> ```
> Ablation(Table II)에서 EIoU 단독 추가는 +4.74%p — 세 손실·모듈 중 개별 기여가 가장 큼. 다만 LVFE와 EIoU를 함께 쓰면 오히려 -1.35%p(성능 저하)로, 모듈 간 상호작용에 따라 단순 가산적이지 않은 비선형적 효과가 관찰됨.

# 실험 결과

### 핵심 결과 (RSSVG/SARVG/DIOR-RSVG test, 기존 최고 SOTA 대비)
| 벤치마크 | 지표 | Before(RSVG, 이전 SOTA) | After(VGRSS) |
|---|---|---|---|
| RSSVG | test-accu / mIoU | 57.12(TransVG) / 51.08 | 66.16(+9.04%p) / 56.91 |
| SARVG | test-accu / mIoU | 95.82(VLTVG) / 82.57 | 95.94(+0.12%p) / 83.51 |
| DIOR-RSVG | test-accu / mIoU | 76.78(RSVG) / 68.04 | 83.01(+6.23%p) / 74.85 |

> [!note]- 세부 결과 및 Ablation
> #### 전체 SOTA 비교 (Table I)
> | 방법 | Venue | RSSVG val/test/mIoU | SARVG val/test/mIoU | DIOR-RSVG val/test/mIoU |
> |---|---|---|---|---|
> | FAOA | ICCV'19 | 31.85/31.07/27.44 | 91.64/90.91/73.77 | 51.23/49.39/40.65 |
> | ReSC | ECCV'20 | 52.81/55.95/47.63 | 93.47/93.75/75.20 | 70.72/69.73/57.31 |
> | LBYL-Net | CVPR'21 | - | - | 73.29/-/65.86 |
> | TransVG | ICCV'21 | 57.31/57.12/51.08 | 95.67/95.82/82.57 | 76.78/75.19/65.81 |
> | VLTVG | CVPR'22 | - | - | 69.41/-/59.96 |
> | RSVG | TGRS'23 | - | - | 76.78/75.19/68.04 |
> | **Ours(VGRSS)** | - | **66.28/66.16/56.91** | **96.53/95.94/83.51** | **83.68/83.01/74.85** |
> - RSSVG(선박, 광학)에서 가장 큰 개선폭 — val-accu +8.97%p, test-accu +9.04%p, mIoU +5.83%p. 선박이라는 소형·유사 클래스 타겟에서 이 방법의 강점이 가장 두드러짐.
> - SARVG는 이미 baseline 성능이 높아(95%대) 개선폭이 상대적으로 작음(+0.12%p test-accu) — SAR 이미지 자체의 특성(단순한 형태, 명확한 대비)상 여지가 적었을 가능성.
>
> #### 메인 ablation (Table II, RSSVG test)
> | LVFE | VLF | EIoU | Accuracy | Change |
> |---|---|---|---|---|
> | - | - | - | 57.12 | - |
> | ✓ | - | - | 59.27 | +2.15 |
> | - | ✓ | - | 60.69 | +3.57 |
> | - | - | ✓ | 61.86 | +4.74 |
> | ✓ | ✓ | - | 64.73 | +7.61 |
> | ✓ | - | ✓ | 55.77 | −1.35 |
> | - | ✓ | ✓ | 59.45 | +2.33 |
> | ✓ | ✓ | ✓(전체) | **66.16** | **+9.04** |
> - LVFE+EIoU 조합만 유일하게 baseline보다 낮음(음의 상호작용) — VLF 없이 LVFE와 EIoU만 결합하면 오히려 방해가 됨을 시사, 세 모듈 모두 함께 있어야 최대 시너지.
>
> #### LVFE/VLF 레이어 수 ablation (Table III, DIOR-RSVG)
> LVFE 3→4층: 83.15→84.20(+1.05%p, 최적) → 5층: 83.47(소폭 하락). VLF 3층이 최적(84.20%), 4층 이상은 정체·소폭 하락 — 두 모듈 모두 과도한 깊이는 오히려 역효과.
>
> #### 정성 분석 (Fig. 6)
> LVFE 적용 전후 attention heatmap 비교 — LVFE 적용 시 "오른쪽 위 선박", "왼쪽 아래 중형 정부 선박" 같은 위치·크기 속성이 명시된 쿼리에서 해당 영역에 attention이 뚜렷하게 집중되는 반면, 미적용 시 attention이 분산됨.

# Discussion

### 이 아이디어의 잠재적 부작용
- LVFE와 EIoU를 함께 쓰되 VLF가 빠지면 성능이 baseline보다 낮아짐(Table II, -1.35%p) → <mark style="background: #FF5582A6;">논문은 이 음의 상호작용의 원인을 분석하지 않는다 — LVFE가 만든 강화된 feature 표현이 VLF의 공간 보존 융합 없이 EIoU의 정밀한 기하 손실과 결합될 때 왜 학습이 오히려 방해받는지 설명이 없다.</mark>
- LVFE 레이어·VLF 레이어 모두 특정 깊이(4층/3층) 이후 성능이 정체·하락 → <mark style="background: #FF5582A6;">과도한 깊이가 왜 역효과를 내는지(과적합, 정보 희석 등) 메커니즘적 설명이 제시되지 않는다.</mark>

### 한계
- <mark style="background: #FF5582A6;">저자가 Conclusion에서 직접 명시: "향후 연구는 원격탐사 선박 영상의 특성에 더 잘 맞도록 VGRSS를 더 정교화하는 데 초점을 맞출 것" — 현재 버전이 아직 완성형이 아니라는 것을 인정.</mark>
- <mark style="background: #FF5582A6;">데이터셋 구축이 완전히 자동화되어 있어(사람 검수 없는 템플릿 기반 생성), 생성된 표현의 자연어다움(naturalness)이나 실제 사용자 질의와의 유사성이 검증되지 않는다 — RefCOCO류처럼 사람이 직접 작성한 표현과 비교했을 때의 표현 다양성·품질 차이가 논의되지 않음.</mark>
- Two-stage 방법(사전학습 detector 기반)과의 비교가 Table I에 포함되지 않아, One-stage/Transformer 계열에서만 비교가 이뤄짐 — 저자도 "기존 detector의 원격탐사 도메인 비호환성"을 two-stage의 한계로 지적하지만 실제 정량 비교는 제시하지 않는다.
- SARVG에서의 개선폭이 매우 작아(test-accu +0.12%p), 제안 모듈이 SAR 이미지라는 다른 센서 도메인에서는 상대적으로 기여가 제한적일 가능성이 있으나 이에 대한 원인 분석은 없다.

### 생각할 점
- <mark style="background: #A6E3A1A6;">이 논문은 이 위키에서 완전히 새로운 task(visual grounding)를 여는 첫 사례로, "자연어로 특정 객체를 지목한다"는 문제는 이 위키의 다른 small object detection 논문들이 다루는 "모든 객체를 빠짐없이 찾는다"는 문제와 근본적으로 다른 목표(특정성 vs 완전성)를 가진다 — 그럼에도 "소형·유사 클래스 객체를 어떻게 구별할 것인가"라는 하위 문제는 공유한다.</mark>
- <mark style="background: #A6E3A1A6;">LVFE의 "언어로 시각 feature를 융합 이전에 미리 강화한다"는 설계는, 이 위키의 dynamic query DETR 계열이 다루는 "density map으로 encoder feature를 미리 강화한다"(DQ-DETR의 CGFE 등)는 패턴과 구조적으로 유사하다 — 신호의 출처(언어 vs 밀도 맵)만 다를 뿐 "정제 대상이 되는 feature를 미리 보강해둔다"는 상위 전략은 동일하다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">VGRSS의 자동화된 표현 생성 파이프라인(속성 자동 추출+템플릿 기반 문장 생성)은, 이 위키의 [[YOFOR]]가 다루는 long-tailed 클래스 불균형 문제와 결합할 여지가 있다 — 희소 클래스 선박에 대해 더 다양한 표현 템플릿을 생성해 데이터 증강 효과까지 노릴 수 있을 것으로 보인다.</mark>
- <mark style="background: #A6E3A1A6;">EIoU 손실(대각선+너비+높이 차이를 명시적으로 반영)은 이 위키의 [[Density-Aware-DETR]]이 쓰는 anchor L1(log-ratio) 손실과 마찬가지로 "표준 IoU/L1이 소형 객체에서 충분한 gradient를 주지 못한다"는 문제의식을 공유한다 — 두 손실을 직접 비교하면 소형 객체 박스 회귀에 어떤 기하학적 보정이 가장 효과적인지 밝힐 수 있을 것으로 보인다.</mark>

# 관련 개념
- [[Language_Guided_Pre_Fusion_Feature_Enhancement]] — 이 논문의 LVFE 핵심 기여. 언어 정보로 시각 feature를 융합 이전 단계에서 미리 강화하는 기법.

# 관련 문서
- 비교: (아직 없음 — 이 위키에서 visual grounding을 다룬 첫 논문이라 비교 대상이 없음)

# 읽어볼 만한 논문
- 참고문헌 기반: Y. Zhan, Z. Xiong, Y. Yuan, "RSVG: Exploring data and models for visual grounding on remote sensing data" (IEEE Trans. Geosci. Remote Sens. 2023) [4] — DIOR-RSVG 데이터셋과 MGVLF 모델의 원조. 이 논문의 세 벤치마크 중 하나(DIOR-RSVG)의 출처이자, 원격탐사 VG 분야의 직접적인 선행 연구로 우선순위가 매우 높음.
- 참고문헌 기반: Z. Deng, Y. Yang, T. Chen, Z. Zhou, H. Li, "TransVG: End-to-end visual grounding with transformers" (ICCV 2021) [41] — 이 논문의 모델이 시각-언어 Transformer 구조 전체를 계승하는 원조 end-to-end Transformer VG 네트워크.
- 참고문헌 기반: L. Yang, Y. Xu, C. Yuan, W. Liu, B. Li, W. Hu, "Improving visual grounding with visual-linguistic verification and iterative reasoning" (VLTVG, CVPR 2022) [44] — 이 논문이 language-guided visual feature aggregation과 multilevel cross-modal decoder를 직접 비교하는 강력한 Transformer 기반 baseline.
- 자유 추천(검증 필요): SAR과 광학 두 센서 도메인을 함께 다루는 멀티모달 원격탐사 visual grounding/retrieval 후속 연구 — 검색 키워드: `SAR optical multimodal visual grounding remote sensing cross-sensor 2025 2026`. SARVG에서의 개선폭이 작았던 이유를 다른 SAR 특화 연구와 비교해 이해하는 데 도움될 것으로 예상.

---
**보안 참고**: PDF 전체를 확인했으며, 프롬프트 인젝션이나 지시문처럼 보이는 텍스트는 발견되지 않았다.
