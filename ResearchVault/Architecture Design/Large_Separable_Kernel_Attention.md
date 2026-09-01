---
title: "Large Separable Kernel Attention (LSKA)"
tags: [architecture, attention, spatial-attention, large-kernel]
created: 2026-08-31
updated: 2026-08-31
---

# 역할
큰 커널의 convolution attention(예: Visual Attention Network의 LKA)이 갖는 넓은 수용영역의 이점은 유지하면서, 정사각형 대형 커널을 그대로 쓸 때 발생하는 파라미터·연산량 폭증을 없애기 위해 **하나의 큰 커널을 수평(1×k) + 수직(k×1) 1D depthwise convolution 쌍으로 분해**하는 spatial attention 기법이다. "큰 수용영역"과 "적은 연산량"을 동시에 달성하는 것이 목적이다.

# 구조
## 입력/출력
- 입력: feature map `F (C, H, W)`
- 출력: `F̄ (C, H, W)` — attention map을 곱해 정제된 feature, shape 불변.

## 내부 동작
1. **수평/수직 depthwise convolution**:
   정사각형 대형 커널 `(2d-1)×(2d-1)` 대신, 수평 방향 `1×(2d-1)`와 수직 방향 `(2d-1)×1` depthwise conv를 순차 적용해 넓은 문맥을 저비용으로 확보한다.
   `Z̄ᶜ = Σ_HW W^C_(2d-1)×1 * (Σ_HW W^C_1×(2d-1) * Fᶜ)`
2. **Depthwise dilated convolution (1D 분해)**:
   여기에 dilation rate `d`를 적용한 depthwise-dilated convolution도 같은 방식으로 수평·수직 1D 커널로 분해해, 더 넓은 유효 수용영역을 추가 확보한다.
   `Zᶜ = Σ_HW W^C_(k/d)×1 * (Σ_HW W^C_1×(k/d) * Z̄ᶜ)`
3. **Attention map 생성 및 적용**:
   위 결과를 1×1 conv([[1x1_Convolution]])로 채널 혼합해 attention map `A^C`를 얻고, 원본 feature `F̄ᶜ`와 Hadamard product(원소별 곱)로 결합한다.
   `A^C = W_1×1 * Z^C`, `F̄^C = A^C ⊗ F^C`

> [!example]- 대형 커널을 분해하는 이유
> `(2d-1)×(2d-1)` 크기의 2D convolution 파라미터 수는 `O(k²)`으로 커널이 커질수록 급격히 증가한다. 이를 `1×k`와 `k×1` 두 개의 1D convolution으로 분해하면 파라미터·연산량이 `O(k²)`에서 `O(2k)`로 줄어들면서도, 두 연산을 순차 적용한 유효 수용영역은 원본 2D 커널과 동일하게 유지된다 — separable convolution의 표준적인 이점을 large-kernel attention에 적용한 것.

# 왜 이렇게 되는가
- **왜 큰 수용영역이 필요한가**: Self-attention([[Multi_Head_Self_Attention]])처럼 전역 문맥을 직접 모델링하려면 `O((HW)²)` 비용이 들지만, 큰 커널의 convolution attention은 훨씬 적은 비용으로 넓은 문맥을 흉내낼 수 있다. 다만 커널을 그대로 키우면 다시 파라미터·연산이 커지는 딜레마가 생긴다.
- **왜 1D로 분해하는가**: 2D 대형 커널의 연산을 두 개의 1D depthwise convolution(수평+수직)으로 순차 적용해도 수학적으로 유효 수용영역은 동일하게 유지되면서, 파라미터·FLOPs는 커널 크기에 선형으로만 비례하게 된다.
- **비용/트레이드오프**: Depthwise 분해이므로 채널 간 상호작용은 별도의 1×1 conv에 의존한다 — 공간 정보(수평·수직 1D conv)와 채널 정보(1×1 conv)를 단계적으로 분리해 처리하는 구조라는 점에서, 완전한 2D joint 커널보다는 표현력이 다소 제한될 수 있다.

# 등장 논문
- [[LSOD-YOLO]] — SPPFL(Spatial Pyramid Pooling Fusion with LSKA) 모듈에서, SPPF의 multi-scale pooling 출력에 LSKA를 적용해 LCOR로 단순화된 계층 구조에서 손실될 수 있는 semantic/global 정보를 attention 가중치 재분배만으로 보완하는 데 사용.
