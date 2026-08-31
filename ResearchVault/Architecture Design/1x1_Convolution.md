---
title: "1x1 Convolution"
tags: [architecture, convolution]
created: 2026-08-28
updated: 2026-08-28
---

# 역할
커널 크기가 1×1인 convolution. 공간(가로·세로) 방향으로는 이웃 픽셀을 전혀 보지 않고, **채널 방향으로만 정보를 섞어서 채널 수를 바꾸는(늘리거나 줄이는) 용도**로 쓰인다. "각 위치에 독립적으로 적용되는 fully connected layer"라고 이해하면 정확하다.

# 구조

## 입력/출력
- 입력: feature map `(C_in, H, W)`
- 출력: `(C_out, H, W)` — **H, W(공간 크기)는 그대로**, 채널 수만 `C_in → C_out`으로 바뀐다.

## 내부 동작
- 각 공간 위치 `(h, w)`마다 그 위치의 `C_in`개 채널 값을 입력으로 받아, 학습된 가중치 행렬 `(C_out, C_in)`을 곱해 `C_out`개의 출력값을 만든다.
- 이 연산은 모든 `(h, w)` 위치에서 **동일한 가중치**로 반복된다 — 즉 "위치는 안 보고 채널만 섞는" 연산.
- 수식으로 보면 `y[c_out, h, w] = Σ_{c_in} W[c_out, c_in] · x[c_in, h, w] + b[c_out]` — 이건 각 픽셀 위치에서 독립적으로 수행하는 fully connected layer(선형변환)와 정확히 같은 식이다.

> [!example]- fully connected layer와의 관계
> - FC layer: 입력 벡터 `(C_in,)` → 출력 벡터 `(C_out,)`, 가중치 `(C_out, C_in)`.
> - 1x1 conv: 입력 `(C_in, H, W)`의 각 `(h, w)` 위치를 "C_in 차원 벡터"로 보고, 그 벡터마다 **같은** FC layer를 적용 → 출력 `(C_out, H, W)`.
> - 즉 1x1 conv = "공간의 모든 위치에 가중치를 공유하는 FC layer를 반복 적용"한 것과 동일한 연산.

# 왜 이렇게 되는가
- **왜 채널 축소/확장에 쓰는가**: 3x3, 5x5 같은 큰 커널은 파라미터 수가 `C_in × C_out × k²`로 커널 크기(k)에 비례해 커지는데, 1x1은 `k=1`이라 파라미터가 `C_in × C_out`만큼만 필요해 계산량 대비 채널 변환 효율이 가장 좋다.
- **왜 "위치 정보를 안 섞는다"는 게 중요한가**: transformer의 FFN(feed-forward network)이 "각 토큰(위치)마다 독립적으로 적용되는 2-layer network"인데, 이게 1x1 conv 2개를 이어붙인 것과 동일한 구조다 — 토큰 간 정보 교환은 attention이 담당하고, FFN/1x1conv는 "각 토큰 내부에서 채널 정보만 재조합"하는 역할 분담이 이뤄진다.

# 등장 논문
- [[DETR]] — (1) 트랜스포머 인코더 직전, CNN backbone의 출력 채널을 `2048 → d(=256)`로 줄이는 데 1x1 conv 사용. (2) 트랜스포머 내부 FFN이 "1x1 conv 2겹(with ReLU)"과 동일한 연산이라고 논문이 명시(Appendix A.1) — attention이 원소 간 정보를 섞고, FFN(=1x1conv)이 각 원소 내부에서 채널을 재조합하는 역할 분담.
