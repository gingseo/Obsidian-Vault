---
pm-task: true
projectId: "paperwiki-small-object-detection"
parentId:
id: "t-ig-detr-mmgdtbr6lk"
title: "IG-DETR: Instance-Guided Dynamic Queries for Small Object Detection"
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
  "1frf59rymtcjvske": "ICASSP"
subtaskIds: []
dependencies: []
year: 2026
venue: "ICASSP"
jcr_quartile: Q2
task: [small-object-detection]
direction: [improvement]
paper_tags: [paper, small-object-detection, tiny-object-detection, detr, dynamic-query, feature-enhancement, remote-sensing]
source: "/Users/GyeongSeo/Workspace/논문_pdf/Small_Object_Detection/2026_ICASSP_IG-DETR.pdf"
createdAt: "2026-08-24T03:12:00.000Z"
updatedAt: "2026-08-24T03:12:00.000Z"
---

Project: [[논문 읽기|논문 읽기]]
#paper #small-object-detection #tiny-object-detection #detr #dynamic-query #feature-enhancement #remote-sensing

# 한 줄 요약
<mark style="background: #FFF3A3A6;">DQ-DETR의 4단계보다 세분화된 6단계 장면 난이도 분류(HIP)로 query 개수를 정하고, 곱셈 마스킹 대신 덧셈 residual 주입(additive residual injection)으로 encoder feature의 semantic backbone을 보존하면서 고주파 텍스처를 강화하는 IFE 모듈, 그리고 이 강화된 feature에서 "salient seed"를 top-K로 뽑아 query의 content·position을 동시에 생성하는 IGQ 모듈을 결합해, DINO 대비 AP +5.0%p·DQ-DETR 대비도 우위를 보인 짧은 형식(ICASSP)의 dynamic query DETR.</mark>

# 문제 정의

### 기존 방법의 한계
- **Quantity dilemma(수량 딜레마)**:
  Transformer 기반 detector는 고정된 query 수 K로 초기화되는데, 이는 감지 가능한 최대 객체 수의 상한이 된다. DINO 같은 강력한 baseline도 900 query로는 수천 개 밀집 인스턴스에 압도되어 대량의 false negative가 발생한다(Fig. 1c에서 시각적으로 확인).
- **Quality deficit(품질 결핍)**:
  초기 query는 대개 입력 이미지와 무관하게 학습된 임베딩이라, 명시적 공간 사전 정보(spatial prior)가 없어 넓은 feature map 안의 작고 밀집된 객체를 "건초더미에서 바늘 찾기"처럼 찾아야 한다. SAM 같은 foundation model이 개별 인스턴스를 낮은 수준에서 인지할 수 있음에도(Fig. 1b), 이 저수준 인지가 transformer detector의 성공적인 탐지로 이어지지 않는다는 점을 대조적으로 제시.

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — DETR 아키텍처/수렴 개선**
- Deformable DETR [2,3,4]: 초기 연산 한계를 해결하고 멀티스케일 feature 사용을 가능하게 함.
- DINO [7]: denoising anchor 개선으로 강력한 baseline 확립 — 그러나 여전히 고정 query 수(900).
- Anchor DETR [10], Conditional DETR [8], DAB-DETR [5], DN-DETR [6]: query 형식·수렴 속도 개선 — query "개수"는 다루지 않음.
- 공통 한계: query가 여전히 이미지와 무관한 static·content-agnostic 임베딩([17]~[23] 다수 인용).

**갈래 2 — Dynamic query 시도 (직접 baseline)**
- DQ-DETR [26]: density map 기반 4단계 이산 분류로 query 개수 조정 — 이 논문이 직접 능가 대상으로 삼는 baseline(Table 1에서 DINO와 함께 직접 비교).

**갭**: <mark style="background: #FFF3A3A6;">기존 DETR 계열 연구들은 static·content-agnostic query 메커니즘 자체를 항공 영상의 sparse-to-dense 딜레마의 근본 병목으로 명확히 지목하지 않았다. DQ-DETR이 이 문제에 처음 접근했지만, 이 논문은 (1) 더 세분화된 난이도 분류(4→6단계), (2) semantic backbone을 보존하는 additive feature enhancement, (3) salient seed 기반 query 생성이라는 세 가지 지점에서 추가 개선 여지를 발견했다.</mark>

