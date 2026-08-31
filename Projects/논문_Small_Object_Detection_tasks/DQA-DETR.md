---
pm-task: true
projectId: "paperwiki-small-object-detection"
parentId:
id: "t-dqa-detr-8sx57vf3n4"
title: "DQA-DETR: Dynamic Query Aggregation for Oriented Object Detection in Remote Sensing Images"
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
  "1frf59rymtcjvske": "IEEE JSTARS"
subtaskIds: []
dependencies: []
year: 2026
venue: "IEEE Journal of Selected Topics in Applied Earth Observations and Remote Sensing (JSTARS)"
jcr_quartile: Q1
task: [small-object-detection]
direction: [improvement]
paper_tags: [paper, small-object-detection, oriented-object-detection, detr, dynamic-query, remote-sensing, query-aggregation, label-assignment]
source: "/Users/GyeongSeo/Workspace/논문_pdf/Small_Object_Detection/2026_JSTARS_DQA-DETR.pdf"
createdAt: "2026-08-24T03:17:00.000Z"
updatedAt: "2026-08-28T18:20:00.000Z"
---

Project: [[논문_Small_Object_Detection|Small Object Detection]]
#paper #small-object-detection #oriented-object-detection #detr #dynamic-query #remote-sensing #query-aggregation #label-assignment

> [!quote] 원제
> **DQA-DETR: Dynamic Query Aggregation for Oriented Object Detection in Remote Sensing Images**
> Yongchen Yao, Songwei Pei, Yuanzhou Huang, Qian Li, Shangguang Wang — Beijing University of Posts and Telecommunications, IEEE JSTARS 2026

# 한 줄 요약
<mark style="background: #FFF3A3A6;">Rotated-DINO 기반 oriented object detection에서, dense query를 줄이는 대신 유사한 고품질 query를 "병합(aggregate)"하는 방식으로 one-to-one matching의 중복 고품질 음성 샘플(high-quality negative) 문제를 해결하는 DQA-DETR — 이미지별 밀도 수준(4단계 분류)으로 병합 중심(aggregation center) 개수의 사전(prior)만 제공하고, class-agnostic rotated-NMS로 대표 중심을 선별한 뒤, multi-head cross-attention으로 나머지 dense query의 정보를 중심에 흡수시켜 DOTA-v1.0/v1.5에서 baseline 대비 mAP +2.70/+2.74%p를 달성한 원격탐사 방향성 객체 탐지 논문.</mark>

> [!info] 내 메모
> 

# 정리

