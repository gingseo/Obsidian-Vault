---
pm-task: true
projectId: "paperwiki-small-object-detection"
parentId:
id: "t-dqp-detr-bdupxqtsg1"
title: "DQP-DETR: Object-Density-Guided Query Prioritization for Small Object Detection in UAV Imagery"
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
  "1frf59rymtcjvske": "SSRN"
subtaskIds: []
dependencies: []
year: 2026
venue: "SSRN (preprint, submitted to Elsevier, not peer-reviewed)"
jcr_quartile: arXiv
task: [small-object-detection]
direction: [improvement]
paper_tags: [paper, small-object-detection, tiny-object-detection, detr, dynamic-query, density-map, uav, query-ranking]
source: "/Users/GyeongSeo/Workspace/논문_pdf/Small_Object_Detection/2026_SSRN_DQP-DETR.pdf"
createdAt: "2026-08-24T03:19:00.000Z"
updatedAt: "2026-08-28T18:40:00.000Z"
---

Project: [[논문_Small_Object_Detection|Small Object Detection]]
#paper #small-object-detection #tiny-object-detection #detr #dynamic-query #density-map #uav #query-ranking

> [!quote] 원제
> **DQP-DETR: Object-Density-Guided Query Prioritization for Small Object Detection in UAV Imagery**
> Ningsheng Liao, Hao Sun, Yunhao Gong, Mi Zhu, Bo Peng — Chongqing University of Technology, SSRN preprint 2026 (미출판, Elsevier 제출)

# 한 줄 요약
<mark style="background: #FFF3A3A6;">얕은 feature에서 예측한 density map을 단순 auxiliary branch로 두지 않고, encoder memory에 양방향 cross-modulation으로 직접 주입하고(BCME) 토큰 공간으로 투영해 classification score와 곱해 query 우선순위를 만든 뒤(RCS), GT 밀도 기반 참조 우선순위로 이 순위 자체를 학습시켜 D-Fine-S baseline 대비 VisDrone AP50 +4.0%p·AI-TOD +3.1%p를 얻은 DQP-DETR — 밀도 정보를 "예측 대상"에서 "query 배정을 직접 구동하는 신호"로 격상시킨 UAV 소형 객체 탐지 논문(SSRN 프리프린트, 미출판).</mark>

> [!info] 내 메모
> 

# 정리

| | 문제 ① — 밀도 정보가 query 배정에 간접적으로만 관여 | 문제 ② — Pixel-level density와 token-level 선택 간 단절 |
|---|---|---|
| **문제 정의** | Density map은 crowd counting·dense scene 분석에 널리 쓰여왔지만, 기존 detection 방법들은 이를 보조 예측 브랜치나 feature 강화 신호 정도로만 사용한다 — density map과 실제 decoder query 배정(개수·순위) 사이의 연결이 간접적이다. | Density map을 pixel-space에서 정확히 예측하는 것과, 그 정보로 decoder의 top-k query 선택을 실제로 잘하는 것은 서로 다른 목표다. Pixel-level regression loss만으로 학습하면, 시각적으로 정확한 density map이라도 token-level 순위 매김에는 도움이 안 될 수 있다. |
| **풀고자 하는 문제** | Density map을 encoder feature 강화와 query 개수·순위 결정에 직접 참여하는 일급 신호로 격상시키는 것 | GT 밀도로부터 만든 token-level 참조 우선순위로, 예측된 query 우선순위의 상대적 순서 자체를 직접 감독하는 것 |
| **선행 연구 접근** | - [[DQ-DETR]], IG-DETR, DQA-DETR 등: density/counting 정보를 query 개수 조정에만 사용하거나 auxiliary supervision으로만 사용<br>- 원조 density map 연구(CSRNet, Bayesian loss 등 crowd counting 계열): pixel-level 회귀 정확도에만 집중, detection query 배정과 연결하지 않음 | - 기존 density 기반 detection: pixel-level regression loss로만 density map을 학습<br>- **갭**: density map이 실제로 top-k 선택에 효과적인지(ranking 능력)를 직접 감독하는 방법은 없었음 |
| **해결 방법** | ADPG(얕은 feature에서 density map 생성) + BCME(density feature를 encoder memory에 양방향으로 주입해 표현 자체를 강화) | RCS(Ranking Consistency Supervision) — GT 박스로 만든 density map을 token 공간에 투영해 참조 우선순위를 만들고, margin ranking loss로 고우선순위·저우선순위 토큰 간 상대적 순서를 직접 감독 |
| **예상되는 문제점** | BCME가 encoder의 모든 스케일 feature에 추가 modulation 연산을 가하므로, density map 예측이 부정확하면 이 오류가 encoder 표현 전체로 퍼질 위험 | RCS는 GT 밀도 기반 참조 우선순위에 의존하므로, 밀도 계산 자체가 모호한 극단적으로 조밀하거나 객체 경계가 겹치는 장면에서는 참조값 자체의 신뢰도가 흔들릴 수 있음 |

