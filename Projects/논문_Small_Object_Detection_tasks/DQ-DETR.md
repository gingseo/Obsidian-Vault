---
pm-task: true
projectId: "paperwiki-small-object-detection"
parentId:
id: "t-dq-detr-dkys1fdsbz"
title: "DQ-DETR: DETR with Dynamic Query for Tiny Object Detection"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-08-24"
due:
progress: 0
assignees: []
tags: []
customFields:
  "7l8l795xmtcjvlf6": 2024
  "1frf59rymtcjvske": "ECCV"
subtaskIds: []
dependencies: []
year: 2024
venue: "ECCV"
jcr_quartile: Q1
task: [small-object-detection]
direction: [improvement]
paper_tags: [paper, small-object-detection, tiny-object-detection, detr, dynamic-query, density-map, remote-sensing]
source: "/Users/GyeongSeo/Workspace/논문_pdf/Small_Object_Detection/2024_ECCV_DQ-DETR.pdf"
createdAt: "2026-08-24T03:07:00.000Z"
updatedAt: "2026-08-28T18:00:00.000Z"
---

Project: [[논문_Small_Object_Detection|Small Object Detection]]
#paper #small-object-detection #tiny-object-detection #detr #dynamic-query #density-map #remote-sensing

> [!quote] 원제
> **DQ-DETR: DETR with Dynamic Query for Tiny Object Detection**
> Yi-Xin Huang, Hou-I Liu, Hong-Han Shuai, Wen-Huang Cheng — National Yang Ming Chiao Tung University / National Taiwan University, ECCV 2024

# 한 줄 요약
<mark style="background: #FFF3A3A6;">항공 이미지의 이미지별 인스턴스 수 불균형(1개~2667개)에 대응하기 위해, density map 기반 categorical counting 모듈로 이미지마다 object query 개수(300/500/900/1500) 자체를 다르게 선택하고, 같은 density map으로 encoder feature와 query의 content·position을 함께 강화하는 DQ-DETR을 제안해 AI-TOD-V2에서 기존 SOTA 대비 AP +4.3%p를 달성한 Deformable DETR 기반 tiny object detector.</mark>

> [!info] 내 메모
> 

# 정리

| | 문제 ① — 고정된 query 개수의 부적합성 | 문제 ② — Query 위치의 이미지 무관성 |
|---|---|---|
| **문제 정의** | DETR/Deformable DETR은 각각 K=100/300으로 query 수가 고정되어 있는데, 항공 데이터셋(AI-TOD-V2)은 이미지당 인스턴스 수가 1~2667개까지 극단적으로 편차가 크다(평균 24.64, 표준편차 63.94). 밀집 이미지에서는 query가 부족해 미검출(FN)이 급증하고, 희소 이미지에서는 과잉 query가 오탐(FP)을 유발하며 decoder self-attention의 이차 복잡도로 연산 자원도 낭비된다. | 기존 DETR 계열의 object query 위치는 학습된 임베딩일 뿐 특정 물리적 의미가 없어, 현재 입력 이미지의 실제 객체 분포와 무관하다 — 인스턴스가 특정 영역에 밀집되거나 흩어져 있는 항공 이미지 특성에 대응하지 못한다. |
| **풀고자 하는 문제** | 이미지마다 크게 다른 인스턴스 수에 맞춰 object query 개수를 동적으로 조정하는 것 | Query의 content·position을 현재 이미지의 실제 객체 분포(위치·밀도)에 맞게 강화하는 것 |
| **선행 연구 접근** | - Deformable DETR: Sparse query(K=300), one-to-one assignment — recall이 낮음<br>- DDQ-DETR: Dense distinct query(K=900)로 recall을 일부 완화하지만 여전히 고정된 상한<br>- **갭**: 두 방법 모두 "이미지마다 인스턴스 수가 다르다"는 imbalance 자체는 다루지 않음(Table 1에서 직접 대조) | - Dynamic DETR: decoder에 ROI 기반 dynamic attention 도입<br>- DN-DETR: denoising training으로 이분 매칭 불안정성 완화<br>- Conditional DETR, DAB-DETR: query를 content+position(4D anchor box)으로 분해해 물리적 의미 부여 — 그러나 query 개수는 데이터셋 전체에 대해 고정<br>- **갭**: query의 형식(형태·초기화)은 개선했지만 "몇 개를 쓸지"는 다루지 않음 |
| **해결 방법** | Categorical Counting Module(CCM)이 density map을 4단계로 분류해 decoder에 투입할 query 개수(K=300/500/900/1500)를 이미지별로 선택 | Counting-Guided Feature Enhancement(CGFE)로 density map을 encoder feature에 spatial+channel attention으로 주입한 뒤, 이 강화 feature에서 top-K를 선별해 query의 content·position을 생성(Dynamic Query Selection) |
| **예상되는 문제점** | CCM의 분류 오류가 이후 파이프라인 전체에 연쇄적으로 영향(특히 N>500 구간처럼 학습 샘플이 적은 구간) | Query 수가 최대 1500까지 늘어나면 decoder self-attention의 이차 복잡도 문제가 재발할 위험이 있으나 논문은 FLOPs·FPS를 보고하지 않음 |