| | 문제 ① — One-to-one matching의 고품질 음성 샘플 | 문제 ② — 고정 query 수의 밀도 불일치 |
|---|---|---|
| **문제 정의** | DETR 계열은 GT당 정확히 하나의 query만 양성으로 매칭하는데, 동일 GT 근처에 공간적으로 유사한 고품질 query가 여럿 있으면 나머지는 모두 음성(배경)으로 강제 라벨링된다. 저자들은 focal loss의 gradient를 직접 유도해(Eq. 1-3), 이런 "중복 고품질 negative"가 gradient 구조를 왜곡하고 학습 방향을 오염시킴을 이론적으로 보인다(Fig. 3, gradient ratio가 p>0.5 구간에서 음수로 전환). | DOTA-v1.0은 이미지당 인스턴스 수가 1~1939개로 극단적 편차를 보인다. 소수 query는 밀집 소형 객체의 recall을 낮추고, 다수 query는 희소한 대형 객체 장면에서 중복 예측(redundant box)을 유발한다. |
| **풀고자 하는 문제** | One-to-one matching에서 발생하는 중복 고품질 negative의 gradient 왜곡 문제를 정보 손실 없이 완화하는 것 | 이미지별 객체 밀도에 맞게 병합될 대표 query(aggregation center) 개수를 동적으로 정하는 것 |
| **선행 연구 접근** | - DN-DETR/Group DETR/CO-DETR: 양성 샘플을 늘려 수렴 가속 — 유사 query 간 경쟁 자체는 해소하지 못함<br>- DDQ-DETR: class-agnostic NMS로 추론 시 중복을 제거 — 유사 query를 버릴 뿐 정보 활용 못 하고 적응성 없음<br>- EMO2-DETR: negative 재배정 + 추론 시 NMS로 완화 — end-to-end 설계 훼손 | - DQ-DETR([[DQ-DETR]]): 이미지 레벨 밀도로 query 수 자체를 조정 — 학습 불안정성과 연산 오버헤드가 큼<br>- AO2-DETR, O2-DETR, ARS-DETR, D2Q-DETR: 회전 인식 모듈·query 설계 개선 — 원격탐사 특유의 밀도 편차(sparse·dense 동시 대응)는 대부분 미해결 |
| **해결 방법** | ACS(Aggregation Center Selector)가 class-agnostic rotated-NMS로 유사 query 중 대표만 남겨 Hungarian matching에 진입하는 query 수 자체를 줄이고, QA(Query Aggregator)가 선택 안 된 query의 정보를 attention으로 대표 query에 흡수시켜 "제거"가 아닌 "병합"으로 정보 보존 | ACP(Aggregation Center Predictor)가 이미지를 인스턴스 수 기준 4구간(0/1~9/10~99/100+)으로 분류해, 병합 후 남길 대표 query 개수(300/500/900/1200)의 대략적 사전(prior)만 제공 — 최종 검출 개수를 직접 정하지 않는다는 점이 DQ-DETR과 다름 |
| **예상되는 문제점** | R-NMS가 미분 불가능한 연산이라 선택 단계 자체에는 gradient가 흐르지 않아, 선택 규칙 자체는 고정된 휴리스틱(confidence+IoU)에 의존 | ACP의 밀도 예측이 전역 feature 통계에만 의존해, 극도로 밀집되거나 초대형 이미지에서 일반화가 제한될 수 있음(저자 인정) |

**갭 종합**: <mark style="background: #FFF3A3A6;">기존 dynamic query 연구(DQ-DETR)는 query를 "몇 개 쓸지"만 조정했지, 유사 query 간 경쟁 자체(고품질 negative 문제)를 직접 다루지 않았다. DDQ-DETR은 중복 제거는 하지만 버려지는 query의 정보를 활용하지 못한다. "제거"가 아니라 "병합"을 통해 정보를 보존하면서 동시에 밀도 적응적인 방법은 없었다는 것이 이 논문의 통찰이다.</mark>

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">Dense query를 그대로 두되, <span style="color:#c0392b; font-weight:bold;">Aggregation Center Predictor(ACP)</span>가 이미지 밀도 수준으로 병합 중심 개수의 대략적 사전(prior)을 제공하고, <span style="color:#c0392b; font-weight:bold;">Aggregation Center Selector(ACS)</span>가 class-agnostic rotated-NMS로 대표 중심을 뽑은 뒤, <span style="color:#c0392b; font-weight:bold;">Query Aggregator(QA)</span>가 multi-head attention으로 나머지 dense query의 정보를 그 중심에 흡수시킨다 — "제거"가 아니라 "병합"이라는 점이 DDQ-DETR과의 핵심 차이.</mark>

## 전체 파이프라인 (Fig. 2 기준)

```
입력 이미지
       │
       ▼
① CNN Backbone + Deformable Encoder      → multi-scale feature (d, h_i, w_i)
       │
       ├──────────────────────────────┐
       ▼                              │
② Aggregation Center Predictor (ACP)   │
   (최고해상도 feature → 분류 head)     │  → num_centers ∈ {300,500,900,1200}
       │                              │
       ▼                              ▼
③ Rotated-DINO Decoder Layers × N (중간~후반 레이어까지 진행)  → dense query Q_d (M, d)
       │
       ▼
④ Aggregation Center Selector (ACS)
   Confidence 순위 → Rotated-NMS(임계값 0.8) → Refill(부족분 confidence 순 보충)
       │                                          → aggregation center Q_c (num_centers, d)
       ▼
⑤ Query Aggregator (QA), N층 반복
   Self-Attn(Q_c) → Cross-Attn(Q_c query, Q_d key/value) → FFN
       │                                          → 병합된 Q_c (num_centers, d)
       ▼
⑥ 나머지 Rotated-DINO Decoder Layers (Q_c 입력)  → 최종 decoder 출력
       │
       ▼
⑦ Prediction Head                        → 클래스 + oriented box
       │
       ▼ (학습 시, one-to-one Hungarian matching)
⑧ Loss: L_cls(focal) + L_box(rotated IoU + L1)
```

