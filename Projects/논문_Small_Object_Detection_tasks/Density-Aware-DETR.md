---
pm-task: true
projectId: "paperwiki-small-object-detection"
parentId:
id: "t-density-aware-detr-3c88bcqs6i"
title: "Density-Aware DETR With Dynamic Query for End-to-End Tiny Object Detection"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-08-24"
due:
progress: 0
assignees: []
tags: []
customFields:
  "7l8l795xmtcjvlf6": 2025
  "1frf59rymtcjvske": "IEEE JSTARS"
subtaskIds: []
dependencies: []
year: 2025
venue: "IEEE Journal of Selected Topics in Applied Earth Observations and Remote Sensing (JSTARS)"
jcr_quartile: Q1
task: [small-object-detection]
direction: [improvement]
paper_tags: [paper, small-object-detection, tiny-object-detection, detr, dynamic-query, density-estimation, remote-sensing, label-assignment]
source: "/Users/GyeongSeo/Workspace/논문_pdf/Small_Object_Detection/2025_JSTARS_Density-Aware-DETR.pdf"
createdAt: "2026-08-24T03:09:00.000Z"
updatedAt: "2026-08-28T19:00:00.000Z"
---

Project: [[논문_Small_Object_Detection|Small Object Detection]]
#paper #small-object-detection #tiny-object-detection #detr #dynamic-query #density-estimation #remote-sensing #label-assignment

> [!quote] 원제
> **Density-Aware DETR With Dynamic Query for End-to-End Tiny Object Detection**
> Xianhang Ye, Chang Xu, Haoran Zhu, Fang Xu, Haijian Zhang, Wen Yang — Wuhan University, IEEE JSTARS 2025

# 한 줄 요약
<mark style="background: #FFF3A3A6;">DQ-DETR의 이산 4단계 query 개수 분류를 crowd counting 기법 기반 연속 density map 회귀로 대체하고, 점 단위 density focal loss(DFL)로 이를 정밀하게 학습시키며, tiny object에서 L1 loss의 supervision이 약해지는 문제를 log-ratio 기반 anchor L1 measure로 보완한 D3Q(Density-aware DETR with Dynamic Query) — AI-TOD-v2에서 DINO baseline 대비 mAP +3.6%p, DQ-DETR 대비도 우위를 보이며 새 SOTA(32.1% mAP)를 달성한 plug-and-play 모듈.</mark>

> [!info] 내 메모
> 

# 정리

| | 문제 ① — 고정 query 수와 극단적 밀도 편차의 충돌 | 문제 ② — 정규화 좌표의 약한 회귀 supervision |
|---|---|---|
| **문제 정의** | 원격탐사 영상은 이미지당 인스턴스 수가 COCO(대부분 100개 미만)와 달리 AI-TOD-v2 기준 1~2667개까지 극단적으로 편차가 크다(Fig. 2). Sparse query는 밀집 장면의 recall을 낮추고, dense query는 sparse 장면에서 one-to-one assignment 최적화를 어렵게 만들며 연산도 낭비한다. | DETR은 박스 좌표를 [0,1] 정규화 표현으로 회귀하는데, tiny object는 이 정규화 좌표에서 너비·높이 값 자체가 매우 작아 GT와의 편차도 작아진다. 표준 L1 loss는 절대 오차만 보므로, 이 작은 편차에 대한 유효 gradient가 거의 0에 가까워 supervision이 사실상 사라진다. |
| **풀고자 하는 문제** | 이미지별 실제 객체 밀도를 연속적으로 추정해 object query 수를 데이터셋·이미지에 상관없이 자동으로 적응시키는 것 | Tiny object의 박스 회귀에도 유효한 gradient가 남도록 손실 함수 자체를 재설계하는 것 |
| **선행 연구 접근** | - DQ-DETR([[DQ-DETR]]): 인스턴스 수를 4개 이산 구간(N≤10/10~100/100~500/500+)으로 분류해 query 수(300/500/900/1500)를 결정 — 구간 폭이 넓어 서로 다른 밀도(예: 101개 vs 499개)가 같은 구간으로 뭉개짐<br>- Crowd counting 계열(density map 기반 연속 카운팅): detection과 직접 결합된 사례는 드묾 | - Anchor 기반 검출기: 앵커 대비 상대 오프셋으로 박스를 인코딩해 스케일 불변적 회귀를 함<br>- DETR 계열: 정규화 좌표에 표준 L1만 사용 — tiny object에 특화된 조정 없음 |
| **해결 방법** | IDE(Instance Density Estimation)가 encoder feature에서 연속적인 salient density map을 예측하고, 그 합(적분)으로 이미지의 객체 수를 직접 추정 — 이산 분류가 아닌 연속값이라 세밀한 밀도 차이도 보존 | Anchor L1 measure — 박스의 너비·높이에 로그 비율(`|log(l_p/l_gt)|`)을 적용해 값의 절대 크기와 무관하게 상대 오차에 민감한 gradient를 유지 |
| **예상되는 문제점** | Density map을 pixel-level로 회귀하는 방식이라 예측 개수(K)가 실제 GT 개수를 크게 넘어서는 경우 query 조정 효율이 떨어질 수 있음(Table VII에서 L2 loss 사용 시 실제로 발생) | 로그 비율 방식은 중심 좌표(x,y)에는 적용되지 않고 너비·높이(w,h)에만 적용되므로, 중심 위치 자체의 supervision 약화 문제는 여전히 남음 |

