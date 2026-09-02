---
pm-task: true
projectId: "paperwiki-small-object-detection"
parentId:
id: "t-querydet-bjcya1z67y"
title: "QueryDet: Cascaded Sparse Query for Accelerating High-Resolution Small Object Detection"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-08-05"
due: ""
progress: 0
assignees: []
tags: []
customFields:
  "7l8l795xmtcjvlf6": 2022
  "1frf59rymtcjvske": "CVPR"
subtaskIds: []
dependencies: []
year: 2022
venue: "CVPR"
jcr_quartile: Q1
task: [small-object-detection]
direction: [novel-approach, foundational]
paper_tags: [paper, small-object-detection, sparse-convolution, inference-acceleration, feature-pyramid, query-mechanism]
source: "Projects/_pdf/Small_Object_Detection/2022_CVPR_QueryDet.pdf"
source_type: personal
createdAt: "2026-08-18T11:00:00.000Z"
updatedAt: "2026-08-18T11:07:50.810Z"
---

#paper #small-object-detection #sparse-convolution #inference-acceleration #feature-pyramid #query-mechanism

> [!quote] 원제
> **QueryDet: Cascaded Sparse Query for Accelerating High-Resolution Small Object Detection**
> Chenhongyi Yang, Zehao Huang, Naiyan Wang — University of Edinburgh / TuSimple, CVPR 2022
> https://doi.org/10.1109/CVPR52688.2022.01330

# 한 줄 요약
<mark style="background: #FFF3A3A6;">저해상도 feature map에서 소형 객체가 있을 만한 대략적 위치를 먼저 query로 예측하고, 그 위치 주변에만 sparse convolution으로 detection head를 적용해 정확도 손실 없이 고해상도 feature의 연산 비용을 74%에서 약 1%로 줄이는 Cascade Sparse Query(CSQ) 프레임워크.</mark>

> [!info] 내 메모
> 

# 정리

## 기존 방법의 한계
- **고해상도 feature의 연산 비용 폭증**:
  FPN에 고해상도 레벨 P2를 추가하면 RetinaNet 기준 detection head 연산량(FLOPs)이 300% 증가하고, 추론 속도가 2080Ti GPU 기준 13.6 FPS에서 4.85 FPS로 급락한다. P3까지만 써도 전체 연산의 43%, P2까지 추가하면 74%를 차지한다.
- **Dense 연산의 공간적 낭비**:
  소형 객체는 이미지·feature map 상에서 극히 일부 영역에만 sparse하게 분포하는데, 기존 detection head는 고해상도 feature map 전체에 dense하게 동일 연산을 적용해 대부분의 연산이 배경 영역에 낭비된다.

## 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Multi-scale feature 재사용**
- FPN[26]/SSD[29]/DSSD[10]/HyperNet[22]: 다양한 CNN 레이어의 multi-scale feature를 재사용해 고해상도 정보를 확보.
- Scale-aware Trident Networks[25]: receptive field-객체 크기 mismatch를 지적 — 연산 가속과는 무관.
- **타겟/해결**: 고해상도 feature의 연산 비용 폭증(문제 ①) — 고해상도 정보 확보는 가능해졌지만 그 레벨에서의 dense 연산 비용 자체는 해결하지 못한다.

**갈래 2 — Gating network 기반 sparse 연산**
- PerforatedCNN[9], Dynamic Convolution[47], SACT[8], SBNet[38]: Gumbel-Softmax나 별도 gating network로 sparse mask를 학습.
- **타겟/해결**: Dense 연산의 공간적 낭비(문제 ②) — sparse 연산 자체는 가능하지만 결정론적 샘플링이나 추가 sparsity loss/decision network가 필요해 학습이 복잡하다.

**갈래 3 — Coarse-to-fine 재처리**
- AutoFocus[33]: coarse scale에서 관심 영역을 예측해 crop 후 고해상도로 재처리.
- **타겟/해결**: Dense 연산의 공간적 낭비(문제 ②) — image pyramid 상에서 동작해 backbone 자체를 다시 돌려야 하는 중복 연산이 남는다.