> [!info] 내 메모
> 

### ① CNN Backbone + Deformable Encoder
- **역할**: 입력 이미지에서 다중 스케일 feature를 추출하고 encoder로 문맥을 반영한다. Rotated-DINO의 표준 구조를 그대로 사용하며, 이 논문의 기여는 이 이후 decoder 안에 삽입되는 ACS·QA에 있다.
- **구현**: CNN backbone(ResNet-50 또는 Swin-T) + deformable encoder([[Multi_Head_Self_Attention]]의 변형, 소수 샘플링 위치만 참조).
- **입출력 shape**: 입력 이미지 → multi-scale feature `(d, h_i, w_i)`.

> [!info] 내 메모
> 

### ② Aggregation Center Predictor (ACP)
- **역할**: encoder feature로부터 "이 이미지에 객체가 대략 몇 개나 있는지"를 대략적으로 가늠해, 뒤이어 ACS가 몇 개의 대표 query를 남길지에 대한 **거친 사전(coarse prior)**을 제공한다. DQ-DETR의 CCM과 마찬가지로 회귀 대신 분류를 쓰지만, 여기서는 이 출력이 "최종 검출 개수"가 아니라 어디까지나 ACS를 위한 참고값일 뿐이라는 점이 근본적으로 다르다 — one-to-one Hungarian matching도, 최종 detection 출력도 직접 건드리지 않는다.
- **구현**: DINO multiscale feature 중 최고해상도 feature map을 2차원으로 재구성(reconstruct)한 뒤, 경량 분류 head(Cls Score → FFN, "ACP Head")에 통과시켜 이미지를 인스턴스 수 기준 4개 구간(count=0, 1~9, 10~99, 100+)으로 분류한다. 각 구간은 미리 정해둔 병합 중심 개수(300/500/900/1200, "ACP List")에 대응한다.
- **입출력 shape**: 최고해상도 encoder feature `(d, h_1, w_1)` → 분류 head → 4-class 분류 결과 → `num_centers ∈ {300, 500, 900, 1200}`.

```python
# 논문 Section III.C 기반 의사코드
F_hires = reconstruct_2d(encoder_multiscale_feature, level=1)   # 최고해상도, 2D로 재구성
logits = ACP_head(F_hires)                                        # 경량 분류 head (Cls Score -> FFN)
count_class = argmax(logits)                                       # 0 / 1~9 / 10~99 / 100+
num_centers = {0: 300, 1: 500, 2: 900, 3: 1200}[count_class]       # ACP List
```

(연한 노랑 하이라이트) <mark style="background: #FFF9D6A6;">"정리" 표의 두 번째 문제(고정 query 수의 밀도 불일치)를, ACP가 만든 대략적 사전으로 ACS의 선택 강도를 데이터 의존적으로 조정함으로써 완화한다 — 정확한 개수 추정을 목표로 하지 않고 "coarse guidance"로만 기능하게 설계한 것은, 부정확한 카운팅이 파이프라인 전체를 오염시키는 DQ-DETR류의 위험을 구조적으로 줄이려는 의도로 읽힌다.</mark>

> [!info] 내 메모
> 