**갭 종합**: <mark style="background: #FFF3A3A6;">기존 density 기반 detection 방법들은 density map을 "보조 예측 결과물"로 다뤄, 이 예측이 실제로 decoder의 query 배정에 얼마나 유용한지는 간접적으로만 검증됐다. 이 논문은 density map을 (1) encoder 표현 강화, (2) query 개수 추정, (3) 개별 토큰 순위 결정 세 지점 모두에 직접 참여시키고, token-level ranking 능력 자체를 별도로 감독(RCS)함으로써 이 간극을 메운다는 것이 통찰이다.</mark>

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">얕은 feature로부터 <span style="color:#c0392b; font-weight:bold;">Adaptive Density Prior Generation(ADPG)</span>이 density map을 예측하고, <span style="color:#c0392b; font-weight:bold;">Bidirectional Cross-Modulation Enhancement(BCME)</span>가 이 density feature를 encoder memory에 양방향으로 주입해 표현 자체를 강화한다. 이 density map을 token 공간으로 투영해 classification score와 결합한 query 우선순위를 만들고, decoder query 선택에 직접 사용하되, <span style="color:#c0392b; font-weight:bold;">Ranking Consistency Supervision(RCS)</span>으로 이 우선순위의 상대적 순서 자체를 GT 밀도 기반 참조와 정렬시킨다.</mark>

## 전체 파이프라인 (Fig. 1 기준)

```
입력 이미지
       │
       ▼
① Backbone                              → shallow feature F_s + multi-scale P3,P4,P5
       │
       ▼
② Adaptive Density Prior Generation (ADPG)
   F_s → multi-branch conv(k=3,5,7,9,11) → 가중합 G → density map D + density-enhanced feature F_d
       │                                          → D (1, h_s, w_s), F_d (C, h_s, w_s)
       ▼
③ Encoder (P3,P4,P5)                     → memory M_l (C, h_l, w_l), l=3,4,5
       │
       ▼
④ Bidirectional Cross-Modulation Enhancement (BCME)
   F_d를 각 스케일로 다운샘플 → spatial modulation → channel modulation → semantic feedback → cross-scale gating
       │                                          → enhanced memory M'_l (C, h_l, w_l)
       ▼
⑤ Density-guided Query Prioritization
   D를 token 공간에 투영(multi-scale pooling+flatten) → 분류 score와 결합 → query priority q
   전역 density 합으로 동적 query 개수 N_q 추정 → top-N_q 선택
       │                                          → decoder query 초기화 (N_q, C)
       ▼
⑥ Decoder & Head                         → 클래스 + 박스 (N_q개)
       │
       ▼ (학습 시에만)
⑦ Ranking Consistency Supervision (RCS)
   GT 박스 → GT density map → 참조 우선순위 → margin ranking loss L_rank
```

> [!info] 내 메모
> 

### ① Backbone
- **역할**: 입력 이미지에서 얕은 feature(공간 디테일이 살아있는 고해상도 feature, density map 생성에 사용)와 다중 스케일 feature(encoder 입력용)를 함께 추출한다.
- **구현**: 표준 CNN backbone(D-Fine-S baseline 그대로 사용).
- **입출력 shape**: 입력 이미지 → shallow feature `F_s (C, h_s, w_s)` + multi-scale `P_3, P_4, P_5`.

> [!info] 내 메모
> 

