---
title: "Multi-Head Self-Attention"
tags: [architecture, attention]
created: 2026-08-28
updated: 2026-08-28
---

# 역할
시퀀스(토큰/픽셀 등 원소들의 나열) 안에서 **각 원소가 다른 모든 원소를 얼마나 참고해야 하는지**를 계산해서, 그 참고 비율(가중치)만큼 정보를 섞어주는 레이어. CNN의 conv가 "가까운 이웃"만 보는 것과 달리, self-attention은 **거리와 무관하게 시퀀스 전체를 한 번에** 본다.

# 구조

## 입력/출력
- 입력: 시퀀스 `X` — shape `(N, d)` (N: 원소 개수, d: 채널/임베딩 차원)
- 출력: 같은 shape `(N, d)` — 각 원소가 "다른 원소들의 정보를 반영해 업데이트된" 벡터로 바뀐다. 개수(N)와 차원(d)은 그대로 유지된다.

## 내부 동작 (Query/Key/Value)
1. **Q, K, V 생성**:
   입력 `X (N, d)`에 각각 다른 학습 가능한 선형변환(fully connected layer) `W_Q, W_K, W_V (d, d)`를 곱해 세 가지 벡터를 만든다.
   `Q = X·W_Q`, `K = X·W_K`, `V = X·W_V` — 셋 다 shape `(N, d)`.
   - Q(query, 질의): "나는 무엇을 찾고 있는가"
   - K(key, 열쇠): "나는 무엇을 갖고 있는가" (Q와 비교되는 대상)
   - V(value, 값): "실제로 전달할 정보"
2. **유사도 계산**: `Q·Kᵀ` → shape `(N, N)`. i번째 행, j번째 열의 값은 "i번째 원소가 j번째 원소와 얼마나 관련 있는지"를 나타내는 점수(내적).
3. **정규화(softmax)**: 각 행을 softmax에 통과시켜 합이 1이 되는 가중치로 만든다(스케일링을 위해 보통 `√d`로 나눈 뒤 softmax). → attention weight `α (N, N)`.
4. **가중합**: `α·V` → shape `(N, d)`. 각 원소가 "다른 모든 원소의 value를 attention weight만큼 섞어서" 받은 최종 출력.

## Multi-head란
위 과정을 차원 `d` 전체로 한 번에 하지 않고, `d`를 `M`개의 head로 쪼개서(각 head 차원 `d' = d/M`) **M번 병렬로 독립 수행**한 뒤 결과를 다시 concat하고 선형변환(`L`)으로 합친다.
- 왜 나누는가: 한 attention이 "물체의 위치 관계"만 보는 동안, 다른 head는 "색깔이 비슷한 것"을 보는 식으로 **서로 다른 관점을 동시에** 학습하게 하려는 의도.

> [!example]- 수식 (DETR 논문 Appendix A.1 표기 기준)
> ```
> attn(X_q, X_kv, T') 함수:
>   [Q; K; V] = [T1'(X_q+P_q); T2'(X_kv+P_kv); T3'(X_kv)]
>   α_ij = softmax_j( Q_i · K_j / √d' )
>   출력_i = Σ_j α_ij · V_j
>
> multi-head: M개 head를 독립 계산 → concat → 선형변환 L
> ```
> `P_q`, `P_kv`는 positional encoding(위치 정보를 더해주는 벡터) — attention 자체는 순서 정보가 없어서(permutation-invariant) 별도로 위치 정보를 주입해야 한다.

# 왜 이렇게 되는가
- **왜 거리 제약이 없는가**: conv는 커널 크기만큼만 이웃을 보지만, attention은 `Q·Kᵀ`가 모든 쌍(i, j)의 조합을 다 계산하므로 이론상 이미지/시퀀스 전체를 한 번에 참조할 수 있다 — 멀리 떨어진 두 객체 사이의 관계도 레이어 하나로 즉시 반영 가능.
- **왜 self-attention인가**: Q, K, V가 전부 같은 입력 `X`에서 나오면 "self"(자기 자신끼리 참조) attention, Q는 디코더에서 K/V는 다른 시퀀스(예: 인코더 출력)에서 오면 "cross-attention"이라 부른다 — 계산 방식은 동일하고 Q/K/V의 출처만 다르다.
- **비용**: 시퀀스 길이 N에 대해 `Q·Kᵀ`가 `(N, N)` 행렬이므로 계산량이 `O(N²)`로 커진다 — 이미지처럼 N(=H×W)이 큰 경우 비용이 크다는 게 흔한 트레이드오프.

# 등장 논문
- [[DETR]] — 인코더의 self-attention(이미지 feature 전체를 서로 참조), 디코더의 self-attention(object query끼리 서로 참조해 중복 예측 억제) 두 곳에 사용.