### ③④ Rotated-DINO Decoder(중간까지) + Aggregation Center Selector (ACS)
- **역할**: Decoder가 중간~후반 레이어까지 query를 정제한 시점에서, 이 dense query들 중 "같은 GT 객체를 가리키는 중복 고품질 query"의 대표만 남기고 나머지는 병합 대상으로 남긴다.
- **구현**: 모든 decoder query를 classification confidence로 순위를 매긴 뒤, class-agnostic rotated-NMS(R-NMS, 억제 임계값 0.8)로 같은 GT 주변에 몰린 고품질 query 중 대표만 남긴다. ACP가 정한 목표 개수(`num_centers`)에 남은 query가 못 미치면 confidence 순으로 보충(Refill)해 개수를 맞춘다. R-NMS는 미분 불가능한 연산이라 gradient가 흐르지 않지만, 선택된 query가 이후 decoder layer로 전달되어 최종 detection loss로 최적화되므로 전체 파이프라인은 end-to-end 유지. 삽입 위치는 decoder의 중간~후반 레이어 — semantic이 충분히 안정화되었지만 완전히 수렴하기 전 시점을 실험적으로 선택.
- **입출력 shape**: dense query `Q_all (M, d)` → confidence 순위 + R-NMS + Refill → aggregation center `Q_c (num_centers, d)` + 나머지 dense query `Q_d (M-num_centers, d)`.

```python
# 논문 Section III.D 기반 의사코드
scores = classification_confidence(Q_all)                    # (M,)
ranked = sort_by_score(Q_all, scores)
kept = rotated_nms(ranked, iou_threshold=0.8, class_agnostic=True)   # R-NMS, non-differentiable
if len(kept) < num_centers:
    kept = refill_by_confidence(kept, ranked, target=num_centers)     # 부족분 confidence순 보충
Q_c = kept[:num_centers]           # aggregation center
Q_d = Q_all - Q_c                  # 나머지 dense query (key/value로 재사용)
```

(연한 노랑 하이라이트) <mark style="background: #FFF9D6A6;">"정리" 표의 첫 번째 문제(고품질 negative의 gradient 왜곡)를, 유사 query 중 대표만 남겨 이후 Hungarian matching에 진입하는 query 수 자체를 줄임으로써 완화한다 — semantic 신뢰도(confidence)와 공간적 다양성(NMS)을 함께 고려해 "같은 GT를 가리키는 중복"만 정확히 골라낸다는 점이 단순 top-K confidence 선택과 다르다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> R-NMS가 미분 불가능한 연산이라 선택 단계 자체에는 gradient가 흐르지 않는다 — 저자도 "class-agnostic R-NMS는 학습 단계에 삽입된 중간 query 선택 단계이지 후처리가 아니며, gradient 계산에는 참여하지 않는다"고 명시한다. 즉 선택 자체는 학습되지 않고 고정 규칙(confidence+IoU)에 의존한다는 근본적 한계가 남는다.

> [!info] 내 메모
> 

### ⑤ Query Aggregator (QA)
- **역할**: ACS가 선별한 대표 중심 `Q_c`가 선택되지 않은 나머지 dense query `Q_d`로부터 정보를 "흡수"해 더 풍부한 표현이 되도록 만든다 — 버려지는 대신 흡수된 dense query 정보가 최종 query 표현에 남아, "제거하면 정보 손실, 다 남기면 경쟁 심화"라는 trade-off를 완화한다.
- **구현**: 3단계로 구성(각 단계 residual+LayerNorm). (1) Self-attention: `Q_c`끼리 서로 참고해 대표 query 간 구조적 관계를 모델링. (2) Cross-attention: `Q_c`를 query로, 선택 안 된 dense query `Q_d`를 key·value로 삼아 [[Multi_Head_Self_Attention]] 계산 — 유사한 dense query로부터 정보를 흡수. (3) FFN으로 표현력 강화. 이 3단계를 `N`층(기본 3층) 반복.
- **입출력 shape**: `Q_c (num_centers, d)` + `Q_d (M-num_centers, d)` → self-attn → cross-attn(query=Q_c, key/value=Q_d) → FFN → 병합된 `Q_c (num_centers, d)` (shape 불변, 값만 갱신).

```python
# 논문 Eq.(6)-(8) 기반 의사코드
Q_c1 = LayerNorm(Q_c + MHA_self(Q_c, Q_c, Q_c))              # Eq.6, self-attention
Q_c2 = LayerNorm(Q_c1 + MHA_cross(query=Q_c1, key=Q_d, value=Q_d))  # Eq.7, cross-attention(흡수)
Q_c_final = LayerNorm(Q_c2 + FFN(Q_c2))                        # Eq.8
# 위 3단계를 N층(기본 3) 반복
```