### ② Adaptive Density Prior Generation (ADPG)
- **역할**: Small object는 feature abstraction 과정에서 쉽게 약해지므로, 공간 디테일이 가장 잘 보존된 얕은 feature로부터 "객체가 밀집한 정도"를 나타내는 density map을 만든다. 동시에 이 density 정보를 다시 feature에 residual로 결합해, encoder에 넣기 전부터 이미 밀도 인식이 반영된 feature를 준비한다.
- **구현**: 얕은 feature `F_s`를 서로 다른 수용영역을 가진 R개 convolution branch(kernel size 3/5/7/9/11)에 통과시켜 각 branch 응답 `G_r`을 얻는다([[Dilated_Convolution]]처럼 여러 수용영역을 병렬로 두는 설계와 유사한 목적, 다만 여기서는 dilation이 아니라 커널 크기 자체를 다양화). Global average pooling+경량 매핑으로 만든 적응형 가중치 `α_r`로 branch 출력을 가중합해 `G`를 얻고, density prediction head(`f_den`, sigmoid)로 초기 density map `D_0`를 만든 뒤 refinement branch(`f_ref`)로 보정해 최종 `D = D_0 + f_ref(D_0)`를 얻는다. 이 `D`(정확히는 `F_s` 크기로 보간한 `D̃`)를 다시 `F_s`에 곱하고 더해 density-enhanced feature `F_d = G + G⊙D̃ + F_s`를 만든다.
- **입출력 shape**: `F_s (C, h_s, w_s)` → 여러 branch → 가중합 `G (C, h_s, w_s)` → `D (1, h_s, w_s)` + `F_d (C, h_s, w_s)`.

```python
# 논문 Eq.(1)-(4) 기반 의사코드
G_r = [conv_branch_k(F_s) for k in [3,5,7,9,11]]         # Eq.1, R개 branch
alpha = softmax(lightweight_map(global_avg_pool(F_s)))     # 적응형 branch 가중치
G = sum(alpha[r] * G_r[r] for r in range(R))               # Eq.2

D0 = sigmoid(f_den(G))                                      # 초기 density map
D  = D0 + f_ref(D0)                                          # Eq.3, refinement 보정

D_tilde = interpolate(D, size=F_s.shape[-2:])                # F_s 크기로 보간
F_d = G + G * D_tilde + F_s                                  # Eq.4, density-enhanced feature
```

(연한 노랑 하이라이트) <mark style="background: #FFF9D6A6;">여러 수용영역의 conv branch를 적응적으로 가중합함으로써, 장면마다 다른 객체 군집 패턴(조밀한 군집 vs 넓게 퍼진 분포)에 유연하게 대응한다 — "정리" 표의 첫 번째 문제(밀도 정보가 간접적으로만 쓰임)를 해결하는 전 단계로, 이 density map과 density-enhanced feature가 이어지는 BCME·query prioritization 두 곳 모두의 원재료가 된다.</mark>

> [!info] 내 메모
> 

### ③④ Encoder + Bidirectional Cross-Modulation Enhancement (BCME)
- **역할**: 표준 encoder가 다중 스케일 feature를 문맥화한 memory를 만들면, BCME가 ②에서 만든 density-enhanced feature를 이 memory에 주입해 "밀도가 높은 영역에 더 집중하고, 배경 간섭은 억제하는" encoder 표현으로 재구성한다. "양방향(bidirectional)"인 이유는 density feature가 memory를 변조할 뿐 아니라, memory의 semantic 정보도 다시 density feature 쪽으로 피드백되어 density feature 자체가 더 detection에 유용한 semantic을 흡수하기 때문이다.
- **구현**: `F_d`를 각 encoder 스케일에 맞춰 다운샘플링한 density feature `F_d^l`을 만든다. (1) **Spatial modulation**: `F_d^l`로부터 spatial weight `A_l`을 생성해 memory `M_l`에 곱해 밀도 높은 영역을 강조(`M̃_l = M_l ⊙ A_l`). (2) **Channel modulation**: `F_d^l`로부터 channel weight `C_l`을 생성해 다시 곱해 채널별 응답을 조정(`M'_l = M̃_l ⊙ C_l`). (3) **Semantic feedback**: memory로부터 만든 feedback gate `B_l`을 density feature에 곱해 더함으로써(`F_d^l = F_d^l + F_d^l ⊙ B_l`), density feature가 단순 저수준 공간 응답에 머물지 않고 detection semantic을 점차 흡수하게 함. (4) **Cross-scale gating**: 이전(고해상도) 스케일의 density feature를 보간해 현재 스케일과 결합(`F_d^l = F_d^l + Gate(Interp(F_d^{l-1}))`)해, 세밀한 디테일이 저해상도 스케일까지 전파되도록 함.
- **입출력 shape**: `F_d (C, h_s, w_s)` + `M_l (C, h_l, w_l)` → 스케일별 다운샘플 `F_d^l (C, h_l, w_l)` → spatial+channel modulation → `M'_l (C, h_l, w_l)` (memory와 동일 shape).