**갭 종합**: <mark style="background: #FFF3A3A6;">DQ-DETR이 처음으로 밀도 기반 dynamic query 개념을 제시했지만, 이산 분류라는 선택이 밀도 정보의 세밀함을 희생시켰다. 이 논문은 "이산 분류 대신 연속 회귀"라는 하나의 축과, "정규화 좌표의 회귀 약화"라는 완전히 다른 축(query 개수와 무관한 loss 설계 문제)을 함께 해결함으로써, DQ-DETR이 놓친 정밀도와 DETR 계열 전체가 겪는 tiny object 회귀 문제를 동시에 메운다는 것이 통찰이다.</mark>

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">Encoder feature로부터 <span style="color:#c0392b; font-weight:bold;">연속적인 salient density map</span>을 회귀해 이미지의 객체 수를 직접 추정하고(IDE), 이 추정치에 balance factor를 곱해 정한 개수만큼 query를 동적으로 선택·초기화하며(DQA), 회귀 손실은 <span style="color:#c0392b; font-weight:bold;">log-ratio 기반 anchor L1 measure</span>로 대체해 tiny object에서도 유효한 gradient를 확보한다.</mark>

## 전체 파이프라인 (Fig. 3 기준)

```
입력 이미지
       │
       ▼
① Backbone + Encoder (N층)                → encoder memory (flattened feature)
       │
       ├──────────────────────────────┐
       ▼                              │
② IDE (Instance Density Estimation)    │
   Reconstruction → Density Head       │  → 연속 density map Ŷ (W,H,1), 객체 수 K=Σ(Ŷ)
       │                              │
       ▼                              ▼
③ DQA — Query Adjustment (QA)          First Stage Head (encoder memory → classification score + box coords)
   K × T(balance factor) = N(최종 query 수) → top-N proposal 선택         → query proposals (N, 4)
       │
       ▼
④ DQA — Dynamic Mix Selection (DMS)
   Static embedding[:N] = query content, PE(reference points) = query position
       │                                          → query content (N,d) + query position (N,d)
       ▼
⑤ Decoder (N층)                          → decoder 출력 (N, d)
       │
       ▼
⑥ Fully Connected Layers (Head)           → 클래스 + 박스 (N개)
       │
       ▼ (학습 시)
⑦ Loss: L_cls + λ2·L_box(anchor L1) + λ3·L_IOU + λ_dm·L_density(DFL)
```

> [!info] 내 메모
> 

### ① Backbone + Encoder
- **역할**: 입력 이미지에서 다중 스케일 feature를 추출하고 encoder로 문맥을 반영한 memory를 만든다. 이 논문의 기여가 시작되는 지점은 이 memory를 IDE가 다시 2D feature map으로 재구성하는 부분부터다.
- **구현**: CNN backbone + deformable encoder(baseline은 Deformable DETR 또는 DINO 그대로 사용). Encoder의 flatten된 feature `F ∈ R^(n×c)`를 다시 2D feature pyramid `F_d^i ∈ R^(c×h^i×w^i)`로 재구성(reconstruction)한다.
- **입출력 shape**: 입력 이미지 → encoder memory `F (n, c)` → reconstruction → `{F_d^i}` (multi-level 2D feature).

> [!info] 내 메모
> 

