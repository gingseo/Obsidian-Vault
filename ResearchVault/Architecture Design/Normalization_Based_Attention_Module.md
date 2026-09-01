---
title: "Normalization-based Attention Module (NAM)"
tags: [architecture, attention, channel-attention, spatial-attention, batch-normalization]
created: 2026-08-31
updated: 2026-08-31
---

# 역할
CBAM류의 channel+spatial attention 구조를 계승하되, attention 가중치를 처음부터 새로 학습하지 않고 **이미 계산되어 있는 Batch Normalization(BN)의 scaling factor를 그대로 채널·공간 중요도로 재활용**하는 경량 attention 기법. 추가 파라미터·연산 비용을 거의 들이지 않고 "덜 중요한 feature를 억제"하는 것이 목적이다.

# 구조
## 입력/출력
- 입력: feature map `F (C, H, W)`
- 출력: `M (C, H, W)` — channel attention과 spatial attention을 순차 적용해 비중요 feature가 억제된 feature map. shape 불변.

## 내부 동작
1. **Channel attention (BN scaling factor 재사용)**:
   Batch Normalization의 scaling factor `γ`(채널별 분산 정도를 나타내는 학습 파라미터)를 채널 중요도의 척도로 삼는다. 입력 `F₁`을 BN에 통과시킨 뒤, 각 채널의 정규화된 `γ` 값에 sigmoid를 취해 채널별 가중치 `W_γ`를 만든다.
   `M_c = sigmoid(W_γ(BN(F₁)))`, `W_γ_c = γ_c / Σ_j γ_j`
2. **Spatial attention (pixel normalization scaling factor 재사용)**:
   같은 원리를 공간 축에도 적용한다 — pixel normalization의 scaling factor `λ`를 공간적 중요도로 삼아, 살아남을 픽셀에 더 높은 가중치를 부여한다.
   `M_s = sigmoid(W_λ(BN_s(F₂)))`, `W_λ_i = λ_i / Σ_j λ_j`
3. **Sparsity 정규화**:
   손실 함수에 `γ`, `λ`에 대한 sparsity 정규화 항을 추가해, 비중요 feature/픽셀의 가중치가 0에 가까워지도록 학습을 유도한다.
   `Loss = Σ l(f(x,W), y) + p·Σ g(γ) + p·Σ g(λ)`

> [!example]- CBAM과의 구조적 차이
> CBAM은 channel/spatial attention 가중치를 별도의 FC/conv 레이어로 새로 학습한다. NAM은 이미 backbone 학습 과정에서 계산되는 BN의 `γ`(channel)·`λ`(spatial pixel normalization) 파라미터를 "재사용"하므로, 사실상 추가 파라미터 없이 attention을 구현한다 — Table 8(LSOD-YOLO)에서 C2f-N 적용 전후 파라미터가 3.755M→3.755M로 사실상 불변인 것이 이를 보여준다.

# 왜 이렇게 되는가
- **왜 BN scaling factor가 중요도의 좋은 척도인가**: BN의 `γ`는 학습 과정에서 "이 채널의 activation이 얼마나 분산되어(=변별력 있게) 있는지"를 반영하도록 최적화된다 — 분산이 크다는 것은 그 채널이 입력에 따라 크게 달라지는, 즉 판별적인(informative) 신호를 담고 있다는 의미로 해석할 수 있다.
- **왜 저비용인가**: 별도의 attention 파라미터를 새로 두지 않고, 이미 존재하는 정규화 계층의 파라미터를 재해석해서 쓰기 때문에 attention을 위한 추가 학습 파라미터가 거의 없다 — SE([[Squeeze_And_Excitation_Channel_Attention]])나 CBAM이 FC/conv 레이어를 추가로 두는 것과 대비된다.
- **비용/트레이드오프**: 중요도 척도가 BN 파라미터라는 간접 신호에 묶여 있어, self-attention이나 CBAM처럼 입력 content에 직접 조건화된 정교한 attention만큼의 표현력은 없을 수 있다.

# 등장 논문
- [[2025_ESWA_LSOD-YOLO|LSOD-YOLO]] — C2f 모듈의 residual branch 출력에 NAM을 삽입한 C2f-N 구조로, Neck의 저해상도 feature map에 적용. Table 8 ablation에서 CBAM(mAP0.5 36.6)·CA(36.7)·SA(36.7)·SE(36.9) 대비 C2f-N(37.0)이 최소 파라미터로 최고 성능을 기록.