**갈래 4 — Point-wise sparse 예측**
- PointRend[19]: 불확실한 위치를 골라 고해상도 segmentation을 sparse하게 계산.
- **타겟/해결**: Dense 연산의 공간적 낭비(문제 ②) — point-wise 분류라 단일 위치 feature만 쓰고 detection이 필요로 하는 주변 context를 놓치기 쉽다.

**갭**: <mark style="background: #FFF3A3A6;">기존 sparse 연산 기법들은 별도의 gating network·복잡한 학습 목표를 필요로 하거나(갈래 2), image pyramid 상에서 동작해 backbone 연산까지 반복되거나(갈래 3), point-wise 처리로 context 정보를 놓친다(갈래 4). Feature pyramid의 "레벨 간 강한 구조적 상관관계"(저해상도 예측이 고해상도 위치를 가리킬 수 있다는 성질) 자체를 직접 활용해, 단순한 GT 박스 supervision만으로 sparse 연산을 유도하는 방법은 없었다.</mark>

## 이 논문이 풀고자 하는 문제
1. 고해상도 feature map의 어느 위치에 소형 객체가 있을지 저비용으로 미리 알아내는 것.
2. 알아낸 위치에만 detection head 연산을 sparse하게 적용해 실제 속도 이득으로 연결하는 것.

**갭 종합**: <mark style="background: #FFF3A3A6;">저해상도 feature만으로도 "어디에 소형 객체가 있을지"는 저비용으로 예측 가능하고, feature pyramid의 레벨 간 구조적 상관관계를 이용하면 별도 gating network나 image pyramid 재처리 없이 GT 박스 supervision만으로 sparse 연산을 유도할 수 있다는 것이 이 논문의 통찰이다.</mark>

> [!info] 내 메모
> 

# 해결 방법 요약

| | 문제 ① — 고해상도 feature의 연산 비용 폭증 | 문제 ② — Dense 연산의 공간적 낭비 |
|---|---|---|
| **해결 방법** | 저해상도 feature map만으로 정확한 박스 회귀는 어렵지만 "이 근방에 소형 객체가 있는지"는 높은 신뢰도로 추론 가능하다는 관찰에서, query head로 이 대략적 위치를 예측 | 예측된 위치(query key)를 고해상도 feature map(query value)에 매핑해 그 위치에서만 sparse convolution으로 detection head 연산을 수행 |
| **예상되는 문제점** | Query head가 위치를 놓치면(recall 실패) 소형 객체를 원천적으로 탐지 못한다 — sparse 연산의 정확도 상한이 query head 성능에 종속된다. | 레벨마다 query를 직접 매핑하면 레벨 차가 커질수록 후보 위치 수가 지수적으로 폭증한다(아래 "제안 방법" ① 참고). |

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">저해상도 feature map에서 <span style="color:#c0392b; font-weight:bold;">query head</span>로 소형 객체가 있을 만한 대략적 위치(query key)를 예측하고, 한 단계 아래 고해상도 feature map(query value)에서 그 위치 주변만 <span style="color:#c0392b; font-weight:bold;">sparse convolution</span>으로 detection head 연산을 수행하며, 이를 레벨마다 순차적으로 반복하는 <span style="color:#c0392b; font-weight:bold;">Cascade Sparse Query(CSQ)</span>로 dense 연산을 회피한다.</mark>

## 전체 파이프라인 (Fig. 3 기준, RetinaNet 기반)