(연한 노랑 하이라이트) <mark style="background: #FFF9D6A6;">"정리" 표에서 R-NMS 단독으로는 정보를 그냥 버리게 되는 한계를, ACS가 제거한 것이 아니라 "선택되지 않은" query들을 QA의 key/value로 재활용함으로써 해결한다 — Table III ablation에서 QA를 제거하면 AP50이 78.29→76.89로 하락해, 병합 자체가 단순 선택보다 더 컴팩트하고 표현력 있는 query를 만든다는 것을 정량적으로 뒷받침한다.</mark>

> [!info] 내 메모
> 

### ⑥⑦⑧ 나머지 Decoder + Prediction Head + Loss
- **역할**: QA가 병합해 만든 compact query `Q_c`를 나머지 decoder layer에 통과시켜 최종 (클래스, oriented box)를 예측하고, one-to-one Hungarian matching으로 정답과 매칭한 뒤 focal loss(분류)+rotated IoU loss+L1(박스)로 학습한다.
- **구현**: Rotated-DINO의 표준 decoder·prediction head·loss를 그대로 사용(이 부분은 이 논문의 기여가 아님). Loss 가중치: rotated IoU loss 2.0, focal loss 1.0, L1 loss 5.0.
- **입출력 shape**: `Q_c (num_centers, d)` → 나머지 decoder → 클래스 + oriented box(5-parameter: x,y,w,h,θ).

```python
# 표준 DETR-style loss, Rotated-DINO 그대로
sigma = hungarian_algorithm(cost_matrix(preds, targets))   # one-to-one 매칭
L_total = 1.0 * focal_loss(cls, gt_cls) + 2.0 * rotated_iou_loss(box, gt_box) + 5.0 * L1(box, gt_box)
```

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| ① Backbone+Encoder | 입력 이미지 | (d,h_i,w_i) | 다중 스케일 feature 추출 | ResNet-50/Swin-T + deformable encoder |
| ② ACP | 최고해상도 feature (d,h_1,w_1) | num_centers∈{300,500,900,1200} | 밀도 분류 → 병합 개수 사전 제공 | 경량 분류 head(FFN) |
| ③④ Decoder(중간) + ACS | dense query Q_all (M,d) | Q_c(num_centers,d) + Q_d(M-num_centers,d) | 대표 query 선별(제거 아님) | Confidence 순위 + Rotated-NMS(0.8) + Refill |
| ⑤ QA | Q_c + Q_d | Q_c(num_centers,d) | 선택 안 된 query 정보 흡수 | Self-Attn+Cross-Attn+FFN, N층(기본 3) |
| ⑥⑦ Decoder(나머지)+Head | Q_c(num_centers,d) | 클래스+oriented box | 최종 검출 예측 | Rotated-DINO 표준 decoder/head |
| ⑧ Loss | 예측 + 정답 | 스칼라 loss | 학습 신호 결정 | Hungarian + Focal + Rotated IoU + L1 |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table I·V (DOTA-v1.0/v1.5 test, Rotated-DINO baseline 기준)
**보는 법**: NMS-free(트랜스포머 계열) 그룹 안에서 baseline(Rotated-DINO, 재현치)과 DQA-DETR을 같은 backbone끼리 비교하면 이 논문의 순수 이득을 볼 수 있다.

| 벤치마크 | 지표 | Before(Rotated-DINO, 재현) | After(DQA-DETR) |
|---|---|---|---|
| DOTA-v1.0 | mAP (ResNet-50) | 75.59 | 78.29 (+2.70%p) |
| DOTA-v1.0 | mAP (Swin-T) | — | 79.37 |
| DOTA-v1.5 | mAP (ResNet-50) | 68.79 | 71.53 (+2.74%p) |
| DOTA-v1.5 | mAP (Swin-T) | — | 72.83 |