### 이 논문이 풀고자 하는 문제
1. 장면 복잡도(밀도)를 세밀하게 분류해 query 개수를 이미지별로 적절히 배정하는 것(quantity dilemma 해소)
2. Query 초기화에 쓰일 encoder feature를 semantic 정보 손실 없이 고주파(tiny object) 정보로 보강하는 것(quality deficit 해소)
3. 위 두 요소를 결합해 query의 content와 position을 처음부터 실제 객체 위치에 가깝게 생성하는 것

# 제안 방법

<mark style="background: #FFF3A3A6;">고해상도 encoder feature로 장면 난이도를 6단계로 분류해 query 개수를 정하고(HIP), 이 난이도 정보를 spatial saliency prior로 변환해 곱셈 마스킹이 아닌 덧셈 residual 주입으로 encoder feature를 보강한 뒤(IFE), 보강된 feature에서 카테고리에 무관한 salient seed를 top-K로 뽑아 query의 content(semantic projection)와 position(coordinate 정제)을 함께 생성한다(IGQ).</mark>

### ① Hierarchical Instance Perception (HIP)
- 가장 고해상도인 encoder feature `F_enc^(1)`을 1×1 conv + 일련의 dilated convolution으로 처리해 instance perception feature map `F_inst` 생성(넓은 문맥 정보 확보).
- `F_inst`를 2-layer 분류 head에 통과시켜 이미지를 6단계 이산 난이도로 분류 — 각 단계는 사전 정의된 query 수(300, 500, 700, 900, ...)에 직접 대응.
- 회귀 대신 분류를 택한 근거는 DQ-DETR과 동일한 논리(AI-TOD-V2의 극단적 객체 수 편차, 1~2267)이지만 구간을 4→6개로 세분화.

<mark style="background: #FFF9D6A6;">"문제 정의"의 첫 번째 문제(quantity dilemma)를, DQ-DETR보다 더 세밀한 6단계 구간으로 query 수를 배정해 완화한다 — Table 4에서 고정 query 수를 300~1500까지 바꿔가며 비교한 결과 1200에서 정점(30.5)을 찍고 1500에서 오히려 하락하는데, 동적 방식(30.9)이 모든 고정 구성을 능가함을 실험으로 확인해, "세분화된 동적 배정"이 "가장 좋은 단일 고정값"보다도 우월함을 보인다.</mark>

### ② Instance-Aware Feature Enhancement (IFE)
- `F_inst`를 1×1 conv로 각 encoder 레벨 해상도에 맞게 다운샘플링해 multi-scale instance perception feature `F_inst^(i)` 생성.
- Average/max pooling → concat → 7×7 conv → sigmoid로 spatial saliency prior `W_sp^(i)` 산출(DQ-DETR의 CGFE와 동일한 spatial attention 계산식).
- **핵심 차별점**: 기존 방법들의 곱셈 마스킹(`F ⊗ W`) 대신, `(1+W_sp)` 형태의 **덧셈 residual 주입**을 사용 — semantic backbone 정보를 그대로 보존하면서 고주파 텍스처만 명시적으로 더함.
- 이어서 channel 차원에서도 동일한 dual-residual 방식(`(1+W_ch)`)으로 2차 보강.

> [!example]- 구현 디테일
> ```
> W_sp^(i) = σ(C7×7(Concat[AvgP(F_inst^(i)), MaxP(F_inst^(i))]))
> F_sp^(i) = F_enc^(i) ⊗ (1+W_sp^(i)) ≡ F_enc^(i) + F_enc^(i) ⊗ W_sp^(i)
> W_ch^(i) = σ(MLP(AvgP(F_sp^(i))) + MLP(MaxP(F_sp^(i))))
> F_tot^(i) = F_sp^(i) ⊗ (1+W_ch^(i))
> ```
> "Dual-residual 설계가 tiny object 정보 복원을 위한 gradient 흐름을 촉진한다"고 저자가 직접 명시 — 순수 곱셈 마스킹은 attention 값이 0에 가까울 때 원본 정보 자체가 소실되는 반면, `(1+W)` 형태는 최소 원본 정보(1배)를 항상 보존.

<mark style="background: #FFF9D6A6;">"문제 정의"의 두 번째 문제(quality deficit)를, DQ-DETR/Density-Aware DETR과 유사한 attention 계산식을 쓰되 결합 방식을 곱셈에서 덧셈 residual로 바꿔 해결한다 — 순수 곱셈 마스킹이 attention 가중치가 낮은 영역의 원본 semantic 정보까지 억제해버리는 위험을 원천 차단하며, 저자는 이것이 "tiny object 정보 복원을 위한 gradient 흐름"에 중요하다고 설명한다.</mark>

