---
title: "Mixture-of-Experts Top-k Sparse Gating"
tags: [architecture, routing, sparse-computation]
created: 2026-08-28
updated: 2026-08-28
---

# 역할
전문성이 다른 여러 개의 작은 네트워크("expert") 중에서, **입력마다 그 입력에 맞는 일부(전부 아님)만 선택적으로 활성화**해 계산하는 구조. 모든 expert를 항상 다 쓰면(dense) 파라미터가 늘어난 만큼 연산량도 그대로 늘어나지만, 이 방식은 expert 개수(용량)를 늘려도 실제 활성화되는 expert 수(연산량)는 고정할 수 있어 "표현력은 크게, 계산은 적게"라는 트레이드오프를 가능케 한다.

# 구조

## 입력/출력
- 입력: 라우팅 기준이 되는 feature `X`(예: `(H×W, C)` 또는 `(N, C)`), `D`개의 expert network, 학습 가능한 라우터 파라미터 `W_router`.
- 출력: 활성화된 상위 `ρ`개(`ρ < D`) expert의 출력을 라우터가 계산한 가중치로 조합한 최종 표현. 라우터의 결정 자체는 sparse gating matrix `G`(대부분 0, 상위 ρ개 위치만 0이 아닌 값)로 표현된다.

## 내부 동작
1. **Gating logit 계산**: 입력 `X`를 라우터(경량 네트워크, 보통 1×1 conv나 FC 하나)에 통과시켜 `D`개 expert 각각에 대한 점수(logit)를 얻는다 — `logits = W_router · X`, shape `(D,)` (위치별로 독립 계산하면 `(H×W, D)`).
2. **Top-k(top-ρ) 선택**: `D`개 logit 중 값이 큰 상위 `ρ`개만 남기고 나머지는 `-∞`(또는 마스킹)로 지워 "이 위치는 이 ρ개 expert만 쓴다"고 결정한다.
3. **Softmax 정규화**: 살아남은 ρ개 logit에만 softmax를 적용해 합이 1인 가중치로 만든다 — 나머지 `D-ρ`개는 이 과정에서 자동으로 0이 된다. 이 결과가 sparse gating matrix `G`.
4. **가중합**: 각 expert network에 입력을 통과시킨 뒤, `G`의 값(0이 아닌 ρ개 항목)으로 가중합해 최종 출력을 만든다 — 선택되지 않은 expert는 아예 forward pass를 생략할 수 있어 실제 연산량도 ρ개 분량으로 줄어든다.

> [!example]- 수식 (일반형)
> ```
> logits = W_router · X                              # (D,)
> G = Softmax(Top-ρ(logits))                         # 상위 ρ개만 값을 갖고 나머지는 0
> output = Σ_{i∈TopK} G_i · Expert_i(X)
> ```
> `Top-ρ(·)`는 상위 ρ개를 제외한 나머지를 `-∞`로 치환하는 연산 — 그 뒤 softmax를 취하면 자동으로 0이 된다.

# 왜 이렇게 되는가
- **왜 top-k만 쓰는가(dense 대신)**: expert 개수 `D`를 늘리면 "다양한 상황에 맞는 특화된 지식"을 더 많이 저장할 수 있지만, 매번 D개를 전부 계산하면 연산량이 D에 비례해 커진다. 상위 ρ개만 활성화하면 D를 키워도(용량 증가) 연산량은 ρ로 고정되므로, 저장 용량과 연산 비용을 분리할 수 있다.
- **왜 "winner-takes-all" 방식이 중요한가**: 모든 expert를 매번 균등하게 섞으면(uniform routing) 서로 다른 expert가 각자 다른 패턴에 특화될 유인이 사라진다 — 특정 입력에 강하게 반응하는 소수만 선택되게 강제해야, expert별로 서로 다른 패턴에 특화된 "해석 가능한" 분업 구조가 형성된다.
- **한계**: 라우터가 무엇을 기준으로 expert를 고르는지(라우팅에 쓰는 feature의 품질)에 따라 라우팅 품질이 크게 좌우된다 — 얽혀 있거나(entangled) 노이즈가 섞인 feature로 라우팅하면, sparsity를 줘도 무작위 선택과 큰 차이가 없어질 수 있다(아래 등장 논문 참고).

# 등장 논문
- [[Detection_Oriented_Rectification]] — task-specific router가 학습된 degradation basis `B_d`에 대한 Top-ρ(ρ=4, D=8) 게이팅으로 rectification prompt를 합성. 라우팅 기준을 "얽힌 원본 feature"가 아니라 "열화 semantic으로 재문맥화된 feature"로 바꿔 라우팅 품질을 크게 높였음(Uniform 29.7% → Degradation-aware 30.4% AP).