**갭 종합**: <mark style="background: #FFF3A3A6;">DETR 계열의 query 형식 개선(문제②의 선행 연구)과 sparse/dense 전략(문제①의 선행 연구) 모두 "이미지마다 인스턴스 수가 다르다"는 imbalance를 정면으로 다루지 않았다. Query의 "개수"와 "위치" 둘 다를 입력 이미지 내용에 따라 동적으로 조정하는 DETR 계열은 없었다는 것이 이 논문의 통찰이다.</mark>

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">Deformable DETR 위에 <span style="color:#c0392b; font-weight:bold;">categorical counting 모듈</span>로 이미지의 인스턴스 수를 4단계로 분류해 query 개수 자체를 선택하고, 같은 모듈이 만든 density map으로 <span style="color:#c0392b; font-weight:bold;">encoder feature를 spatial+channel attention으로 보강</span>한 뒤, 이 보강된 feature에서 top-K를 선별해 <span style="color:#c0392b; font-weight:bold;">query의 content와 position(anchor box bias)을 동시에 생성</span>한다.</mark>

## 전체 파이프라인 (Fig. 1 기준)

```
입력 이미지
       │
       ▼
① CNN Backbone + Deformable Encoder      → multi-scale feature S_i (d, h_i, w_i), i=1..l
       │
       ├──────────────────────────────┐
       ▼                              │
② Categorical Counting Module (CCM)    │
   (S_1에 dilated conv → density map F_c → 분류) → F_c (1, h_1, w_1) + K ∈ {300,500,900,1500}
       │                              │
       ▼                              ▼
③ Counting-Guided Feature Enhancement (CGFE)
   F_c를 각 레벨에 맞춰 축소 → spatial attention → channel attention
       │                                          → F_t,i (d, h_i, w_i)
       ▼
④ Dynamic Query Selection (DQS)
   F_t flatten → top-K(②의 K) 선별 → Q_content, Q_position(4D anchor+bias)
       │                                          → Q_content(K,d), Q_position(K,4)
       ▼
⑤ Deformable Decoder (Q_content, Q_position, encoder memory)  → decoder 출력 (K, d)
       │
       ▼
⑥ Prediction Head                        → 클래스 + 박스 (K개)
       │
       ▼ (학습 시)
⑦ Loss: L_hungarian(L1+GIoU+Focal) + L_aux + L_counting
```

> [!info] 내 메모
> 