### ③ Instance-Guided Adaptive Query Prediction (IGQ)
- DAB-DETR 방식을 따라 query를 semantic content `Q_cont`와 geometric position `Q_pos`로 분리.
- HIP이 정한 query 수 K에 따라, IFE로 보강된 feature `F_tot`를 flatten한 `F_flat`을 scoring 함수 `Φ_score`에 통과시켜 category-agnostic한 confidence map `S`를 얻고, 단순 threshold가 아니라 top-K개의 "salient seed" 인덱스 `Ω_K`를 추출.
- 선택된 seed feature `F_Ω`로부터, semantic projection `P_sem`으로 `Q_cont`를, coarse grid 좌표를 기준점(`B_init`)으로 삼아 regressor `Ψ_reg`가 예측한 offset을 더한 geometric rectification으로 `Q_pos`를 생성.

> [!example]- 구현 디테일
> ```
> S = Φ_score(F_flat);  Ω_K = TopK_idx(S, K)
> Q_cont = P_sem(F_Ω)
> Q_pos  = B_init ⊕ Ψ_reg(F_Ω)     # ⊕: 좌표 정제 연산
> ```

<mark style="background: #FFF9D6A6;">"문제 정의"의 세 번째 문제를, query 생성의 원재료 자체가 이미 고주파 정보로 보강된 `F_tot`이기 때문에 자연스럽게 해결한다 — 저자는 `Q_pos`가 "tiny object 중심에 자연스럽게 정렬"되고 `Q_cont`가 "복원된 텍스처 semantic을 물려받아" decoder의 빠른 수렴을 보장한다고 설명한다. 즉 이 모듈 자체의 새로운 메커니즘이라기보다, HIP+IFE로 이미 만들어진 좋은 재료를 salient seed 선별로 최종 조립하는 역할.</mark>

# 실험 결과

### 핵심 결과 (AI-TOD-V2 test, ResNet50, 24 epoch)
| 벤치마크 | 지표 | Before(DINO baseline) | After(IG-DETR) |
|---|---|---|---|
| AI-TOD-V2 | AP | 25.9 | 30.9 (+5.0%p) |
| AI-TOD-V2 | AP_vt / AP_t / AP_s / AP_m | 12.7 / 25.3 / 32.0 / 39.7 | 16.0(+3.3) / 31.2(+5.9) / 37.2(+5.2) / 45.3(+5.6) |
| TinyPerson | AP_tiny50 (전체) | 55.8(DINO) | 60.1 |