```python
# 논문 Eq.(5)-(8) 기반 의사코드
F_d_l = downsample(F_d, level=l)                          # 스케일별 density feature

A_l = spatial_weight_from(F_d_l)
M_tilde_l = M_l * A_l                                       # Eq.5, spatial modulation

C_l = channel_weight_from(F_d_l)
M_prime_l = M_tilde_l * C_l                                  # Eq.6, channel modulation

B_l = feedback_gate_from(M_l)
F_d_l = F_d_l + F_d_l * B_l                                  # Eq.7, semantic feedback

F_d_l = F_d_l + gate(interpolate(F_d_prev_level))            # Eq.8, cross-scale gating
```

(연한 노랑 하이라이트) <mark style="background: #FFF9D6A6;">기존 방법들은 density map을 auxiliary branch로만 다뤄 encoder 표현 자체에는 관여하지 못했는데, BCME는 이를 memory에 직접 주입해 encoder 표현 단계에서부터 밀도 인식을 반영한다 — "정리" 표의 첫 번째 문제를 정면으로 겨냥한다. Table 3에서 ADPG+BCME만 추가해도 AP가 25.5→26.1로 개선되는 것이 이 근거다.</mark>

> [!info] 내 메모
> 

### ⑤ Density-guided Query Prioritization
- **역할**: BCME로 강화된 memory와 density map을 이용해, decoder에 넣을 query의 "개수"와 "어떤 토큰을 우선할지"를 결정한다. 단순히 classification confidence만 보면 배경 토큰이 불안정하게 높은 응답을 내는 경우와, 진짜 소형 객체인데 외형이 약해 confidence가 낮은 경우를 구별하지 못하는데, density 정보를 곱하면 이 둘을 더 잘 구별할 수 있다.
- **구현**: density map `D`를 encoder의 각 스케일에 맞춰 pooling한 뒤 flatten+concat해 encoder token과 정렬된 밀도 가중치 시퀀스 `d`를 만들고, 극단값의 영향을 줄이기 위해 온도 스무딩·정규화(`d̂_i = d_i^(1/τ) / max_j(d_j^(1/τ))`)를 적용한다. Encoder classification branch의 최대 클래스 응답을 confidence `s_i`로 삼아, 토큰별 query priority `q_i = s_i · d̂_i`를 만든다(confidence와 density 둘 다 높아야 priority가 높음). 전역 density 합을 스케일링 계수 `λ`로 조정하고 최소/최대 범위로 clip해 이번 이미지에 쓸 동적 query 개수 `N_q`를 추정한 뒤, `q_i` 기준 top-`N_q` 토큰을 decoder query로 선택한다.
- **입출력 shape**: `D (1, h_s, w_s)` + 각 스케일 encoder token → `d, d̂ (ΣH_lW_l,)` + `s (ΣH_lW_l,)` → `q (ΣH_lW_l,)` → top-`N_q` 선택 → decoder query `(N_q, C)`.

```python
# 논문 Eq.(9)-(14) 기반 의사코드
d = concat([flatten(pool(D, level=l)) for l in scales])       # Eq.9, 토큰 정렬된 density weight
d_hat = d ** (1/tau) / (max(d ** (1/tau)) + eps)                # Eq.10, 온도 스무딩+정규화

s = max_k(S_cls)                                                # Eq.11, encoder classification 최대 응답
q = s * d_hat                                                    # Eq.12, token-level query priority

N_q = clip(lam * sum(D), q_min, q_max)                          # Eq.13, 동적 query 개수 추정
Q = max(N_q for image in batch)                                  # Eq.14, batch 내 top-k 크기 통일

top_k_idx = select_top_k(q, k=Q)
decoder_query_init = gather(memory, top_k_idx)                   # (N_q, C)
```