### ② Instance Density Estimation (IDE)
- **역할**: Tiny object 검출에는 고해상도 feature가 특히 중요하므로, 여러 레벨의 재구성된 feature 중 **가장 해상도가 높은 첫 번째 레벨**(`F_d^1`)만을 사용해 "이 이미지에 객체가 어디에, 얼마나 밀집해 있는지"를 나타내는 연속값 density map을 예측한다. 이 density map의 총합(적분)이 곧 이미지의 추정 객체 수가 된다 — DQ-DETR처럼 이산 구간으로 분류하지 않고, crowd counting 기법에서 착안한 연속 회귀 방식을 쓴다.
- **구현**: `F_d^1`을 CenterNet의 heatmap head와 유사한 density head(3×3 conv + ReLU → [[1x1_Convolution]] → sigmoid)에 통과시켜 density map `Ŷ ∈ R^(W×H×1)`을 얻는다. 객체 수 `K`는 `Ŷ`의 모든 픽셀 합(`K = Σ_xyc Ŷ`)으로 계산한다. 학습 시 GT 감독을 위해, 각 GT 중심점에 객체 크기에 비례한 표준편차 `σ`를 갖는 정규화된 2D Gaussian kernel(Fig. 4)을 씌워 density map GT를 만든다.
- **입출력 shape**: `F_d^1 (c, h^1, w^1)` → conv+ReLU+1×1conv+sigmoid → `Ŷ (W, H, 1)` → 합산 → `K` (스칼라).

```python
# 논문 Eq.(1)-(3) 기반 의사코드
Y_hat = sigmoid(conv1x1(relu(conv3x3(F_d_1))))    # Eq.1, density head
K = Y_hat.sum(dim=[x,y,c])                          # Eq.2, 객체 수 추정

# GT target 생성 (Eq.3, Fig.4): 각 GT 중심 (x_hat, y_hat)에 정규화 가우시안 커널
G_xy = exp(-((x-x_hat)**2+(y-y_hat)**2) / (2*sigma**2)) / sum(exp(...))   # sigma는 객체 크기에 비례
```

(연한 노랑 하이라이트) <mark style="background: #FFF9D6A6;">이산 분류가 아니라 연속 회귀를 쓰기 때문에, DQ-DETR에서는 같은 구간으로 뭉개졌을 서로 다른 밀도(예: 101개 vs 499개)도 서로 다른 query 수로 반영된다 — "정리" 표의 첫 번째 문제(고정 query 수와 밀도 편차의 충돌)를 더 세밀하게 해결하는 근거가 된다. Table III에서 DQ-DETR 대비 mAP +6.3%p, AP75 +16.1%p로 이 정밀도 차이가 실제 성능으로 이어짐을 확인.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> Density map이 pixel-level 회귀 결과이므로, 학습이 불안정하면(특히 L2 loss로 학습할 경우) 예측 객체 수(K)가 실제보다 훨씬 크게 나올 위험이 있다 — Table VII에서 L2 loss 사용 시 "예측 밀도 맵의 정확도가 낮아 적분으로 얻은 객체 수가 설정한 query 상한을 초과해, query의 동적 조정 자체를 방해한다"고 저자가 직접 분석.

> [!info] 내 메모
> 

### ③ Dynamic Query Adaptation (DQA) — Query Adjustment (QA)
- **역할**: IDE가 추정한 객체 수 `K`를 그대로 query 수로 쓰지 않고, balance factor `T`를 곱해 최종 query 수 `N`을 정한다 — 이는 학습 중 양성:음성 샘플 비율을 일정하게 유지해 recall 저하와 최적화 불안정을 동시에 막기 위함이다. 그 다음, encoder memory에서 별도 first-stage head(분류+박스 좌표 예측)로 얻은 proposal 중 confidence 상위 `N`개를 선택해 decoder에 넣을 초기 reference point로 삼는다.
- **구현**: `N = K × T`(T는 하이퍼파라미터, ablation에서 T=3이 최적). Encoder memory `F_e`를 first-stage head(binary classification: 전경 여부)에 통과시켜 각 proposal의 classification score와 좌표 bias(`x,y,w,h`)를 얻고, score 기준 top-N proposal을 선택한다. 선택된 proposal의 좌표가 decoder deformable attention의 초기 reference point가 된다.
- **입출력 shape**: encoder memory `F_e (n, c)` → first-stage head → score+coords `(n, 5)` → top-N 선택 → `query_proposals (N, 4)`.

```python
# 논문 Algorithm 1 기반 의사코드
K = Y_d.sum()                          # IDE의 밀도 맵 적분
N = K * T                              # balance factor T (ablation 최적값 T=3)

scores, coords = detection_head(F_e)   # first-stage head, binary classification
query_proposals = topk(scores, coords, N)   # confidence 상위 N개 선택
reference_points = original_proposals + query_proposals   # decoder 초기 reference point
```

(연한 노랑 하이라이트) <mark style="background: #FFF9D6A6;">Balance factor `T`로 양성:음성 샘플 비율을 일정하게 유지함으로써, IDE의 밀도 추정 오차가 곧바로 학습 불안정으로 이어지는 것을 완충한다 — "정리" 표의 첫 번째 문제 해결을 마무리하는 단계로, sparse 장면에서는 적은 query로 효율을, dense 장면에서는 많은 query로 recall을 확보한다.</mark>