> [!note]- 세부 결과 및 Ablation
> #### AI-TOD-V2 전체 비교 (Table 1)
> | 방법 | Epochs | AP | AP50 | AP75 | AP_vt | AP_t | AP_s | AP_m |
> |---|---|---|---|---|---|---|---|---|
> | RFLA(CNN 최고) | 12 | 25.7 | 58.9 | 18.8 | 9.2 | 25.5 | 30.2 | 40.2 |
> | Deformable-DETR | 50 | 18.9 | 50.0 | 10.5 | 6.5 | 17.6 | 25.3 | 34.4 |
> | DAB-DETR | 50 | 22.4 | 55.6 | 14.3 | 9.0 | 21.7 | 28.3 | 38.7 |
> | DINO | 24 | 25.9 | 61.3 | 17.5 | 12.7 | 25.3 | 32.0 | 39.7 |
> | DQ-DETR | 24 | 30.2 | 68.6 | 22.3 | 15.3 | 30.5 | 36.5 | 44.6 |
> | **IG-DETR(Ours)** | 24 | **30.9(+5.0)** | **69.3** | **23.0** | **16.0** | **31.2** | **37.2** | **45.3** |
> - DQ-DETR 대비도 전 지표에서 근소하지만 일관된 우위(AP +0.7, AP_vt +0.7 등).
>
> #### TinyPerson 비교 (Table 2, 극소형 인물 탐지)
> | 방법 | AP_tiny1_50 | AP_tiny2_50 | AP_tiny3_50 | AP_tiny_50 | AP_small_50 |
> |---|---|---|---|---|---|
> | DINO | 37.5 | 60.2 | 66.5 | 55.8 | 70.1 |
> | IG-DETR | **42.8** | **64.5** | **70.1** | **60.1** | **73.5** |
> - 가장 작은 크기 구간(tiny1)에서 개선폭이 가장 큼(37.5→42.8, +5.3) — 극소형 객체에 특히 강함을 시사.
>
> #### 모듈별 ablation (Table 3, DINO baseline)
> | HIP | IFE | IGQ | AP | AP_vt | AP_t | AP_s | AP_m |
> |---|---|---|---|---|---|---|---|
> | ✓ | | | 25.9 | 12.7 | 25.3 | 32.0 | 39.7 |
> | ✓ | ✓ | | 28.5 | 13.0 | 28.1 | 35.0 | 44.0 |
> | ✓ | ✓ | ✓ | 30.9 | 16.0 | 31.2 | 37.2 | 45.3 |
> - HIP+IFE(feature 보강)만으로 이미 +2.6 AP, IGQ(동적 query 배정) 추가로 +2.4 AP — 저자는 "동적 query 메커니즘이 가장 실질적인 기여를 제공한다"고 서술.
>
> #### Query 수 ablation (Table 4)
> 고정 300→1200까지 증가하며 AP 27.5→30.5로 상승 후 1500에서 30.3으로 소폭 하락 — 동적 방식(30.9)이 모든 고정 구성 대비 최고.
>
> #### 분류 vs 회귀 (Table 5)
> Baseline 25.9 → 회귀 15.5(급락) → 분류(채택) 30.9 — DQ-DETR과 동일한 패턴 재확인.
>
> #### 연산 비용 비교 (Table 6)
> | 방법 | GFLOPs | FPS | AP |
> |---|---|---|---|
> | DINO | 205 | 22.0 | 25.9 |
> | DQ-DETR(재구현) | 215 | 20.8 | 29.6 |
> | IG-DETR | 222 | 20.1 | 30.9 |
> - IG-DETR이 DQ-DETR보다 약간 더 무겁지만(GFLOPs +7, FPS -0.7) AP는 더 높음 — 저자가 직접 자신들의 DQ-DETR 재구현 결과(29.6, 논문 기재값 30.2와 차이)를 명시해 공정 비교를 시도한 점이 눈에 띔.

# Discussion

### 이 아이디어의 잠재적 부작용
- IFE의 `(1+W)` 덧셈 residual이 항상 "최소 원본 정보 보존"을 보장하지만, 이는 동시에 배경 영역의 노이즈도 항상 일정 비율 이상 유지된다는 의미 → <mark style="background: #FF5582A6;">논문은 이 트레이드오프(semantic 보존 vs 배경 억제력 약화)를 정량적으로 분석하지 않는다 — 곱셈 마스킹 대비 배경 억제가 더 약해질 가능성을 검증하지 않았다.</mark>
- 6단계 분류가 4단계보다 세밀하지만 경계 구간의 오분류 위험은 여전히 남음 → <mark style="background: #FF5582A6;">DQ-DETR·Density-Aware DETR과 달리 이 논문은 분류 정확도 자체(confusion matrix 등)를 전혀 보고하지 않는다 — "6단계가 4단계보다 낫다"는 주장의 직접적 정량 근거가 없다.</mark>

### 한계
- <mark style="background: #FF5582A6;">ICASSP 형식의 짧은 논문(4페이지)이라 방법론 설명이 상대적으로 간결하고, ablation도 AI-TOD-V2 단일 데이터셋에서만 수행 — TinyPerson에는 ablation이 없음.</mark>
- <mark style="background: #FF5582A6;">6단계 분류의 구간 경계값이 명시되지 않음("300, 500, 700, 900, ...") — DQ-DETR·Density-Aware DETR과 달리 구체적 임계값(예: N≤?)이 논문에 기재되지 않아 재현성이 떨어진다.</mark>
- Table 6에서 저자 스스로 DQ-DETR 재구현 결과(29.6)가 원 논문 기재값(30.2)과 다르다고 명시 — 재현 편차가 있음을 인정하면서도 이 편차의 원인은 분석하지 않음.
- DQ-DETR·Density-Aware DETR 대비 이 논문의 실질적 차별점(6단계 분류, additive residual)이 성능 개선에 각각 얼마나 기여하는지 분리된 ablation이 없음 — "HIP+IFE"가 하나로 묶여 보고되어(Table 3) 두 모듈 각각의 개별 기여를 알 수 없다.