### ① CNN Backbone + Deformable Encoder
- **역할**: 입력 이미지에서 다중 스케일 feature를 추출하고, Deformable DETR의 encoder(전체 픽셀이 아니라 학습된 소수 샘플링 위치만 참조하는 attention, 자세한 동작은 [[Multi_Head_Self_Attention]] 참고)로 문맥을 반영한 feature `S_i`로 갱신한다. 이후 CCM이 dilated convolution을 적용할 수 있도록, flatten된 1차원 시퀀스 feature를 다시 2차원 feature map으로 복원("Unflattening")하는 절차가 추가된다 — 이렇게 복원한 feature를 논문은 EMSV(reconstructed Encoder's Multi-Scale Visual features)라 부른다.
- **구현**: ResNet-50 backbone + 5-scale feature map(backbone stage 1~4 + stage 4를 다운샘플링한 추가 레벨) + 6층 deformable encoder.
- **입출력 shape**: 입력 이미지 → multi-scale feature `P_i (d, h_i, w_i)` → flatten+concat → encoder 통과 → 다시 unflatten한 `S_i (d, h_i, w_i)`, `i=1..l`(l=5).

> [!info] 내 메모
> 

### ② Categorical Counting Module (CCM)
- **역할**: EMSV feature 중 해상도가 가장 높은 레벨(`S_1`, tiny object의 공간 디테일이 가장 잘 보존된 레벨)만으로 이미지 전체의 "밀도 지도(density map)"를 만들고, 그 지도를 근거로 "이 이미지에 객체가 대략 몇 개나 있는지"를 4개 구간 중 하나로 분류해 decoder에 투입할 query 개수 `K`를 결정한다.
- **구현**: `S_1`을 density extractor(dilated convolution을 반복 적용해 넓은 수용영역으로 tiny object의 장거리 의존성을 포착, 자세한 동작은 [[Dilated_Convolution]] 참고)에 통과시켜 density map `F_c`를 얻는다. `F_c`를 average pooling 후 2-layer FFN(분류 head)에 통과시켜 인스턴스 수 N을 `N≤10 / 10<N≤100 / 100<N≤500 / N>500` 4구간으로 분류하고, 각 구간을 각각 `K=300/500/900/1500`에 대응시킨다. 회귀(regression) 대신 분류를 택한 이유는 N이 1~2667까지 극단적으로 편차가 커 정확한 회귀 자체가 어렵고, 회귀 오차가 곧바로 부적절한 query 수로 이어져 학습이 불안정해지기 때문(Table 8에서 회귀 사용 시 AP가 baseline 25.9에서 14.9로 급락 확인).
- **입출력 shape**: `S_1 (d, h_1, w_1)` → dilated conv 반복 → `F_c (1, h_1, w_1)` → AvgPool+FFN → 4-class 분류 결과 → `K ∈ {300, 500, 900, 1500}`.

```python
# 논문 §3.2-3.3 기반 의사코드
F_c = dilated_conv_stack(S_1)              # (d,h1,w1) -> (1,h1,w1), density map
logits = FFN(avg_pool(F_c))                # 4-class 분류 head
N_class = argmax(logits)                    # N<=10 / 10<N<=100 / 100<N<=500 / N>500
K = {0: 300, 1: 500, 2: 900, 3: 1500}[N_class]
```

(연한 노랑 하이라이트) <mark style="background: #FFF9D6A6;">"정리" 표의 첫 번째 문제(고정 query 수의 부적합)를, 이미지 내용으로부터 직접 추정한 밀도 수준에 따라 decoder에 투입할 query 수 자체를 4단계로 이산화해 선택함으로써 해결한다 — Appendix Table A1에서 이 설계가 실제로 희소 이미지의 LRP FP(29.4→25.7)와 밀집 이미지의 LRP FN(75.1→51.5)을 동시에 낮춤을 정량적으로 확인.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> CCM의 분류 정확도는 전체 94.6%이지만, N>500 구간은 학습 샘플이 46장뿐이라 정확도가 56.6%까지 떨어진다(Table 7). 이 구간의 분류 오류는 곧바로 부적절한 query 개수 선택으로 이어져, DQ-DETR의 개선폭이 이 구간에서만 유독 작아지는(+1.2%p, 다른 구간은 +3.6~4.0%p) 원인이 된다 — 저자도 이를 직접 인정.

> [!info] 내 메모
> 

### ③ Counting-Guided Feature Enhancement (CGFE)
- **역할**: CCM이 만든 density map은 "어디에 객체가 밀집해 있는지"를 담고 있으므로, 이 정보를 attention map으로 변환해 encoder feature에 곱하면 전경(foreground) 영역이 명시적으로 강조된다. 즉 CCM이 "몇 개"를 정했다면, CGFE는 "어디"에 대한 정보를 feature 자체에 새겨 넣는 역할이다.
- **구현**: 먼저 `F_c`를 [[1x1_Convolution]]으로 각 encoder 레벨의 채널·해상도에 맞춰 다운샘플링해 `F_c,i`를 얻는다. `F_c,i`에 채널축 average pooling·max pooling을 적용해 concat한 뒤 7×7 conv+sigmoid로 spatial attention map `W_s,i`를 만들고, encoder feature `S_i`에 곱해 spatial-강조 feature `E_i`를 얻는다(CBAM 방식의 spatial attention). 이어서 `E_i`에 공간축 average pooling·max pooling → 공유 MLP → element-wise 합산 → sigmoid로 channel attention map `W_c,i`를 만들어 다시 곱해 최종 강화 feature `F_t,i`를 얻는다(CBAM 방식의 channel attention을 순차 결합).
- **입출력 shape**: `F_c (1, h_1, w_1)` + `S_i (d, h_i, w_i)` → `F_c,i (1, h_i, w_i)`(레벨별 축소) → spatial attention → `E_i (d, h_i, w_i)` → channel attention → `F_t,i (d, h_i, w_i)` (encoder feature와 동일 shape).

```python
# 논문 Eq.(1)-(4) 기반 의사코드
F_c_i = conv1x1_downsample(F_c, level=i)                          # (1,h1,w1) -> (1,hi,wi)
W_s_i = sigmoid(conv7x7(concat([avg_pool_c(conv1x1(F_c_i)),
                                  max_pool_c(conv1x1(F_c_i))])))   # Eq.1, spatial attention
E_i   = W_s_i * S_i                                                # Eq.2

W_c_i = sigmoid(MLP(avg_pool_s(E_i)) + MLP(max_pool_s(E_i)))       # Eq.3, channel attention
F_t_i = W_c_i * E_i                                                # Eq.4
```

(연한 노랑 하이라이트) <mark style="background: #FFF9D6A6;">Density map은 객체의 위치·밀도 정보를 담고 있으므로, 이를 attention map으로 변환해 encoder feature에 곱하면 전경 영역이 명시적으로 강조된다 — "정리" 표의 두 번째 문제(query 위치가 이미지 내용과 무관)를 해결하기 위한 전 단계로, 이렇게 강화된 feature가 이어지는 query 생성의 원재료가 된다. Ablation(Table 5)에서 CGFE 단독 기여(AP +3.2)가 DQS 단독 기여(+2.2)보다 큰 것도 이 근본적 역할을 뒷받침한다.</mark>

> [!info] 내 메모
> 

### ④ Dynamic Query Selection (DQS)
- **역할**: CGFE로 밀도·위치 정보가 이미 반영된 feature에서, ②가 정한 개수(`K`)만큼의 위치를 직접 선별해 decoder에 넣을 query의 content(내용)와 position(위치)을 동시에 만든다.
- **구현**: `F_t,i`를 모두 flatten+concat해 `F_flat`을 만들고, 분류 score를 산출하는 FFN에 통과시켜 top-K(CCM이 정한 `K`) 위치를 선별한다. 선별된 feature `F_select`의 선형 변환으로 query content `Q_content`를 생성하고, 별도 FFN으로 원본 anchor box에 대한 보정값(bias) `Δb=(Δb_x,Δb_y,Δb_w,Δb_h)`를 예측해 DAB-DETR 스타일 4D anchor box(x,y,w,h) 형태의 query position을 만든다.
- **입출력 shape**: `{F_t,i}` → flatten+concat `F_flat (d, ΣH_iW_i)` → top-K 선별 `F_select (K, d)` → `Q_content (K, d)` + `Q_position (K, 4)`.

```python
# 논문 Eq.(5)-(6) 기반 의사코드
F_flat = concat([flatten(F_t_i) for i in levels])   # (d, sum(hi*wi))
score = FFN(F_flat)                                  # (m, sum(hi*wi)), m=클래스 수
F_select = topK_by_score(F_flat, K=K, score=score)   # (K, d)

Q_content = linear(F_select)                          # (K, d)
delta_b = FFN(F_select)                               # (K, 4), anchor box에 더할 보정값
Q_position = original_anchor_box(F_select) + delta_b  # (K, 4), DAB-DETR 스타일 4D anchor
```

(연한 노랑 하이라이트) <mark style="background: #FFF9D6A6;">Query가 CGFE로 밀도·위치 정보가 이미 반영된 feature에서 직접 선별되므로, 최종 query의 content와 position 둘 다 "현재 이미지의 실제 객체 분포"를 반영하게 된다 — "정리" 표의 두 번째 문제(query 위치의 이미지 무관성)를 정면으로 해소하며, CCM의 개수 조정과 결합해 "몇 개를, 어디에" 둘 다를 동적으로 결정하는 완결된 해법이 된다.</mark>

> [!info] 내 메모
> 

### ⑤⑥ Deformable Decoder + Prediction Head
- **역할**: DQS가 만든 `K`개의 (content, position) query를 표준 Deformable DETR decoder에 넣어 encoder memory를 참조하며 정제한 뒤, 최종 (클래스, 박스)를 예측한다. 이 부분은 이 논문의 기여가 아니라 Deformable DETR의 표준 구조를 그대로 사용한다.
- **입출력 shape**: `Q_content (K,d)` + `Q_position (K,4)` + encoder memory → decoder 출력 `(K, d)` → 클래스 + 박스.

> [!info] 내 메모
> 

### ⑦ Loss
- **역할**: 예측과 정답을 [[Bipartite_Matching_Hungarian_Algorithm]]으로 매칭한 뒤, 매칭된 쌍에 대해 L1+GIoU(박스)와 focal loss(분류)를 결합한 Hungarian loss로 학습한다. 여기에 CCM의 분류를 감독하는 counting loss(cross-entropy)를 더하고, decoder 각 층 출력에도 동일한 Hungarian loss를 보조 손실로 건다.
- **구현**: `L_hungarian = λ1·L1 + λ2·L_GIoU + λ3·L_focal`(λ1=5, λ2=2, λ3=1, focal loss는 α=0.25, γ=2). `L_total = L_hungarian + L_aux + L_counting`. 학습은 2단계로 진행 — 먼저 CCM만 학습해 counting 결과를 안정화한 뒤, CGFE를 추가해 encoder feature 강화까지 함께 학습(오차 전파 최소화).
- **입출력**: (파라미터 갱신용 스칼라 loss이므로 별도 tensor shape 없음).

```python
# 논문 Eq.(7)-(8) 기반 의사코드
L_hungarian = 5 * L1 + 2 * L_GIoU + 1 * focal_loss(alpha=0.25, gamma=2)
L_total = L_hungarian + L_aux + cross_entropy(N_class_pred, N_class_gt)   # L_counting
```

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| ① Backbone+Encoder | 입력 이미지 | S_i (d,h_i,w_i), i=1..5 | 다중 스케일 feature 추출·문맥 반영 | ResNet-50 + 6층 deformable encoder |
| ② CCM | S_1 (d,h_1,w_1) | F_c (1,h_1,w_1) + K∈{300,500,900,1500} | 인스턴스 수 추정 → query 개수 결정 | Dilated conv([[Dilated_Convolution]]) + 4-class FFN |
| ③ CGFE | F_c + S_i | F_t,i (d,h_i,w_i) | 밀도 정보로 encoder feature 강화 | Spatial+channel attention(CBAM 방식) |
| ④ DQS | F_t (flatten) | Q_content(K,d) + Q_position(K,4) | Top-K 선별 → query content·position 생성 | FFN 분류 score + linear + anchor box bias FFN |
| ⑤⑥ Decoder+Head | Q_content, Q_position, memory | 클래스+박스(K개) | 최종 검출 | 표준 Deformable DETR decoder |
| ⑦ Loss | 예측 K개 + 정답 | 스칼라 loss | 학습 신호 결정 | Hungarian(L1+GIoU+Focal) + counting CE |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table 2 (AI-TOD-V2 test, ResNet50 backbone)
**표를 보는 법**: 각 행이 하나의 모델, `AP_vt/AP_t/AP_s/AP_m`이 객체 크기 구간별(very tiny/tiny/small/medium) 성능이다 — DINO-DETR(이전 SOTA)와 DQ-DETR을 비교하면 이 논문의 이득을 볼 수 있다.

| 벤치마크 | 지표 | Before(DINO-DETR*, 이전 SOTA) | After(DQ-DETR) |
|---|---|---|---|
| AI-TOD-V2 | AP / AP50 / AP75 | 25.9 / 61.3 / 17.5 | 30.2(+4.3%p) / 68.6 / 22.3 |
| AI-TOD-V2 | AP_vt / AP_t / AP_s / AP_m | 12.7 / 25.3 / 32.0 / 39.7 | 15.3(+20.5%) / 30.5(+20.6%) / 36.5(+14.1%) / 44.6(+12.3%) |

> [!note]- 세부 결과 및 Ablation
> #### Table 3 — VisDrone val
> **보는 법**: DQ-DETR AP 37.0으로 DINO-DETR*(35.8), DNTR(33.1), SDP(30.2) 등 상회하는지 확인.
> DQ-DETR이 baseline DINO-DETR 대비 AP/AP50/AP75 각각 +1.2/+2.6/+1.1로 일관 개선.
>
> #### Table 4 — COCO val (유일하게 열세)
> **보는 법**: Epochs 열까지 함께 봐야 공정 비교임을 알 수 있다 — DQ-DETR은 24 epoch만 학습.
> DINO-DETR(24 epoch) AP 51.3 > DQ-DETR(24 epoch) AP 50.2 — 저자는 이 열세를 (1) GPU 자원 제약으로 학습 최적화가 제한됨, (2) COCO는 이미지당 인스턴스 수가 비교적 균등해 이 논문의 핵심 설계(imbalance 대응)가 강점을 발휘할 여지가 적다는 두 가지로 설명.
>
> #### Table 5 — 메인 ablation (AI-TOD-V2 test)
> **보는 법**: CC(counting)/DQS(query selection)/FE(feature enhancement) 체크 조합별 AP 비교.
>
> | CC | DQS | FE | AP | AP_vt | AP_t | AP_s | AP_m |
> |---|---|---|---|---|---|---|---|
> | | | | 25.9 | 12.7 | 25.3 | 32.0 | 39.7 |
> | ✓ | ✓ | | 28.1 | 12.3 | 27.8 | 34.6 | 44.1 |
> | ✓ | | ✓ | 29.1 | 14.4 | 29.3 | 35.2 | 44.1 |
> | ✓ | ✓ | ✓ | **30.2** | **15.3** | **30.5** | **36.5** | **44.6** |
>
> Feature Enhancement(CGFE)의 기여(+3.2 AP)가 Dynamic Query Selection(+2.2 AP)보다 큼 — density map을 encoder feature 보강에 쓰는 것 자체가 query 선별보다 더 직접적인 이득을 줌.
>
> #### Table 6 — 밀도 구간별 성능 비교 (DINO-DETR vs DQ-DETR)
> **보는 법**: `#Query` 열에서 DINO-DETR은 항상 900, DQ-DETR은 구간마다 다른 값을 쓰는지 확인 — "Dynamic"이 실제로 구간별 최적 K를 쓴다는 근거.
>
> | 인스턴스 수 구간 | #Query(DINO-DETR) | DINO-DETR AP | #Query(DQ-DETR) | DQ-DETR AP |
> |---|---|---|---|---|
> | N≤10 | 900 | 22.5 | 300 | 26.1(+16.0%) |
> | 10<N≤100 | 900 | 24.4 | 500 | 28.4(+16.4%) |
> | 100<N≤500 | 900 | 31.6 | 900 | 33.7(+6.6%) |
> | 500<N | 900 | 13.5 | 1500 | 14.7(+8.9%) |
>
> N>500 구간에서 DINO-DETR은 query 900개로 900개 넘는 인스턴스를 감당 못해 AP가 전체 구간 중 최저(13.5) — DQ-DETR은 query를 1500으로 늘려 AP_vt를 42.1% 개선(Table 6 본문 서술).
>
> #### Table 7 — CCM 분류 정확도별 DQ-DETR 성능
> **보는 법**: Accuracy(%) 열이 CCM이 그 구간을 맞게 분류한 비율 — 정확도가 낮은 구간일수록 DQ-DETR의 개선폭도 작은지 대조.
>
> | #Objects | Accuracy(%) | AP(DQ-DETR) | AP(Baseline) | #Sample |
> |---|---|---|---|---|
> | N≤10 | 97.7 | 26.1(+3.6) | 22.5 | 8674 |
> | 10<N≤100 | 90.5 | 28.4(+4.0) | 24.4 | 4393 |
> | 100<N≤500 | 86.5 | 33.7(+2.1) | 31.6 | 905 |
> | 500<N | 56.5 | 14.7(+1.2) | 13.5 | 46 |
> | Total | 94.6 | 30.2 | 25.9 | 14018 |
>
> N>500 구간은 학습 샘플이 46장뿐이라 분류 정확도가 56.6%로 급락, 이 때문에 개선폭도 다른 구간(+2.1~4.0)보다 작은 +1.2에 그침.
>
> #### Table 8 — Counting 방식: 분류 vs 회귀
> **보는 법**: 같은 모델에서 counting head만 분류→회귀로 바꿨을 때 성능이 어떻게 붕괴하는지 확인.
>
> | Method | AP | AP_vt | AP_t | AP_s | AP_m |
> |---|---|---|---|---|---|
> | Baseline | 25.9 | 12.7 | 25.3 | 32.0 | 39.7 |
> | Regression | 14.9 | 5.2 | 16.3 | 19.9 | 14.3 |
> | Classification(채택) | **30.2** | **15.3** | **30.5** | **36.5** | **44.6** |
>
> 회귀는 N이 1~2667까지 편차가 너무 커 정확한 값 예측이 어렵고, 그 오차가 곧바로 부적절한 query 수로 이어져 baseline보다도 크게 하락 — 분류가 필수적인 설계 선택임을 뒷받침.

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- CCM의 분류 오류가 이후 파이프라인 전체(query 개수 결정)에 연쇄적으로 영향 → <mark style="background: #FF5582A6;">Table 7에서 N>500 구간의 분류 정확도가 56.6%에 불과하며, 저자도 이 구간의 성능 개선폭이 작은 이유를 직접 CCM의 낮은 정확도로 귀속시킨다 — 오차 전파를 완전히 막지 못했음을 스스로 인정.</mark>
- Query 수가 최대 1500까지 늘어나면 decoder self-attention의 이차 복잡도 문제가 재발할 위험 → <mark style="background: #FF5582A6;">논문은 연산량(FLOPs)이나 실제 추론 속도(FPS) 수치를 전혀 보고하지 않는다 — "동적으로 줄인다"는 주장이 밀집 이미지에서는 오히려 기존보다 더 많은 query(1500 vs 900)를 쓰게 되는데도 이 트레이드오프를 정량화하지 않았다.</mark>

### 한계
- <mark style="background: #FF5582A6;">COCO에서 DINO-DETR보다 낮은 성능(50.2 vs 51.3) — tiny object·imbalance 특화 설계가 범용 벤치마크에서는 오히려 이점이 되지 못함을 저자가 인정.</mark>
- <mark style="background: #FF5582A6;">CCM의 구간 경계(10/100/500)가 AI-TOD-V2 통계에 맞춰 수동 설정되어 있고, "다른 데이터셋에는 재조정이 필요하다"고 명시 — 완전 자동화된 것이 아니라 데이터셋별 하이퍼파라미터 튜닝이 필요.</mark>
- Batch size가 메모리 제약으로 1에 고정 — 대규모 배치 학습 시의 안정성·성능은 검증되지 않음.
- 회귀 대신 분류를 택하며 발생하는 이산화 손실(500<N≤2667 구간을 세분화하지 못함)은 근본적으로 해결되지 않은 trade-off로 남음.

### 생각할 점
- <mark style="background: #A6E3A1A6;">Ablation(Table 5)에서 CGFE(feature enhancement)의 단독 기여(+3.2 AP)가 DQS(query selection)의 단독 기여(+2.2 AP)보다 크다는 점은, "query를 몇 개 쓸지 정교하게 정하는 것"보다 "feature 자체를 밀도 정보로 보강하는 것"이 더 근본적인 개선 지점일 수 있음을 시사한다 — 이는 이 위키의 [[ORFENet]], [[Deformable-DETR]] 등에서 반복 관찰된 "다중 소스 활용 자체의 기여가 정교화보다 크다"는 패턴과 다시 일치한다.</mark>
- <mark style="background: #A6E3A1A6;">이 논문은 뒤이어 비교할 dynamic query DETR 계열 중 "density map(회귀가 아닌 분류)으로 query 개수를 정하고, 같은 density map으로 feature까지 보강한다"는 방식을 취한다 — 이후 처리할 IG-DETR(instance-guided)이나 PaQ-DETR(pattern/quality-aware)이 query를 어떻게 다르게 생성하는지와 정확히 대조할 지점.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">CCM의 밀도 분류가 4단계 이산 값이라는 점은, 이 위키에서 다루는 Gaussian 기반 label assignment 계열([[Unc-SOD]], [[CDATOD-Diff]])의 "연속값 uncertainty/의미 정보" 접근과 대조된다 — query 개수 결정에도 이산 분류 대신 연속적인 신뢰도 기반 조정을 적용하면 더 세밀한 대응이 가능할지 검토할 가치가 있다.</mark>
- <mark style="background: #A6E3A1A6;">CGFE의 spatial+channel attention 구조는 [[FFCA-YOLO]]의 SCAM, [[RS-TOD]]의 attention 모듈과 유사한 "density/context 정보를 attention map으로 변환해 feature를 보강한다"는 공통 패턴을 공유한다 — DETR 계열과 YOLO/CNN 계열 모두에서 이 패턴이 독립적으로 재발견되고 있다는 점이 흥미롭다.</mark>

> [!info] 내 메모
> 

# 관련 개념
- [[Density_Guided_Dynamic_Query]] — 이 논문의 핵심 기여. Density map으로 query 개수(counting)와 feature 강화를 동시에 수행하는 메커니즘.
- [[Dilated_Convolution]] — CCM의 density extractor에서 넓은 수용영역으로 tiny object의 장거리 의존성을 포착하는 데 사용.
- [[1x1_Convolution]] — CGFE에서 density map을 각 인코더 레벨 채널·해상도로 다운샘플링하는 데 사용.
- [[Multi_Head_Self_Attention]] — Deformable encoder/decoder의 attention 기반 개념.
- [[Bipartite_Matching_Hungarian_Algorithm]] — 예측-정답 매칭에 사용되는 Hungarian loss의 기반.

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]] — dynamic query DETR 계열(DQ-DETR 외 여러 편)의 첫 사례로 새 소그룹에 편입 예정.