> [!info] 내 메모
> 

### ④ Dynamic Query Adaptation (DQA) — Dynamic Mix Selection (DMS)
- **역할**: 기존 DETR 계열의 object query는 대개 고정된 학습 파라미터(static embedding)이거나, 위치만 동적으로 인코딩하는 두 극단 중 하나였다. DMS는 이 둘을 결합해, query content는 학습 가능한 static embedding에서 순차 선택하고, query position은 ③에서 뽑은 동적 reference point를 positional encoding으로 변환해 사용한다 — DINO의 "mixed selection"을 동적 query 개수 상황에 맞게 확장한 것.
- **구현**: 최대 query 수(`n_max`) × 차원(`c`) 크기의 static embedding을 학습 파라미터로 유지하고, 그중 앞의 `N`개를 순차적으로 선택해 query content로 쓴다. Query position은 ③의 reference point 좌표를 positional encoding(PE)에 통과시켜 얻는다. 최종 query는 content와 position을 더해 만든다.
- **입출력 shape**: static embedding `(n_max, c)` → 순차 선택 `(N, c)` = query content. Reference point `(N, 4)` → PE → `(N, c)` = query position. `query = query_content + query_position (N, c)`.

```python
# 논문 Algorithm 1 기반 의사코드
query_content = static_embedding[:N, ...]     # 학습 가능한 static embedding에서 순차 선택
query_position = PE(reference_points)          # 동적 reference point를 positional encoding
query = query_content + query_position          # (N, c)
```

(연한 노랑 하이라이트) <mark style="background: #FFF9D6A6;">Static content(안정적인 학습 신호)와 dynamic position(이미지별 실제 객체 위치 반영)을 결합함으로써, DQ-DETR의 단일 초기화 방식보다 query의 판별력이 향상된다 — Fig. 9의 t-SNE 시각화에서 D3Q의 query feature가 baseline보다 카테고리별로 더 뚜렷하게 군집을 이루는 것이 이 근거다.</mark>

> [!info] 내 메모
> 

### ⑤⑥ Decoder + Head
- **역할**: DMS가 만든 `N`개의 query를 표준 decoder에 넣어 encoder memory를 참조하며 정제하고, 최종 (클래스, 박스)를 예측한다. 이 부분은 이 논문의 기여가 아니라 baseline(Deformable DETR 또는 DINO)의 표준 구조를 그대로 사용한다.
- **입출력 shape**: `query (N, c)` + encoder memory → decoder 출력 `(N, c)` → 클래스 + 박스.

> [!info] 내 메모
> 

### ⑦ Loss (Anchor L1 Measure + Density Focal Loss)
- **역할**: 두 가지 손실 개선을 도입한다. 첫째, tiny object의 박스 회귀(특히 너비·높이)에서 정규화 좌표의 절대 오차가 너무 작아 표준 L1 loss의 gradient가 거의 사라지는 문제를, anchor 기반 검출기의 "앵커 대비 상대 오프셋 회귀" 아이디어에서 착안한 log-ratio 측정으로 대체한다. 둘째, IDE의 density map 학습에도 표준 L2가 아니라 focal loss 스타일의 point-level density focal loss(DFL)를 써서, 배경 픽셀이 대부분인 density map에서 foreground와 background 샘플의 불균형을 완화한다.
- **구현**: Anchor L1 measure는 너비·높이에만 적용(`L_box = |log(l_p/l_gt)|`, `l`은 w 또는 h) — 중심 좌표(x,y)에는 표준 L1을 그대로 쓴다. Hungarian 매칭의 cost도 동일하게 `C_box = -|log(l_p/l_gt)|`로 맞춘다. DFL은 예측값이 GT와 가까워질수록(즉 이미 잘 맞춘 샘플일수록) loss 기여를 줄이는 modulating factor를 도입 — 양성 픽셀(`Y_xyc>0`)에는 `(Ŷ_xyc)^γ`, 음성 픽셀(`Y_xyc=0`)에는 `|Y_xyc - Ŷ_xyc|^γ`를 각각 BCE에 곱한다(γ=2).
- **입출력**: (파라미터 갱신용 스칼라 loss).