(연한 노랑 하이라이트) <mark style="background: #FFF9D6A6;">Classification 응답만으로 top-k를 뽑는 기존 DETR 방식은 "정리" 표의 두 번째 문제(배경의 불안정한 고응답, 소형 객체의 약한 응답)에 취약한데, 밀도 가중치를 곱해 두 신호를 결합함으로써 이를 완화한다 — Table 4에서 Dynamic Query(density 기반 개수 추정)만 추가해도 AP가 25.5→26.1로 개선되고, RCS까지 추가하면 28.2까지 오른다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> `q_i = s_i · d̂_i`는 두 신호의 단순 곱이라, 둘 중 하나가 극단적으로 낮으면(예: density가 거의 0인 위치) confidence가 아무리 높아도 priority가 낮게 억제된다 — 실제로는 객체이지만 주변에 다른 객체가 없어 density 응답이 약한 고립된 소형 객체가 저평가될 위험이 있다.

> [!info] 내 메모
> 

### ⑥⑦ Decoder & Head + Ranking Consistency Supervision (RCS, 학습 시에만)
- **역할**: 선택된 query로 표준 decoder가 최종 (클래스, 박스)를 예측한다. 여기에 더해, RCS는 "pixel-level에서 정확한 density map"과 "token-level에서 실제로 유용한 순위"가 다를 수 있다는 간극을 메우기 위해, GT 박스로 만든 density map을 똑같이 token 공간으로 투영해 참조 우선순위를 만들고, 예측된 priority가 이 참조와 같은 상대적 순서를 갖도록 margin ranking loss로 직접 감독한다.
- **구현**: GT 박스로부터 GT density map `D_gt`를 만들고(가우시안 커널 등으로 각 객체 중심에 밀도를 배치하는 방식으로 추정), ⑤와 동일한 방식(pooling+flatten+정규화)으로 GT 밀도 가중치 `d̂_gt`를 만든다. 이를 encoder classification 응답과 결합한 GT 참조 우선순위 `q_gt_i = s_i · d̂_gt_i`를 만들고, `q_gt`가 높은 토큰(고우선순위, 대체로 객체 군집·객체 근방)과 낮은 토큰(저우선순위, 대체로 배경)을 각각 뽑는다. 고우선순위 토큰 `p`와 저우선순위 토큰 `n`에 대해, 예측 priority가 `q_p ≥ q_n + m`(margin `m`)을 만족하도록 hinge 형태의 margin ranking loss `L_rank`를 부과한다.
- **입출력 shape**: `D_gt (1, h_s, w_s)` → `q_gt (ΣH_lW_l,)` → 고/저 우선순위 토큰 인덱스 선택 → `L_rank` (스칼라 loss, 추론 시에는 사용하지 않음).

```python
# 논문 Eq.(15)-(19), Algorithm 1 기반 의사코드 (학습 시에만 실행)
D_gt = generate_density_map(gt_boxes)                            # GT 박스 -> GT density map
d_gt_hat = normalize(pool_flatten(D_gt) ** (1/tau))               # Eq.16, 동일 투영 절차
q_gt = s * d_gt_hat                                                # Eq.15, GT 참조 우선순위

I_pos, I_neg = select_high_low(q_gt)                               # 고/저 우선순위 토큰 인덱스
L_rank = mean(max(0, margin - q[I_pos] + q[I_neg]))                # Eq.17, margin ranking loss

L_density = beta1 * L_den + beta2 * L_cnt + beta3 * L_rank          # Eq.18
L_total = L_det + L_density                                         # Eq.19
```