# 읽어볼 만한 논문
- 참고문헌 기반: X. Zhu, W. Su, L. Lu, B. Li, X. Wang, J. Dai, "Deformable DETR: Deformable transformers for end-to-end object detection" (ICLR 2021) [31] — 이미 위키에 있음: [[Deformable-DETR]]. 이 논문의 직접 baseline이자 5-scale deformable attention 구조 전체를 그대로 계승.
- 참고문헌 기반: H. Zhang, F. Li, S. Liu, L. Zhang, H. Su, J. Zhu, L. M. Ni, H.-Y. Shum, "DINO: DETR with improved denoising anchors for end-to-end object detection" (ICLR 2023) [28] — 이 논문이 실험 전반에서 가장 강력한 DETR 계열 baseline으로 비교하는 대상. Denoising anchor 설계를 이해하면 DQ-DETR과의 차이가 명확해짐.
- 참고문헌 기반: S. Liu, F. Li, H. Zhang, X. Yang, X. Qi, H. Su, J. Zhu, L. Zhang, "DAB-DETR: Dynamic anchor boxes are better queries for DETR" (ICLR 2022) [11] — DQ-DETR의 query formulation(4D anchor box, content+position 분리)이 그대로 채택한 원조 설계.
- 자유 추천(검증 필요): Crowd counting 분야에서 density map 기반 개수 추정과 downstream detection/localization을 결합한 연구 — 검색 키워드: `density map crowd counting guided object detection query`. CCM의 density map 활용이 crowd counting 분야의 어떤 기법 계보에서 왔는지 배경 이해에 도움될 것으로 예상.