```python
# 논문 Eq.(4)-(8) 기반 의사코드
L_box = abs(log(l_pred / l_gt))                    # Eq.5, w/h에만 적용(anchor L1 measure)
C_box = -abs(log(l_pred / l_gt))                    # Eq.6, Hungarian cost도 동일하게

# Density Focal Loss (Eq.4)
gamma = 2
def dfl_term(Y, Y_hat):
    if Y == 0:
        return (Y_hat ** gamma) * log(1 - Y_hat)      # 배경 픽셀
    else:
        return (abs(Y - Y_hat) ** gamma) * bce(Y, Y_hat)  # 전경 픽셀
L_density = -mean(dfl_term(Y_xyc, Y_hat_xyc) for xyc in all_pixels)

L_total = lam1*L_cls + lam2*L_box + lam3*L_IOU + lam_dm*L_density   # Eq.7, lam1=lam3=2(DINO 관례), lam2=1, lam_dm=2
```

(연한 노랑 하이라이트) <mark style="background: #FFF9D6A6;">"정리" 표의 두 번째 문제(정규화 좌표의 약한 supervision)를, 절대 오차 대신 상대 비율의 로그를 측정함으로써 정면으로 해결한다 — 값의 절대 크기와 무관하게 오차 비율이 같으면 동일한 크기의 gradient를 갖게 되어, tiny object에서도 회귀 신호가 사라지지 않는다. Fig. 7에서 anchor L1 measure를 추가한 모델이 baseline보다 수렴이 빠르고 최종 AP도 높은 것으로 확인.</mark>

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| ① Backbone+Encoder | 입력 이미지 | encoder memory (n,c) | 다중 스케일 feature 추출·문맥 반영 | CNN backbone + deformable encoder |
| ② IDE | 최고해상도 재구성 feature | density map Ŷ(W,H,1) + K(스칼라) | 연속 밀도 추정 → 객체 수 계산 | conv3x3+ReLU+[[1x1_Convolution]]+sigmoid |
| ③ QA | encoder memory + K | query_proposals(N,4), N=K×T | 동적 query 수 결정 + top-N 선별 | first-stage head(분류+좌표) + top-k |
| ④ DMS | static embedding + reference points | query content(N,c) + position(N,c) | Static+dynamic 혼합 query 초기화 | Static embedding 순차 선택 + PE |
| ⑤⑥ Decoder+Head | query(N,c) | 클래스+박스(N개) | 최종 검출 예측 | 표준 deformable decoder |
| ⑦ Loss | 예측+정답+density map | 스칼라 loss | 학습 신호 결정 | Anchor L1 measure + Density Focal Loss |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table I (AI-TOD-v2, DINO baseline 기준)
**표를 보는 법**: baseline(DINO)과 D3Q(DINO w/ D3Q, 두 학습 방식: from scratch / two-stage)를 나란히 비교하면 이 논문의 순수 이득을 볼 수 있다. `*`는 DQ-DETR과 동일한 2단계 학습 스킴을 적용한 실험.

| 벤치마크 | 지표 | Before(DINO baseline) | After(DINO w/ D3Q) |
|---|---|---|---|
| AI-TOD-v2 | mAP / AP50 / AP75 | 28.5 / 61.0 / 22.6 | 32.1(+3.6) / 67.0(+6.0) / 25.9(+3.3) (2단계 학습*) |
| AI-TOD-v2 | AP_vt / AP_t / AP_s / AP_m | 14.1 / 24.2 / 28.0 / 39.6 | 17.3 / 32.7 / 36.5 / 46.1 (2단계 학습*) |