(연한 노랑 하이라이트) <mark style="background: #FFF9D6A6;">Pixel-level density regression loss만으로는 "이 density map이 실제로 좋은 query 순위를 만드는가"를 직접 검증하지 못한다는 "정리" 표의 두 번째 문제를, GT 밀도를 token-level 참조 우선순위로 변환해 순서 자체를 직접 감독함으로써 해결한다 — Table 3에서 RCS 추가 시 AP가 26.1→27.6/28.2로 개선되고, Fig. 3의 정성적 비교에서도 RCS 적용 후 density 응답이 실제 객체 군집에 더 집중되고 배경의 산발적 응답이 억제됨을 확인했다.</mark>

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| ① Backbone | 입력 이미지 | F_s(C,h_s,w_s) + P3~P5 | 얕은/다중스케일 feature 추출 | 표준 CNN backbone(D-Fine-S) |
| ② ADPG | F_s | D(1,h_s,w_s) + F_d(C,h_s,w_s) | 밀도 예측 + feature 강화 | Multi-branch conv(k=3~11) 가중합 + density head + refinement |
| ③④ Encoder+BCME | P3~P5, F_d | M'_l(C,h_l,w_l) | Density feature를 memory에 양방향 주입 | Spatial+channel modulation, semantic feedback, cross-scale gating |
| ⑤ Query Prioritization | D, memory, classification score | decoder query init(N_q,C) | Query 개수·순위를 밀도로 결정 | Multi-scale pooling+투영, s·d̂ 결합, 동적 top-N_q |
| ⑥⑦ Decoder+Head+RCS | query init(N_q,C) | 클래스+박스(N_q개) + L_rank | 최종 검출 + 순위 학습 감독 | 표준 decoder + margin ranking loss(GT 참조 우선순위) |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table 1 (VisDrone val, D-Fine-S baseline)
**표를 보는 법**: DETR 계열 detector들과 나란히 비교한 표에서, baseline인 D-Fine-S와 DQP-DETR의 차이가 이 논문의 순수 기여다.

| 벤치마크 | 지표 | Before(D-Fine-S baseline) | After(DQP-DETR) |
|---|---|---|---|
| VisDrone val | AP / AP50 / AP75 | 26.2 / 42.4 / 26.5 | 28.2(+2.0) / 46.4(+4.0) / 28.7(+2.2) |
| VisDrone val | AP_S / AP_M / AP_L | 18.2 / 35.9 / 45.3 | 20.8(+2.6) / 37.5 / 45.9 |

