---
title: "Global Context Modeling (GAP/GMP 기반)"
tags: [architecture, attention, global-context]
created: 2026-08-28
updated: 2026-08-28
---

# 역할
Feature map의 **모든 픽셀 쌍 사이의 관계**를 계산해 채널·공간 방향 전역(global) 문맥 정보를 얻는 구조. Self-attention과 목적은 같지만(멀리 떨어진 위치도 한 번에 참조), Query/Key를 전부 픽셀 단위로 생성하는 대신 **GAP(Global Average Pooling)·GMP(Global Max Pooling)로 먼저 정보를 압축**해 연산량을 크게 줄인 경량 버전이다. NLNet(Non-local Network) → GCNet → SCP로 이어지며 계속 간소화된 계보의 최신 형태.

# 구조

## 입력/출력
- 입력: feature map `(C, H, W)`
- 출력: 같은 shape `(C, H, W)` — 입력에 "전역 문맥이 반영된 보정값"을 더하거나 곱해 갱신한 결과.

## 내부 동작
1. **압축(GAP/GMP)**:
   입력 `(C, H, W)`를 공간 축으로 평균(GAP)·최대(GMP) pooling해 `(C, 1, 1)` 두 벡터를 얻는다. 이 둘을 concat/결합해 "이 feature map 전체가 어떤 채널에 얼마나 반응했는지"를 압축한 요약 정보로 쓴다.
2. **Value/Query-Key 생성**:
   원본 feature에 [[1x1_Convolution]]을 적용해 value(정보 전달용)와 query·key(유사도 계산용)를 만든다 — 픽셀별로 만들지만 채널 수는 1×1 conv로 줄여 비용을 낮춘다.
3. **행렬곱으로 문맥 계산**:
   압축된 요약 정보(GAP/GMP 결과)와 query-key, value를 행렬곱해 "채널 방향 문맥"과 "공간 방향 문맥"을 각각 얻는다 — 전체 `(HW, HW)` 크기 attention map을 만드는 대신 `(C, 1)`이나 `(HW, 1)` 크기로 압축된 벡터끼리만 곱하므로 NLNet의 `O((HW)²)` 비용을 크게 줄인다.
4. **결합**: 두 문맥(채널/공간)을 broadcast Hadamard product(원소별 곱, 크기가 다른 두 텐서를 브로드캐스팅해 곱함)로 합쳐 최종 출력을 만든다.

> [!example]- NLNet → GCNet → SCP → 현재 구조로의 간소화 흐름
> - NLNet: 모든 픽셀 쌍의 pairwise correlation을 직접 계산 (`O((HW)²)`, 매우 비쌈).
> - GCNet: 1×1 conv + softmax로 모든 위치를 하나의 전역 attention map으로 근사해 비용을 크게 줄임(그러나 위치별 개별 정보는 손실).
> - SCP: GCNet에 pixel-wise value path(1×1 conv)를 추가해 개별 픽셀 정보를 보존하려 시도.
> - 최신 변형(GAP+GMP 결합): SCP의 pixel-wise path는 유지하되, 전역 정보 집약 단계에 GMP(최대값 — "가장 두드러진 신호")를 추가로 결합해 GAP(평균값 — "전반적 경향")만 쓸 때보다 채널 선택 정보를 보강.

# 왜 이렇게 되는가
- **왜 GAP만으로는 부족한가**: 평균은 배경처럼 넓게 퍼진 신호에 쏠리기 쉬워, 작고 두드러진 신호(소형 객체 같은)가 평균에 묻힐 수 있다. 최댓값(GMP)을 함께 쓰면 "가장 강하게 반응한 위치"라는 보완적 정보를 반영해 이 문제를 완화한다.
- **왜 pairwise correlation을 직접 계산하지 않는가**: `(HW, HW)` 크기의 attention map은 이미지 해상도가 커질수록 메모리·연산 비용이 제곱으로 증가한다([[Multi_Head_Self_Attention]]과 동일한 트레이드오프). GAP/GMP로 먼저 압축하면 이 비용을 선형에 가깝게 낮출 수 있다.
- **한계**: 압축 과정에서 세부 공간 정보 일부가 손실되므로, NLNet만큼 정교한 위치별 관계를 포착하지는 못한다 — 정확도와 비용 사이의 트레이드오프를 비용 쪽으로 기울인 설계.

# 등장 논문
- [[FFCA-YOLO]] — SCAM(Spatial Context Aware Module)에서 GAP+GMP로 전역 정보를 집약(GCNet/SCP 대비 GMP 추가)하고, 채널 방향·공간 방향 문맥을 각각 계산해 배경 혼동을 억제하는 데 사용.