> [!note]- 세부 결과 및 Ablation
> #### Table I — AI-TOD-v2 전체 비교 (CNN 계열 vs DETR 계열)
> **보는 법**: CNN 기반(RetinaNet~FFCA-YOLO)과 DETR 기반(Deformable-DETR~DQ-DETR) 그룹으로 나뉜 표에서, DETR 그룹 안에서 D3Q의 위치를 확인.
> Deformable-DETR(baseline) 17.8 → Deformable-DETR w/ D3Q **23.9(+6.1)**. DINO(baseline) 28.5 → DINO w/ D3Q 30.2(+1.7, from scratch) → DINO w/ D3Q* 32.1(+3.6, 2단계 학습) — 이 시점 CNN 계열 최고인 RFLA(25.7)도 상회.
>
> #### Table II — AI-TOD-v1 (일반화 검증)
> DINO w/ D3Q(ours) mAP 33.0으로 BAFNet(30.5), HS-FPN(25.1) 등 상회 — 다른 데이터셋에서도 일관된 우위.
>
> #### Table III — DQ-DETR과의 직접 비교 (동일 DINO baseline)
> **보는 법**: Scheme 열(From Scratch/Two Stage)까지 함께 봐야 공정 비교임을 알 수 있다. DQ-DETR은 2단계 학습이 필수인 반면 D3Q는 from scratch로도 학습 가능.
>
> | Scheme | Queries | AP | AP_vt | AP_t | GFLOPs | Params |
> |---|---|---|---|---|---|---|
> | DINO(baseline, From Scratch) | 900 | 28.5 | 14.1 | 29.0 | 520 | 48M |
> | DQ-DETR(Two Stage, 4 Levels) | 903 | 30.2 | 15.3 | 30.5 | 903 | 59M |
> | D3Q(ours, From Scratch, Dynamic) | 543 | 30.2 | 16.0 | 30.4 | 543 | 49M |
> | D3Q(ours, Two Stage, Dynamic) | 543 | **32.1** | **17.3** | **32.7** | 543 | 49M |
>
> D3Q는 DQ-DETR과 동일 학습 방식(Two Stage)에서 AP75 +16.1%, mAP +6.3% 더 높으면서 GFLOPs는 543 vs 903으로 훨씬 적다 — "밀도 회귀의 정밀도"와 "CGFE 같은 무거운 멀티스케일 모듈 없이도 효율적"이라는 두 가지 우위를 동시에 보여줌. D3Q는 pretrained weight 없이 from scratch로 학습해도(30.2) DQ-DETR의 pretrained 2단계 학습 결과(30.2)와 동등한 성능을 낸다는 점도 저자가 강조.
>
> #### Table IV — 메인 ablation (Deformable-DETR 기준, AI-TOD-v2)
> **보는 법**: IDE&QA / Anchor L1 Measure / DMS를 순서대로 추가하며 AP가 어떻게 개선되는지 확인.
>
> | 구성 | AP | AP_vt | AP_t | AP_s | AP_m | FPS |
> |---|---|---|---|---|---|---|
> | Deformable DETR(baseline) | 17.8 | 5.4 | 17.2 | 23.5 | 32.2 | 23.4 |
> | + IDE & QA | 22.4(+4.6) | 8.7(+3.3) | 22.6(+5.4) | 27.3(+3.8) | 33.9(+1.7) | 23.8(+0.4) |
> | + Anchor L1 Measure | 23.3(+5.5) | 10.2(+4.8) | 23.5(+6.3) | 27.5(+4.0) | 34.2(+2.0) | 23.8(+0.4) |
> | + DMS | 23.9(+6.1) | 11.3(+5.9) | 24.3(+7.1) | 27.9(+4.4) | 35.2(+3.0) | 23.8(+0.4) |
>
> IDE&QA(dynamic query 메커니즘 핵심)의 기여가 가장 크고(+4.6 AP), 세 요소 모두 FPS 저하를 거의 유발하지 않으면서(+0.4) 누적 개선.
>
> #### Table V — Balance factor T 민감도
> **보는 법**: T값(2~6)에 따른 AP·AP_s 변화를 비교 — T=3에서 precision이 가장 좋음을 확인.
> T=2(18.5)~T=3(**18.9** AP, **27.0** AP_s 최고)~T=6(18.8) — 이상적으로는 양성 샘플 비율 0.25에 대응, 일반 2-stage detector의 positive sampling 비율 관례와 일치.
>
> #### Table VI, VII — Loss 가중치(λ3, λ_density) 및 density loss 종류 민감도
> **보는 법**: λ_density를 늘리면 IDE 학습이 향상되는지, λ3를 과하게 높이면 분류-회귀 균형이 무너지는지 확인.
> λ_density=2에서 최적(AP 19.7). λ3=1 채택(λ3가 과도하게 크면 회귀에 지나치게 치중해 분류와의 균형이 깨짐). Table VII: DFL(19.8/49.1/19.5/24.0/28.8) > L2 loss(16.8) > baseline(17.8) — L2 loss는 오히려 baseline보다 나쁨, density map 회귀 정확도가 낮아 query 상한을 초과하는 문제가 원인.
>
> #### Table VIII — Dynamic query 메커니즘의 효과와 복잡도
> **보는 법**: 고정 query 수(1000/2000)와 dynamic query의 AP·FPS·GFLOPs를 비교.
> Deformable-DETR(query 2000) AP 21.1이지만 FPS 21.5로 저하 vs Deformable-DETR w/ D3Q(Dynamic) AP **23.9**, FPS 23.8, GFLOPs 140 — 단순히 query를 늘린 것보다 dynamic 조정이 더 효율적. DINO에서도 유사 패턴(query 2000 AP 29.5 vs Dynamic AP **32.1**, GFLOPs 520대 543 유사하지만 성능 차이 큼).
>
> #### Fig. 8 — Gradient norm KDE 분포
> **보는 법**: positive/negative 샘플의 gradient L2 norm 분포를 baseline과 D3Q로 비교 — D3Q의 분포가 더 좁고 이상치가 적은지 확인.
> D3Q의 positive 샘플 gradient 분포가 baseline보다 좁고 왼쪽으로 치우쳐(더 일관된 최적화 방향), negative 샘플도 이상치가 적어 sparse 장면에서 무효한 negative query의 간섭이 줄어듦을 시사.
>
> #### Fig. 9 — Query feature t-SNE 시각화
> **보는 법**: 무작위 선택한 500장의 마지막 decoder layer query feature를 t-SNE로 투영, 카테고리별 색상으로 구분.
> Baseline(a)은 산발적이고 뒤섞인 분포인 반면 D3Q(b)는 카테고리별로 뚜렷이 분리된 군집을 형성 — 다만 windmill(전체의 0.08%)처럼 극소수 카테고리는 D3Q에서도 선택된 이미지에 나타나지 않을 만큼 희소함.
>
> #### Fig. 10 — Label assignment 시각화 (IoU vs 샘플 수)
> **보는 법**: positive/negative 샘플의 IoU 분포를 baseline과 D3Q로 비교 — 두 분포가 잘 분리될수록 label assignment가 명확하다는 뜻.
> D3Q(b)가 baseline(a)보다 positive-negative 분포 간 겹침이 적어, "IoU가 GT와 가까운데도 negative로 라벨링되는" 모호한 샘플이 줄었음을 시사.
>
> #### Fig. 11 — 예측 객체 수 vs 실제 GT 수 산점도
> **보는 법**: x축 GT 인스턴스 수, y축 IDE의 예측 개수 — 대각선(예측=실제)에 가까울수록 밀도 추정이 정확하다는 뜻.
> 대부분의 점이 대각선 근처에 분포해 강한 양의 상관관계를 보이며, IDE가 실제 밀도 변화를 효과적으로 반영함을 확인.
>
> #### Table IX — DOTA-v2·VisDrone2019 일반화
> **보는 법**: 객체 크기 분포가 AI-TOD-v2보다 훨씬 큰(예: VisDrone 평균 35.8px vs AI-TOD-v2 12.7px) 데이터셋에서도 D3Q가 일관되게 개선하는지 확인.
> DOTA-v2: DINO 39.2→DINO-D3Q 39.6(+0.4). VisDrone2019: DINO 28.3→DINO-D3Q 29.4(+1.1), DINO-D3Q*(2단계) 36.7(+8.4) — tiny object 특화 설계임에도 더 큰 객체 스케일 데이터셋에서도 견고하게 작동.

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- IDE의 density map을 잘못된 loss(L2)로 학습하면 오히려 baseline보다 나빠짐 → <mark style="background: #FF5582A6;">Table VII에서 L2 loss 사용 시 AP가 16.8로 baseline(17.8)보다도 낮아지는데, 저자는 이를 "density map의 낮은 정확도로 인해 적분으로 얻은 객체 수가 설정한 query 상한을 초과해, query의 동적 조정 자체를 방해하기 때문"이라 설명 — 밀도 추정의 정확도가 무너지면 dynamic query 메커니즘 전체가 역효과를 낼 수 있음을 스스로 보여준다.</mark>
- Anchor L1 measure가 너비·높이에만 적용되고 중심 좌표(x,y)에는 표준 L1이 그대로 쓰임 → <mark style="background: #FF5582A6;">"정리" 표의 두 번째 문제(정규화 좌표의 약한 supervision)가 좌표 전체가 아니라 크기(w,h) 축에서만 완화되며, 중심 위치 자체의 미세 오차에 대한 gradient 약화는 이 논문에서 다루지 않는다.</mark>