```
입력 이미지
       │
       ▼
Backbone + FPN                                → P2(1/4) ~ P7(1/128) 멀티스케일 feature
       │
       ▼ (P7 → P4까지는 dense conv로 기존과 동일하게 처리)
① Query Head (P4부터 시작)                      → heatmap V_l ∈ R^(H'×W')  [위치별 소형 객체 존재 확률]
       │  σ=0.15 넘는 위치를 query key로 선택
       ▼
② Cascade Sparse Query (CSQ) — 레벨 간 위치 매핑   → key position {k_(l-1)} (4개 최근접 위치로 매핑)
       │
       ▼
③ Sparse Detection Head (spconv, dense head 가중치 재사용) → 해당 위치만 분류(H×W×C)+회귀(H×W×4)+query(H×W×1) 계산
       │
       ▼ (P3 → P2로 반복, 레벨마다 ②③ 재수행)
출력: 전 레벨의 (클래스, 박스) 예측
```

> [!info] 내 메모
> 

### ① Query Head

- **역할**:
  분류·회귀 head와 병렬로 각 레벨 `P_l`에 배치되어, 그리드별로 "이 위치에 소형 객체가 있을 확률" heatmap `V_l ∈ R^(H'×W')`을 출력한다. 이 예측이 뒤이은 sparse 연산이 어디를 볼지 결정하는 유일한 근거가 된다.
- **구현**:
  분류·회귀 head와 동일한 4-conv 구조를 병렬로 추가. 학습 시 레벨별 소형 객체 기준 `s_l`(해당 레벨의 최소 anchor scale, anchor-free는 최소 regression range)보다 GT 중심과의 거리가 가까운 위치를 positive로 삼아 Focal Loss로 학습.
- **입출력 shape**: `P_l (H', W', C)` → `V_l (H', W', 1)`.

```python
# 논문 Eq.(2)-(3) 기반 의사코드
D_l[x][y] = min_o( sqrt((x - x_l^o)**2 + (y - y_l^o)**2) )   # 각 위치와 가장 가까운 GT 중심까지 거리
V_l_star[x][y] = 1 if D_l[x][y] < s_l else 0                  # 이진 target map
Loss_query = FocalLoss(V_l, V_l_star)
```

<mark style="background: #FFF9D6A6;">저해상도 feature만으로 정확한 박스 회귀는 어렵지만 "여기 근방에 소형 객체가 있는지"는 높은 신뢰도로 예측 가능하다는 관찰을 그대로 구현한 것 — "정리" 표 문제 ①(고해상도 dense 연산 비용)을 해결할 위치 정보를 저비용(4-conv 추가)으로 확보한다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> Query head의 recall이 100%가 아니므로, 소형 객체가 있어도 heatmap 값이 threshold σ를 넘지 못하면 그 위치는 이후 sparse 연산에서 완전히 배제된다 — 전체 파이프라인의 정확도 상한이 이 4-conv head 하나의 성능에 종속된다.

> [!info] 내 메모
> 

### ② Cascade Sparse Query (CSQ) — 레벨 간 위치 매핑

- **역할**:
  Query head가 예측한 저해상도 위치(query key)를 한 단계 아래 고해상도 레벨의 위치(key position)로 매핑해, sparse tensor를 구성할 좌표를 만든다. "Cascade"라는 이름대로 이 과정을 레벨마다 순차적으로 반복한다 — `P_(l-2)`의 query는 `P_(l-1)`에서 직접 만든 key position에서만 생성되고, `P_l`에서 곧바로 매핑하지 않는다.
- **구현**:
  `P_l`의 위치 `(x_l^o, y_l^o)`가 threshold를 넘으면, `P_(l-1)`에서 이에 대응하는 4개 최근접 위치를 key position으로 선택한다.
- **입출력 shape**: query key(스칼라 좌표 집합) → key position 집합(레벨 `l-1`의 좌표 4배).

```python
# 논문 Eq.(1)
key_positions_l_minus_1 = {(2*x_l + i, 2*y_l + j) for i in (0,1) for j in (0,1)}
```

<mark style="background: #FFF9D6A6;">한 레벨에서 여러 단계 아래로 한 번에 매핑하면 레벨 차가 커질수록 후보 위치 수가 지수적으로 늘어나는데, 인접 레벨끼리만 순차적으로 좁혀가는 cascade 구조는 이를 막는다 — "정리" 표 문제 ②(dense 연산의 공간적 낭비)를 해소하면서도 sparse 연산 자체가 새로운 폭증 문제를 만들지 않도록 하는 장치다.</mark>