> [!note]- 세부 결과 및 Ablation
> #### Table I — DOTA-v1.0 SOTA 비교 (NMS-free 그룹 발췌)
> **보는 법**: NMS-based(2-stage/anchor 계열)와 NMS-free(트랜스포머 계열)로 나뉜 표에서, NMS-free 그룹 안에서 DQA-DETR의 위치를 확인.
> Deformable DETR-O(ResNet-50) 65.62 < ARS-DETR 74.20 < OrientedFormer 74.83 < Rotated-DINO(재현) 75.59 < **DQA-DETR(ResNet-50) 78.29** < **DQA-DETR(Swin-T) 79.37**(NMS-free 최고). 카테고리별로 BR(bridge), SBF(soccer-ball-field), HA(helicopter)처럼 소형·희소 객체가 많은 클래스에서 개선폭이 두드러짐 — redundant query가 고품질 negative를 특히 많이 만드는 클래스로 저자가 해석. 순수 성능은 최고 2-stage 검출기(ReDet, DCFL)에 소폭 못 미치지만, NMS·수작업 후처리 없는 end-to-end 구조를 유지한 채 이 격차를 좁혔다는 점이 기여로 강조됨.
>
> #### Table II — Recall 비교 (DOTA-v1.0 val)
> **보는 법**: 클래스별 recall을 baseline과 나란히 비교 — 병합이 정보를 잃지 않고 오히려 recall을 높이는지 확인.
> small-vehicle 0.921→0.955, storage-tank 0.887→0.901, helicopter 0.974→0.987 등 15개 전 클래스에서 recall 개선.
>
> #### Table V — DOTA-v1.5 SOTA 비교 (Container Crane 클래스 포함, 더 조밀)
> DQA-DETR(ResNet-50) 71.53%, DQA-DETR(Swin-T) **72.83%**(NMS-free 그룹 최고). NMS 기반 2-stage(DCFL 66.86%) 대비도 우위.
>
> #### Table III — 메인 ablation (leave-one-out, DOTA-v1.0/v1.5)
> **보는 법**: ACP/ACS/QA 세 모듈을 하나씩 빼가며 AP50 하락폭을 확인 — 어느 모듈이 얼마나 기여하는지 판단.
>
> | 구성 | AP50(DOTA-v1.0) | AP50(DOTA-v1.5) |
> |---|---|---|
> | Baseline(모듈 없음) | 75.59 | 68.79 |
> | +ACS+QA (w/o ACP, 고정값 사용) | 76.03(+0.44) | 69.21(+0.42) |
> | +ACP+QA (w/o ACS) | 76.68(+1.09) | 69.33(+0.54) |
> | +ACP+ACS (w/o QA) | 76.89(+1.30) | 69.85(+1.06) |
> | +ACP+ACS+QA(전체) | **78.29(+2.70)** | **71.53(+2.74)** |
>
> ACP를 빼고 고정값을 쓰면 AP50이 78.29→76.03으로 가장 크게 하락 — "동적으로 개수를 조정하는 것" 자체가 핵심 기여임을 보여줌. 세 모듈 모두 있을 때만 큰 시너지가 남.
>
> #### Table IV, Fig. 7 — Query 수 vs 성능/메모리
> **보는 법**: query 수를 900→2400으로 늘려가며 baseline과 DQA-DETR의 mAP·메모리 사용량을 나란히 비교.
> Baseline은 query 수가 늘수록 mAP가 급격히 하락(900:75.59→2400:55.29)하는 반면 DQA-DETR은 900~2400 전 구간에서 77~78%대로 안정(900:77.20, 1800:78.38 최고, 2400:77.52) — "query가 많아도 병합으로 견딘다"는 강건성의 직접 증거. 메모리도 baseline보다 완만하게 증가(query 2400에서 오히려 baseline보다 적음, 17308 vs 20873MB) — 병합이 dense query 자체의 유효 수를 줄여 연산도 절감.
>
> #### Table VI — IoU 임계값·QA 층수 ablation
> **보는 법**: R-NMS 임계값(0.5~0.9)과 QA 층수(0~6)를 각각 바꿔가며 AP50 최적점을 확인.
> IoU 임계값: 0.5(77.17)→0.6(77.65)→0.7(78.05)→**0.8(78.29, 채택)**→0.9(77.67) — 낮으면 과도한 병합으로 대표성 부족, 높으면 중복 억제 효과 약화. QA 층수: 0층(76.89)→1층(77.25)→2층(77.92)→**3층(78.29, 채택)**→4층(78.18)→6층(77.83) — 3층 이후로는 소폭 하락(과도한 병합으로 정보 손실/과적합 추정).
>
> #### Table VII — 파라미터·연산 비교 (DOTA-v1.0)
> **보는 법**: Params/GFLOPs 열까지 함께 봐야 "정확도 대비 비용"을 판단할 수 있다.
> Rotated-DINO(baseline) 47.3M/280.38GFLOPs/75.59 vs DQA-DETR(ours) 49.1M/311.62GFLOPs/**78.29** — 파라미터 +1.8M, GFLOPs +31.24(약 +11%)로 연산 증가가 있지만 mAP 개선폭(+2.70) 대비 합리적 수준.
>
> #### Fig. 6 — QA 적용 전후 정성 비교
> **보는 법**: 같은 지역을 QA 적용 전(위)/후(아래)로 비교 — 소형 차량 클래스 혼동이 QA 이후 줄어드는지 확인.

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- R-NMS가 미분 불가능한 연산이라 선택 단계 자체에는 gradient가 흐르지 않음 → <mark style="background: #FF5582A6;">저자도 "class-agnostic R-NMS는 학습 단계에 삽입된 중간 query 선택 단계이지 후처리가 아니며, gradient 계산에는 참여하지 않는다"고 명시 — 선택 자체는 학습되지 않고 고정 규칙(confidence+IoU)에 의존한다는 근본적 한계가 남는다.</mark>
- ACP의 밀도 예측이 전역 feature 통계에만 의존 → <mark style="background: #FF5582A6;">Discussion(Section V)에서 저자가 직접 인정: "병합 중심 개수 예측이 현재 전역 feature 통계에 의존해, 극도로 밀집되거나 초대형 이미지에서 일반화가 제한될 수 있다."</mark>

### 한계
- <mark style="background: #FF5582A6;">저자가 명시: QA의 계산 비용이 대규모 이미지에서 추가 평가가 필요하다("computational cost warrants further evaluation on larger-scale datasets").</mark>
- <mark style="background: #FF5582A6;">최고 성능의 2-stage/anchor 기반 방법(ReDet, DCFL)에는 여전히 못 미침 — 저자도 "복잡한 설계와 다단계 후처리에 의존하는 방법들이 여전히 최고 점수를 보유한다"고 인정하며, 이 논문의 기여를 "성능과 단순성의 균형"으로 재규정.</mark>
- ACS 삽입 위치(중간~후반 decoder layer)가 실험적 관찰에 기반한 선택이며, 다양한 삽입 위치에 대한 세밀한 정량 비교표는 제시되지 않는다(본문 서술로만 정당화).
- R-NMS 임계값 0.8, QA 3층 등 핵심 하이퍼파라미터가 DOTA 데이터셋 특성에 맞춰 튜닝되어 있어, 다른 원격탐사 데이터셋에 대한 일반화는 검증되지 않음.

### 생각할 점
- <mark style="background: #A6E3A1A6;">이 논문은 dynamic query DETR 계열 중 유일하게 oriented(회전) object detection을 다루며, "query 개수 자체를 줄인다"가 아니라 "비슷한 query를 병합해 정보를 보존한 채 개수를 실질적으로 압축한다"는 독자적인 제3의 전략을 취한다 — [[DQ-DETR]]/Density-Aware DETR/IG-DETR의 "밀도로 개수를 정한다", PaQ-DETR의 "패턴+품질로 병합·제거한다"와 비교하면, DQA-DETR은 PaQ-DETR과 "병합"이라는 상위 개념은 공유하지만 제거(pruning) 없이 병합만 한다는 점, 그리고 병합의 주 목적이 "one-to-one matching의 gradient 왜곡을 이론적으로 규명하고 이를 해소하는 것"이라는 점에서 독자적이다.</mark>
- <mark style="background: #A6E3A1A6;">Table IV·Fig. 7에서 baseline은 query 수 증가에 따라 성능이 붕괴(75.59→55.29)하는 반면 DQA-DETR은 안정적으로 유지되는 패턴은, "query를 늘리는 것 자체는 위험하고, 늘린 query를 어떻게 다루는지가 핵심"이라는 dynamic query DETR 계열 전체를 관통하는 메시지를 가장 극적으로 보여주는 실험이다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">Focal loss의 gradient를 직접 유도해 "중복 고품질 negative가 학습을 방해한다"는 것을 이론적으로 보인 접근(Section III.B, Eq. 1-3)은, 이 위키의 다른 dynamic query DETR 논문들이 실험적 관찰에만 의존하는 것과 달리 이론적 근거를 제시한 유일한 사례다 — 이 분석 틀을 [[DQ-DETR]]·IG-DETR 등 다른 계열에도 적용하면 "왜 밀도 기반 query 조정이 작동하는지"에 대한 공통 이론적 기반을 마련할 수 있을 것으로 보인다.</mark>
- <mark style="background: #A6E3A1A6;">QA의 "선택되지 않은 query를 버리지 않고 attention으로 흡수한다"는 아이디어는, 이 위키의 [[PaQ-DETR]]이 하는 "저품질 query를 완전히 제거"하는 것과 정반대 철학이다 — 두 방법을 결합해 "명백한 배경 query는 제거하되, 애매하지만 유용할 수 있는 중복 query는 병합"하는 하이브리드가 가능할지 검토할 가치가 있다.</mark>

> [!info] 내 메모
> 

# 관련 개념
- [[Pattern_Quality_Aware_Query_Refinement]] — PaQ-DETR과 "유사 query를 다룬다"는 상위 목표는 같지만, 이 논문은 병합(merge)만 하고 제거(prune)는 하지 않는다는 점에서 메커니즘이 다르다.
- [[Density_Guided_Dynamic_Query]] — ACP의 4단계 이미지 밀도 분류는 DQ-DETR의 CCM과 구조적으로 유사(회귀 대신 분류, 4단계 구간)하지만, 이 논문에서 ACP의 출력은 "최종 query 수"가 아니라 ACS 단계를 위한 "대략적 사전(prior)"일 뿐이라는 역할 차이가 있다.
- [[Multi_Head_Self_Attention]] — ACS 이전 decoder self/cross-attention, QA의 self-attention·cross-attention 모두의 기반.
- [[Bipartite_Matching_Hungarian_Algorithm]] — 최종 예측-정답 매칭에 사용되는 one-to-one Hungarian matching.

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]] — dynamic query DETR 계열 중 하나. Oriented object detection이라는 점에서 회전 박스를 다루는 유일한 사례이며, "병합(merge)"이라는 제3의 전략을 대표.

# 읽어볼 만한 논문
- 참고문헌 기반: Y.-X. Huang, H.-I. Liu, H.-H. Shuai, W.-H. Cheng, "DQ-DETR: DETR with dynamic query for tiny object detection" (ECCV 2024) [17] — 이미 위키에 추가됨: [[DQ-DETR]]. 이 논문이 "학습 불안정성과 연산 오버헤드"의 사례로 직접 비판하는 대상.
- 참고문헌 기반: S. Zhang, X. Wang, Y. Chen, J. Chen, W. Li, J. Wang, "Dense distinct query for end-to-end object detection" (DDQ-DETR, CVPR 2023) [16] — 이 논문이 "제거"와 "병합"의 차이를 설명할 때 가장 직접적으로 대조하는 선행 연구.
- 참고문헌 기반: G.-S. Xia et al., "DOTA: A large-scale dataset for object detection in aerial images" (CVPR 2018) [19] — 이 논문의 두 주 벤치마크(DOTA-v1.0/v1.5) 원 데이터셋 논문. 원격탐사 oriented object detection 평가 프로토콜의 배경 이해에 필수.
- 자유 추천(검증 필요): Focal loss의 gradient 분석을 다른 dense prediction task의 label assignment 설계에 적용한 연구 — 검색 키워드: `focal loss gradient analysis label assignment redundant positive negative samples theoretical`. 이 논문의 Section III.B 이론적 분석이 detection 외 다른 task(segmentation 등)의 유사 문제에도 적용 가능한지 확인할 가치가 있음.
