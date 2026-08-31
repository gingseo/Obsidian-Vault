---
title: "Squeeze-and-Excitation Channel Attention"
tags: [architecture, attention, channel-attention]
created: 2026-08-28
updated: 2026-08-28
---

# 역할
Feature map의 채널마다 "이 채널이 지금 얼마나 중요한 정보를 담고 있는지"를 스칼라 값 하나로 요약(squeeze)한 뒤, 그 값을 다시 채널별 가중치로 변환(excitation)해 원본 feature에 곱해주는 기법. 공간(H, W) 축은 건드리지 않고 **채널 축만 선택적으로 증폭/억제**한다 — "어디를 볼지"가 아니라 "어떤 채널을 믿을지"를 학습하는 attention이라는 점에서 spatial attention과 구별된다.

# 구조

## 입력/출력
- 입력: feature map `x (C, H, W)`
- 출력: `x̂ (C, H, W)` — shape은 그대로, 채널마다 다른 스칼라 가중치가 곱해져 값만 재조정된다.

## 내부 동작
1. **Squeeze(압축)**:
   공간 차원 `(H, W)`을 채널마다 스칼라 하나로 압축한다. Global Average Pooling(GAP, 채널의 평균값)을 기본으로 쓰고, 변형에서는 Global Max Pooling(GMP, 채널의 최댓값)을 함께 써서 두 통계를 모두 반영하기도 한다.
   `avgp_c = (1/HW) Σ_{h,w} x(c,h,w)`, `maxp_c = max_{h,w} x(c,h,w)` — 결과는 채널 개수만큼의 벡터 `(C,)`.
2. **Excitation(재조정)**:
   압축된 벡터를 1×1 conv(또는 FC layer, [[1x1_Convolution]] 참고) 두 개를 거치게 해 채널 간 비선형 상호작용을 학습한 뒤, sigmoid로 0~1 범위의 채널별 가중치 `W_c`를 만든다.
   `W_c = σ(Conv₁(Conv₂(avgp)) + Conv₁(Conv₂(maxp)))` (GMP 분기를 추가한 변형 기준)
3. **Scale(재적용)**:
   `W_c`를 원본 feature `x`의 각 채널에 곱한다(broadcasting) — `x̂ = x · W_c`.

> [!example]- 원조 SE-block과의 차이
> - 원조 Squeeze-and-Excitation(Hu et al., CVPR 2018)은 GAP 하나만 쓰고, excitation을 FC(채널 축소) → ReLU → FC(채널 복원) → sigmoid로 구성한다.
> - 이후 변형들은 GAP에 GMP를 더해 "평균적으로 강한 채널"뿐 아니라 "국소적으로 튀는 채널"까지 함께 반영하거나, FC 대신 1×1 conv를 써서 공간 차원이 있는 중간 feature(예: RoI feature)에도 바로 적용할 수 있게 한다.

# 왜 이렇게 되는가
- **왜 채널마다 다른 가중치가 필요한가**: convolution은 모든 채널을 동등하게 다음 레이어에 넘기지만, 실제로는 특정 채널이 노이즈·배경 정보를, 다른 채널이 객체의 판별적 정보를 더 많이 담고 있을 수 있다. Squeeze-Excitation은 이 "채널별 신뢰도"를 데이터로부터 학습해, 유용한 채널의 신호를 키우고 불필요한 채널의 신호를 줄인다.
- **왜 GAP(+GMP)로 압축하는가**: 채널 전체의 공간 정보를 스칼라 하나로 요약해야 이후 단계가 "채널 개수" 크기의 작은 벡터만 다루면 되므로 연산량이 매우 작다 — attention이지만 self-attention([[Multi_Head_Self_Attention]])처럼 `O(N²)` 비용이 들지 않는다.
- **비용/트레이드오프**: 공간적으로 "어느 위치가 중요한지"는 구별하지 못한다(채널 전체에 같은 가중치가 곱해짐) — 위치별 선택성이 필요하면 spatial attention과 결합해야 한다.

# 등장 논문
- [[FANet]] — CAREM(Channel Attention-based RoI Enhancement Module)에서, RoI feature의 고주파 성분(2D-DCT 기반 Gaussian 필터로 추출)에 대해 GMP+GAP 두 분기를 1×1 conv로 합산 후 sigmoid로 채널 가중치를 만들어, "어떤 채널이 tiny object의 고주파 특징을 잘 대변하는지"를 학습하는 데 사용.