> [!note]- 세부 결과 및 Ablation
> #### Table 1 — VisDrone 전체 비교 (YOLO 계열 vs DETR 계열)
> **보는 법**: YOLO 계열(Yolov8m·YOLOv12m·FBRT-YOLO)과 DETR 계열(D-Fine-S·DEIM-S·RT-DETR-r18·Sparse-DETR·Deformable DETR·DQ-DETR)을 함께 나열, DQP-DETR이 DETR 계열 안에서 최고인지 확인.
> DQP-DETR(AP 28.2, AP50 46.4, AP_S 20.8)이 표의 모든 DETR/YOLO 계열 방법 중 AP50·AP_S 최고. 파라미터 수는 11.90M으로 YOLO 계열(FBRT-YOLO 14.89M)보다도 적고 DETR 계열 중에서도 가장 가벼운 축(Deformable DETR 40.56M, DQ-DETR 43.00M 대비 크게 작음). DQ-DETR과 비교하면 AP·AP50·AP_S 모두 더 높으면서 파라미터는 훨씬 적어, "파라미터를 늘려서가 아니라 더 효과적인 query 배정으로" 이득을 얻었음을 저자가 강조.
>
> #### Table 2 — AI-TOD val (tiny object 일반화 검증)
> **보는 법**: D-Fine-S 대비 DQP-DETR의 개선폭이 AI-TOD처럼 더 작고 약한 객체에서도 유지되는지 확인.
> AP 16.5→17.9(+1.4), AP50 35.6→38.7(+3.1), AP75 12.9→14.2(+1.3), AP_S 22.1→23.3(+1.2) — VisDrone보다 절대 성능은 낮지만 개선 경향은 일관되게 유지, tiny object 일반화 능력을 뒷받침.
>
> #### Table 3 — 메인 ablation (ADPG/BCME/RCS, VisDrone)
> **보는 법**: 모듈을 하나씩 추가하며 AP/AP50/AP_S가 어떻게 바뀌는지 확인.
>
> | ADPG | BCME | RCS | AP | AP50 | AP_S |
> |---|---|---|---|---|---|
> | - | - | - | 25.5 | 42.1 | 18.2 |
> | ✓ | ✓ | - | 26.1 | 43.2 | 18.7 |
> | ✓ | ✓ | (Dynamic Query만) | 27.6 | 45.3 | 20.0 |
> | ✓ | ✓ | ✓ | **28.2** | **46.4** | **20.8** |
>
> RCS가 ADPG+BCME(26.1) 대비 가장 큰 단독 증분(27.6→28.2)을 만들어, "density prior를 pixel-level에서 잘 예측하는 것"보다 "token-level 순위를 직접 감독하는 것"이 더 결정적임을 시사.
>
> #### Table 4 — Query 배정 전략 자체의 ablation (Fixed vs Dynamic vs RCS)
> **보는 법**: Query 개수를 고정(300)했을 때와, 동적으로 추정했을 때, RCS까지 더했을 때를 비교.
>
> | Method | Dynamic Query | RCS | AP | AP50 | AP_S |
> |---|---|---|---|---|---|
> | Fixed Query 300 | - | - | 25.5 | 42.1 | 18.2 |
> | Dynamic Query | ✓ | - | 26.1 | 43.2 | 18.7 |
> | Full Model | ✓ | ✓ | **28.2** | **46.4** | **20.8** |
>
> Dynamic Query 단독으로는 AP 개선이 완만(+0.6)한데, RCS를 더하면 도약(+2.1)한다 — "Dynamic Query가 개수를 정하고, RCS가 후보 순위 선택을 정교화한다"는 역할 분담을 보여줌.
>
> #### Fig. 2 — 동적 query 배정의 실제 상관관계 분석
> **보는 법**: 3개 산점도(GT 객체 수 vs 할당 query 수, query 수 분포, 예측 density 합 vs GT 객체 수)를 통해 "동적 배정이 실제로 이미지 내용에 반응하는지" 확인.
> 왼쪽 패널에서 GT 객체 수가 늘수록 할당 query 수도 대체로 증가하는 양의 상관관계 확인 — 고정 개수가 아니라 실제로 이미지별 밀도에 반응해 개수가 달라짐을 실증. 오른쪽 패널의 예측 density 합과 GT 객체 수 간 양의 상관관계는, density map의 전역 응답이 실제 객체 수 변화를 어느 정도 반영한다는 근거.
>
> #### Fig. 3 — RCS 적용 전/후 density 응답 시각화 (밀집 장면 vs 희소 장면)
> **보는 법**: 원본 이미지, GT density prior, RCS 없는 예측 응답+query 분포, RCS 있는 예측 응답+query 분포를 나란히 비교.
> RCS 없이는 query 일부가 배경이나 희소 객체 영역에 흩어져 있는 반면(빨간 점선 표시), RCS 적용 후에는 예측 density 응답이 실제 객체 군집 근처에서 더 연속적이고 집중된 구조를 형성하고, 희소 장면에서는 배경의 산발적 응답이 억제됨 — RCS가 "density map의 시각적 형태"뿐 아니라 "query 배정과의 일치도" 자체를 개선한다는 정성적 근거.

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- Query priority가 confidence와 density의 단순 곱(`s·d̂`)이라 → <mark style="background: #FF5582A6;">density 응답이 낮은 고립된 소형 객체(주변에 다른 객체가 없는 경우)는 confidence가 높아도 priority가 함께 억제될 위험이 있다 — 논문은 이런 "고립된 객체" 케이스를 별도로 분석하지 않는다.</mark>
- BCME가 encoder의 모든 스케일에 density 기반 modulation을 가하므로 → <mark style="background: #FF5582A6;">ADPG의 density map 예측이 애초에 부정확하면 이 오류가 encoder 표현 전체로 전파될 위험이 있으나, 이런 실패 케이스에 대한 강건성 분석은 논문에 없다.</mark>

### 한계
- <mark style="background: #FF5582A6;">이 논문은 SSRN 프리프린트로 아직 동료 심사(peer review)를 거치지 않은 미출판 상태다 — 논문 표지 자체에 "This preprint research paper has not been peer reviewed"라고 명시되어 있다.</mark>
- <mark style="background: #FF5582A6;">저자가 Conclusion에서 직접 명시: 향후 과제로 "더 가벼운 density prior 모델링"과 "더 세분화된 query 배정 전략"을 향후 연구로 남김 — 현재 버전의 추가 연산 비용에 대한 정량적 분석(FLOPs, FPS 등)이 논문에 제시되지 않는다.</mark>
- ADPG의 multi-branch conv(5개 branch, kernel 3~11)가 추가 파라미터·연산을 요구하는데, 이 오버헤드가 baseline 대비 얼마나 늘었는지 구체적 수치가 없다(파라미터 총합만 11.90M으로 보고, D-Fine-S의 10.18M 대비 증가분만 확인 가능).
- GT density map 생성 방식(가우시안 커널 파라미터 등 세부 구현)이 본문에 상세히 기술되지 않아 재현성 확인이 어렵다.