> [!info] 내 메모
> 

### ③ Sparse Detection Head (spconv 기반)

- **역할**:
  CSQ가 골라낸 key position들만 골라 sparse tensor를 구성한 뒤, 그 위치에서만 분류·회귀·query 연산을 수행해 dense 연산을 완전히 회피한다.
- **구현**:
  Key position을 인덱스로 `P_(l-1)`에서 feature를 추출해 sparse tensor(query value feature) 구성. sparse convolution(spconv) 커널로 기존 4-conv dense head의 가중치를 그대로 재사용 — 별도 파라미터 없이 동일 가중치를 sparse 위치에서만 연산. Query 위치 주변 5×5 패치까지 함께 처리해야 context 부족으로 인한 정확도 손실이 없다(ablation 근거는 아래 "실험 결과" 참고).
- **입출력 shape**: sparse feature(key position 개수 × C) → 분류(key position 개수 × C_class) + 회귀(key position 개수 × 4) + query(key position 개수 × 1).

```python
# 의사코드
P_v_l_minus_1 = extract_sparse(P_l_minus_1, key_positions_l_minus_1)   # 지정 위치만 sparse tensor로 추출
sparse_head = build_spconv_kernel(weights=dense_head.weights)          # dense head 가중치 그대로 재사용
cls, reg, query_next = sparse_head(P_v_l_minus_1)                      # 해당 위치만 연산
```

<mark style="background: #FFF9D6A6;">Dense 연산이 낭비되는 배경 영역을 애초에 계산하지 않으므로, 연산 비용이 소형 객체가 실제로 존재하는 sparse한 영역에 비례하게 된다 — 고해상도 P2/P3 연산 비중을 74%에서 약 1%로 줄인 핵심 근거(Fig. 2).</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> 대형 객체 위치가 query head에서 오탐(false positive)으로 활성화되면, 불필요한 sparse 연산이 늘어 오히려 속도가 저하되는 failure case가 있다(논문이 시각화로 직접 제시).

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| ① Query Head | P_l (H', W', C) | V_l (H', W', 1) | 소형 객체 대략적 위치 예측 | 4-conv (분류·회귀 head와 병렬) |
| ② CSQ 매핑 | query key 좌표 | key position 좌표(×4) | 레벨 간 위치를 순차적으로 좁힘 | 4-최근접 매핑, cascade 반복 |
| ③ Sparse Head | sparse tensor(key position 개수 × C) | 분류+회귀+query(key position 개수 기준) | dense 연산 회피, 실제 속도 이득 | spconv, dense head 가중치 재사용 |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table 1(COCO), Table 2(VisDrone)
**표를 보는 법**: CSQ 열이 -/×/✓ 세 상태로, "고해상도 없음" → "고해상도 dense(느림)" → "고해상도+CSQ(빠름)" 순으로 AP와 FPS가 어떻게 바뀌는지 비교하면 된다.

| 벤치마크 | 지표 | Before(고해상도 dense, CSQ×) | After(CSQ 적용) |
|---|---|---|---|
| COCO mini-val (RetinaNet) | AP / APS / FPS | 38.53 / 24.64 / 4.85 | 38.36 / 24.33 / 14.88 |
| VisDrone val (RetinaNet) | AP / FPS | 28.35 / 1.16 | 28.32 / 2.75 |