### 한계
- <mark style="background: #FF5582A6;">저자가 Discussion(Section V)에서 직접 명시: IDE 모듈의 정확도에 아직 개선 여지가 있으며, 학습·추론 효율은 입력 feature map을 pruning하는 방식(Salience DETR 등)과 결합해 더 개선할 수 있다고 인정 — 현재 버전은 이런 효율화를 포함하지 않는다.</mark>
- windmill 같은 극소수 카테고리(AI-TOD-v2 전체의 0.08%)는 Fig. 9의 시각화 샘플(500장)에서 아예 등장하지 않을 만큼 희소해, 이런 극단적 few-shot 카테고리에서 D3Q의 효과가 실제로 어떤지는 정성적으로도 확인되지 않는다.
- DOTA-v2·VisDrone2019처럼 tiny object가 아닌 데이터셋에서는 개선폭이 상대적으로 작다(DOTA-v2 +0.4, VisDrone +1.1 vs AI-TOD-v2 +3.6) — 저자도 "VisDrone2019의 평균 객체 크기가 AI-TOD-v2보다 훨씬 크고, 불완전한 tiny object 라벨링이 AP_vt·AP_t의 변동을 유발할 수 있다"고 언급.

### 생각할 점
- <mark style="background: #A6E3A1A6;">이 논문은 이 위키의 dynamic query DETR 계열([[DQ-DETR]] 등) 중, "밀도 추정 방식"(분류 vs 회귀)이라는 축에서 DQ-DETR과 정면으로 대조되는 실증 실험(Table III)을 제공하는 유일한 사례다 — 같은 DINO baseline, 같은 query 개수(543 vs 903 등)로 직접 비교해 "연속 회귀가 이산 분류보다 낫다"는 주장을 명확한 수치로 뒷받침한다.</mark>
- <mark style="background: #A6E3A1A6;">Anchor L1 measure는 query 개수·밀도와는 완전히 독립적인 축(loss 설계)의 개선이라는 점이 흥미롭다 — 이 논문이 "밀도 기반 query 개수 조정"과 "회귀 loss 재설계"라는 서로 다른 두 문제를 한 논문에서 함께 다뤘다는 점은, 이 두 개선이 서로 배타적이지 않고 임의의 다른 dynamic query DETR에도 독립적으로 이식 가능함을 시사한다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">Anchor L1 measure(log-ratio 기반 상대 오차)를 [[DQA-DETR]]·[[DQP-DETR]] 등 다른 dynamic query DETR 계열에도 그대로 이식할 수 있어 보인다 — 이 loss 설계는 query 개수 결정 메커니즘과 독립적이므로, 이 위키의 다른 논문들의 박스 회귀 손실을 이걸로 교체했을 때 추가 개선이 있는지 검토할 가치가 있다.</mark>
- <mark style="background: #A6E3A1A6;">저자가 Discussion에서 직접 언급한 "Salience DETR과의 결합 가능성"(density map을 salience filtering에도 활용)은, feature map 자체의 계산량을 줄이는 방향으로 이 논문을 확장하는 자연스러운 다음 단계로 보인다.</mark>