### 생각할 점
- <mark style="background: #A6E3A1A6;">이 논문의 IFE는 [[DQ-DETR]]의 CGFE, [[Density-Aware-DETR]]의 spatial/channel attention과 계산식이 거의 동일하지만 "곱셈 vs 덧셈 residual"이라는 한 가지 설계 선택만 다르다 — 6편의 dynamic query DETR 계열 안에서 가장 미세한 차이로 차별화를 시도한 사례이며, 이 작은 차이가 실제로 유의미한지(Table 3에서 HIP+IFE 조합이 DQ-DETR의 CGFE 단독 기여보다 큰지)는 별도 통제 실험 없이는 확정하기 어렵다.</mark>
- <mark style="background: #A6E3A1A6;">Abstract에서 SAM(Segment Anything Model)을 대조군으로 언급(Fig. 1b)한 것은 이 위키의 다른 dynamic query DETR 논문에는 없는 접근 — "저수준 인지는 되지만 탐지로 이어지지 않는다"는 관찰이 향후 SAM류 foundation model을 query 생성에 활용하는 연구로 이어질 수 있는지 궁금증을 남긴다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">이 논문은 "instance-guided"라는 이름과 달리 실제로는 여전히 전역 밀도 분류(HIP) 기반이며, 개별 인스턴스 단위의 직접적 guidance는 salient seed 선택(top-K)에서만 나타난다 — 이후 처리할 PaQ-DETR(pattern/quality-aware)이 실제로 인스턴스 수준에서 얼마나 다른 신호를 쓰는지와 비교해, "instance-guided"라는 명명이 실질과 얼마나 부합하는지 검증할 필요가 있다.</mark>
- <mark style="background: #A6E3A1A6;">Additive residual injection(`1+W`) 방식은 이 위키의 다른 attention 기반 feature 강화 논문들([[FFCA-YOLO]]의 SCAM, [[RS-TOD]] 등)이 대체로 곱셈 마스킹을 쓰는 것과 대조된다 — 곱셈 대신 덧셈 residual을 쓰는 것이 다른 attention 모듈에도 일반적으로 이득이 되는지 검토할 가치가 있다.</mark>

# 관련 개념
- [[Density_Guided_Dynamic_Query]] — DQ-DETR·Density-Aware DETR과 동일 계열(density/난이도 기반 query 개수 결정)의 세 번째 사례로 "등장 논문"에 추가. 다만 이 논문 고유의 additive residual injection·salient seed selection은 이 개념 문서보다는 IFE/IGQ라는 이 논문 특유의 구현 디테일 성격이 강해 별도 concept으로는 분리하지 않음.

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]] — dynamic query DETR 계열 3번째 사례. DQ-DETR과 가장 직접적으로 비교(Table 1, 6)하며 근소하지만 일관된 우위를 보고.

# 읽어볼 만한 논문
- 참고문헌 기반: Y.-X. Huang, H.-I. Liu, H.-H. Shuai, W.-H. Cheng, "DQ-DETR: DETR with dynamic query for tiny object detection" (ECCV 2024) [26] — 이미 위키에 추가됨: [[DQ-DETR]]. 이 논문이 Table 1·6에서 가장 직접적으로 비교하는 baseline.
- 참고문헌 기반: X. Dai, Y. Chen, J. Yang, P. Zhang, L. Yuan, L. Zhang, "Dynamic DETR: End-to-end object detection with dynamic attention" (ICCV 2021) [19] — "dynamic"이라는 이름을 공유하지만 attention 메커니즘 자체를 coarse-to-fine으로 동적화한다는 점에서 이 논문(query 개수·내용의 동적화)과 다른 접근. 두 "dynamic"의 차이를 명확히 구분하는 데 유용.
- 참고문헌 기반: Q. Zhou, C. Yu, Z. Wang, F. Wang, "D2Q-DETR: Decoupling and dynamic queries for oriented object detection with transformers" (ICASSP 2023) [18] — Oriented object detection에 dynamic query를 적용한 선행 사례. 이후 처리할 DQA-DETR(oriented object detection)과 비교하며 읽으면 유용할 것으로 예상.
- 자유 추천(검증 필요): SAM(Segment Anything Model) 기반 pseudo-label이나 prior를 DETR query 생성에 활용하는 연구 — 검색 키워드: `SAM segment anything model object query prior DETR detection`. 이 논문이 도입부에서 SAM을 대조군으로만 언급하고 실제로는 활용하지 않는데, 실제로 SAM feature를 query guidance에 쓰는 후속 연구가 있는지 확인할 가치가 있음.

---
**보안 참고**: PDF 전체를 확인했으며, 프롬프트 인젝션이나 지시문처럼 보이는 텍스트는 발견되지 않았다.