> [!note]- 세부 결과 및 Ablation
> #### Table 3 — Ablation (COCO mini-val, RetinaNet)
> **보는 법**: HR(고해상도 P2 추가)/RB(loss 재조정)/QH(query head)/CSQ를 하나씩 순서대로 켜가며 AP·FPS 변화를 본다.
> HR만 추가 시 AP가 37.46→36.10으로 오히려 하락(학습 샘플 분포 급변) → RB(레벨별 loss 재조정, βl을 P2→P7로 1~3 선형 증가)로 38.11까지 회복 → QH 추가로 38.53(+0.42 AP, +1.58 APS) → CSQ 적용 시 AP는 38.36으로 0.17 손실이지만 FPS가 4.85→14.88로 3.07배.
>
> #### Table 4 — CSQ 시작 레벨 비교
> **보는 법**: 어느 레벨부터 query를 시작하는지에 따른 AP-FPS 트레이드오프.
> P6/P5에서 시작하면 저해상도 연산 자체가 이미 빠르고 sparse tensor 구성 비용이 그 이득을 넘어서 느림(FPS 13.42/13.92) → P4가 최적(FPS 14.88, AP 38.36) → P3에서 시작하면 AP는 약간 오르지만(38.45) FPS는 11.51로 하락.
>
> #### Table 5 — Query 방식 비교
> **보는 법**: CSQ, Crop Query(CQ, AutoFocus 유사), Complete Convolution Query(CCQ, 전체 conv 후 결과만 추출) 세 방식의 AP·FPS.
> 세 방식 모두 AP 손실은 비슷하게 작지만(38.31~38.36) CSQ가 가장 빠름(FPS 14.88 vs CQ 10.49, CCQ 8.73).
>
> #### Table 6 — Context 패치 크기 비교
> **보는 법**: query 위치 주변 몇 ×몇 패치까지 함께 처리하는지에 따른 AP-FPS.
> 1×1(AP 38.25, FPS 14.09) → 5×5(AP 38.36, FPS 14.00)까지 AP가 오르고 그 이상(7×7~11×11)은 AP 이득은 미미(38.37~38.38)한 반면 FPS만 계속 하락 → 5×5가 균형점.
>
> #### Figure 4 — Query threshold σ에 따른 속도-정확도 트레이드오프
> **보는 법**: x축 FPS, y축 AP/AR, σ를 0.05씩 늘려가며 그린 곡선 — 맨 왼쪽 삼각형이 CSQ 미적용 지점.
> 매우 낮은 threshold(0.05)에서도 이미 큰 속도 이득이 나며, 입력 해상도가 클수록(예: s960) AP 상·하한 gap이 작아 높은 threshold에서도 AP 하한을 보장할 수 있음.
>
> #### Table 7 — 경량 backbone 결과
> MobileNetV2: 고해상도 검출 속도 평균 4.1배 향상. ShuffleNetV2: 평균 3.8배 향상 — backbone 연산 비중이 작을수록 CSQ의 상대적 가속 효과가 커짐(backbone은 그대로, head만 가속하기 때문).
>
> #### Table 8 — FCOS(anchor-free) 적용
> AP 38.37(FCOS)→40.05(No CSQ)→39.49(CSQ), 고해상도 속도 1.8배 향상. Anchor-free detector에도 그대로 적용 가능함을 보여줌.
>
> #### Table 9 — Faster R-CNN(2-stage, RPN) 적용
> RPN 입력을 P2~P6로 확장 후 CSQ 적용. AP 38.47→38.20(APS 소폭 하락), FPS 17.57→19.03. 2-stage detector의 RPN 단계 가속에도 유효 — RoI 개수 자체도 줄어드는 부가 효과.
>
> #### Failure case (Fig. 5)
> 1) Query head가 위치를 맞춰도 detection head가 국소화(localization)에 실패하는 경우(VisDrone). 2) 대형 객체 위치가 오탐으로 활성화되어 불필요한 sparse 연산이 발생, 속도가 저하되는 경우(COCO).

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- Query head의 recall 실패 시 소형 객체를 원천적으로 놓칠 위험 → <mark style="background: #FF5582A6;">논문은 매우 낮은 threshold(0.05)에서도 큰 속도 이득이 난다는 점으로 이 위험을 완화했다고 보이나, query head 성능 자체가 상한으로 남는다는 점은 해소되지 않는다.</mark>
- 대형 객체 위치의 오탐(false positive) 활성화로 인한 속도 저하 → <mark style="background: #FF5582A6;">논문이 failure case로 명시했을 뿐 별도 해결책은 제시하지 않았다.</mark>