> [!info] 내 메모
> 

# 관련 개념
- [[Density_Guided_Dynamic_Query]] — 이 논문의 핵심 기여(IDE, 연속 회귀 방식)가 등재된 개념. DQ-DETR의 이산 분류를 회귀로 대체한 두 번째 사례로 기록.
- [[1x1_Convolution]] — IDE density head의 마지막 채널 변환에 사용.
- [[Bipartite_Matching_Hungarian_Algorithm]] — 예측-정답 매칭에 anchor L1 measure를 cost로 사용하는 Hungarian 알고리즘의 기반.

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]] — dynamic query DETR 계열 중 "이산 분류 vs 연속 회귀" 축을 대표하는 사례. [[DQ-DETR]]과 동일 baseline·유사 조건으로 직접 비교된다는 점에서 비교 문서 갱신 시 최우선 반영 대상.

# 읽어볼 만한 논문
- 참고문헌 기반: Y.-X. Huang, H.-I. Liu, H.-H. Shuai, W.-H. Cheng, "DQ-DETR: DETR with dynamic query for tiny object detection" (ECCV 2024) [22] — 이미 위키에 있음: [[DQ-DETR]]. 이 논문이 Table III에서 직접 대조 실험하는 핵심 비교 대상.
- 참고문헌 기반: H. Zhang et al., "DINO: DETR with improved denoising anchor boxes for end-to-end object detection" (ICLR 2023) [18] — 이 논문의 최종 SOTA 실험(32.1% mAP)이 기반하는 baseline. Mixed selection(static+dynamic query 결합)의 원조 아이디어를 제공.
- 참고문헌 기반: C. Xu, J. Wang, W. Yang, H. Yu, L. Yu, G.-S. Xia, "RFLA: Gaussian receptive field based label assignment for tiny object detection" (ECCV 2022) [3] — CNN 계열 baseline 비교의 최고 성능 방법. 이 논문의 density Gaussian target 생성 방식(Fig. 4)과 비교하며 읽으면 유용.
- 자유 추천(검증 필요): Salience DETR 계열의 feature pruning/filtering을 density map 기반 dynamic query와 결합한 연구 — 검색 키워드: `salience filtering DETR encoder pruning density map efficient tiny object detection`. 저자가 Discussion에서 직접 제안한 향후 방향이라 실제 후속 연구가 있는지 확인할 가치가 있음.