### 생각할 점
- <mark style="background: #A6E3A1A6;">이 논문은 이 위키의 dynamic query DETR 계열([[DQ-DETR]], Density-Aware DETR, IG-DETR, DQA-DETR) 중 density 정보를 가장 포괄적으로 활용한다 — encoder 표현 강화(BCME), query 개수 추정, 개별 토큰 순위 결정(RCS) 세 지점 모두에 관여시킨 유일한 사례다.</mark>
- <mark style="background: #A6E3A1A6;">RCS의 margin ranking loss는 "절대적으로 정확한 density map"이 아니라 "상대적으로 올바른 순위"만 요구한다는 점에서, 다른 dynamic query 논문들의 pixel-level regression 목표와 근본적으로 다른 감독 방식이다 — 이 상대적 순위 학습이 절대값 회귀보다 노이즈에 강건할 가능성이 있어 보인다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">RCS의 "GT로 참조 우선순위를 만들고 margin ranking loss로 감독" 방식은, [[Unc-SOD]]의 instance-level uncertainty 기반 접근과 결합해 "밀도 기반 순위"와 "불확실성 기반 순위"를 동시에 반영하는 하이브리드 priority를 만들어볼 여지가 있다.</mark>
- <mark style="background: #A6E3A1A6;">Query priority가 confidence·density의 단순 곱이라는 한계(Discussion 참고)를 보완하려면, [[DQA-DETR]]의 attention 기반 병합처럼 고립된 저밀도 고신뢰 토큰을 별도 경로로 구제하는 메커니즘을 추가하는 방향도 검토할 만하다.</mark>

> [!info] 내 메모
> 

# 관련 개념
- [[Density_Guided_Dynamic_Query]] — 이 논문의 핵심 기여가 등재된 개념. Density map을 encoder memory 강화(BCME), query 개수 추정, token 순위 결정(RCS) 세 곳 모두에 참여시킨 가장 포괄적인 사례로 기록됨.
- [[Dilated_Convolution]] — ADPG의 multi-branch 설계(여러 수용영역을 병렬로 두는 목적)와 개념적으로 유사하나, 이 논문은 dilation이 아니라 커널 크기 자체를 다양화(3~11)한다는 차이가 있음.
- [[Bipartite_Matching_Hungarian_Algorithm]] — 최종 예측-정답 매칭에 사용되는 D-Fine-S 표준 구조의 기반.

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]] — dynamic query DETR 계열 중 density 정보를 가장 포괄적으로 활용하는 사례.

# 읽어볼 만한 논문
- 참고문헌 기반: Y.-X. Huang, H.-I. Liu, H.-H. Shuai, W.-H. Cheng, "DQ-DETR: DETR with dynamic query for tiny object detection" (ECCV 2024) [24] — 이미 위키에 있음: [[DQ-DETR]]. 이 논문이 직접 비교 대상으로 삼는 dynamic query DETR의 원조.
- 참고문헌 기반: Y. Peng, H. Li, P. Wu et al., "D-FINE: Redefine regression task in DETRs as fine-grained distribution refinement" (ICLR 2025) [47] — 이 논문의 baseline. Baseline 구조를 이해하지 못하면 ADPG/BCME/RCS가 어디에 삽입되는지 파악하기 어려움.
- 참고문헌 기반: Y. Ma, X. Wei, X. Hong, Y. Gong, "Bayesian loss for crowd count estimation with point supervision" (ICCV 2019) [20] — Density map 추정의 원조 계보 중 하나. ADPG의 density prediction head 설계 배경 이해에 도움.
- 자유 추천(검증 필요): Margin ranking loss를 object detection의 query/anchor 우선순위 학습에 적용한 다른 연구 — 검색 키워드: `margin ranking loss query priority object detection token selection`. RCS와 유사한 상대적 순위 감독 방식이 다른 detection 프레임워크에도 적용된 사례가 있는지 확인할 가치가 있음.