### 한계
- <mark style="background: #FF5582A6;">Query head가 정확한 위치를 찾아도 detection head 자체가 국소화(localization)에 실패하는 case가 존재(VisDrone 실패 사례, Fig. 5).</mark>
- <mark style="background: #FF5582A6;">Query 시작 레벨, threshold σ, context 패치 크기 등 하이퍼파라미터에 성능-속도 트레이드오프가 민감하게 반응해, 데이터셋·해상도별 재튜닝이 필요해 보인다.</mark>

### 생각할 점
- <mark style="background: #A6E3A1A6;">"저해상도 예측으로 고해상도 연산 위치를 좁힌다"는 아이디어는 본질적으로 coarse-to-fine 접근이며, [[2024_ECCV_SR-TOD|SR-TOD]]/[[2025_RSASE_RS-TOD|RS-TOD]] 등 feature 강화 계열과는 직교적이다 — "어디를 볼지"와 "어떻게 강화할지"를 함께 쓸 여지가 있다.</mark>
- <mark style="background: #A6E3A1A6;">Sparse convolution으로 dense head 가중치를 그대로 재사용하는 방식은 추가 파라미터 없이 기존 detector에 이식 가능하다는 점에서 실용적이다 — 저자들도 RetinaNet/FCOS/Faster R-CNN 세 아키텍처에 동일하게 적용해 일반성을 실증했다(Table 8, 9).</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">[[2026_TIP_Unc-SOD|Unc-SOD]]처럼 "어떤 prior를 positive로 볼지"를 동적으로 정하는 label assignment 계열과 CSQ의 query 메커니즘을 결합하면, sampling 품질과 연산 효율을 동시에 개선할 수 있을 것으로 보인다.</mark>
- <mark style="background: #A6E3A1A6;">원격탐사/드론뷰 논문들([[2025_RSASE_RS-TOD|RS-TOD]], [[2024_TGRS_FFCA-YOLO|FFCA-YOLO]], [[2026_JSTARS_RTP-Net|RTP-Net]])은 대부분 고해상도 입력에서의 연산 비용 문제를 정면으로 다루지 않는데, CSQ의 sparse 연산 아이디어를 결합하면 정확도 손실 없이 실시간성을 확보할 잠재력이 있다.</mark>

> [!info] 내 메모
> 

# 관련 개념
- [[Cascade_Sparse_Query]] — 이 논문의 핵심 기여.

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]]

# 읽어볼 만한 논문
- 참고문헌 기반: A. Kirillov, Y. Wu, K. He, R. Girshick, "PointRend: Image segmentation as rendering" (CVPR 2020) [19] — QueryDet과 유사하게 sparse한 위치만 골라 고해상도 예측을 하지만 point-wise MLP를 쓴다는 점에서 대조되는 접근. Sparse 연산 설계의 대안 이해에 도움.
- 참고문헌 기반: M. Najibi, B. Singh, L. S. Davis, "AutoFocus: Efficient multi-scale inference" (ICCV 2019) [33] — QueryDet이 직접 비교하는 가장 유사한 선행 연구(image pyramid 기반 coarse-to-fine). CSQ가 이를 feature pyramid로 옮겨 backbone 중복 연산을 없앤 차별점을 이해하는 데 필수.
- 참고문헌 기반: B. Zhu, J. Wang, Z. Jiang, F. Zong, S. Liu, Z. Li, J. Sun, "AutoAssign: Differentiable label assignment for dense object detection" (arXiv 2020) [36] — QueryDet의 query head 학습(거리 기반 positive 정의)과 비교할 만한 differentiable label assignment 접근.
- 자유 추천(검증 필요): Sparse convolution 기반 3D object detection(LiDAR) 연구 — 저자들이 Conclusion에서 향후 3D 확장을 명시적으로 언급함. 검색 키워드: `sparse convolution 3D object detection LiDAR point cloud CVPR`
