---
title: "Uncertainty-Guided Refinement for Fine-Grained Salient Object Detection (원문 요약)"
tags: [paper-source]
created: 2026-08-04
---

[[uncertainty-guided-refinement]]의 원문 요약. 분석·평가는 담지 않는다 — 순수 번역/발췌.

# Abstract
최근 salient object detection(SOD) 방법들은 인상적인 성능을 달성했다. 그러나 기존 방법들이 예측한 salient 영역에는 대개 저채도(unsaturated) 영역과 그림자가 포함되어 있어, 모델이 신뢰할 수 있는 fine-grained 예측을 하는 데 한계가 있다. 이를 해결하기 위해 본 논문은 SOD에 불확실성 가이던스 학습(uncertainty guidance learning) 접근법을 도입하여, 모델의 불확실 영역에 대한 인지 능력을 향상시키고자 한다. 구체적으로, Multilevel Interaction Attention(MIA) 모듈, Scale Spatial-Consistent Attention(SSCA) 모듈, Uncertainty Refinement Attention(URA) 모듈이라는 세 가지 중요한 구성 요소를 결합한 새로운 Uncertainty Guided Refinement Attention Network(UGRAN)를 설계한다. 특징을 강화(enhance)하는 데 주력하는 기존 방법들과 달리, 제안하는 MIA는 멀티레벨 특징들 간의 상보적 특성을 활용하여 멀티레벨 특징의 상호작용과 인지를 촉진한다.

그다음, 제안하는 SSCA를 통해 집계된 특징 내의 다양한 스케일에 걸친 salient 정보를 더 포괄적이고 온전하게 통합할 수 있다. 이어지는 단계에서는 saliency 예측 맵으로부터 생성한 불확실성 맵을 활용해 모델의 불확실 영역에 대한 인지 능력을 강화함으로써, 고채도(highly-saturated)의 fine-grained saliency 예측 맵을 생성한다. 추가로, URA 모듈의 계산 부담을 최소화하고 불확실성 가이던스의 활용도를 개선하기 위해 adaptive dynamic partition(ADP) 메커니즘을 고안한다.

7개의 벤치마크 데이터셋에서의 실험은 제안하는 UGRAN이 기존 최신(state-of-the-art) 방법론들보다 우수함을 보여준다. 코드는 https://github.com/I2-Multimedia-Lab/UGRAN 에 공개될 예정이다.

# Introduction

#### SOD의 역할과 현재 우위 접근
Salient Object Detection(SOD)은 이미지에서 시각적으로 가장 눈에 띄는 객체·영역을 식별하는 것으로, object detection·semantic segmentation·image retargeting·video summarization 등 다운스트림 task에서 핵심 역할을 한다. CNN 기반 encoder-decoder와 최근의 Transformer 구조 모두 계층적으로 멀티레벨 특징을 집계해 saliency를 예측한다.

#### 기존 연구 한계 지적 — 저수준 feature 집계의 한계
객체 위치 파악은 잘 되지만 경계 근처 fine-grained 예측은 여전히 어렵다. DSS·NLDF류는 저수준 feature를 끌어와 이를 보강하려 하지만, 저수준 feature에는 노이즈와 non-salient 정보가 많이 섞여 있어 실제로 경계 예측에 기여하는 정보는 일부에 불과하다.

#### 기존 연구 한계 지적 — boundary guidance의 한계
EGNet·PoolNet·F3Net 등은 경계(boundary) 정보로 fine-grained 예측을 가이드하지만, 훈련·추론 전 단계에서 고정된 prior에 의존해 모델이 실제로 어디를 저채도로 예측하는지에 적응하지 못한다.

#### 제안 방법 개요
이를 극복하기 위해 MIA, SSCA, URA 모듈과 Adaptive Dynamic Partition(ADP) 메커니즘을 결합한 UGRAN 프레임워크를 제안한다.

#### MIA·SSCA 설계 근거
멀티레벨 특징 활용을 재고해, (1) 스케일별 특성을 반영한 특징 강화와 (2) 집계된 특징 내 공간적 불일치 해소가 필요하다는 결론에 도달했다. 이에 따라 레벨 간 상호작용·인지를 촉진하는 MIA와, 다운샘플링으로 전역 salient 표현을 포착하는 SSCA를 제안하며 두 모듈은 상호 보완한다.

#### URA 설계 근거
불확실성 가이던스 학습을 도입한다. 현재 예측 맵에서 생성한 불확실성 맵으로 저채도·아티팩트 영역을 식별해 마스크로 삼고, 정제 특징과 저수준 특징 간 masked attention을 계산하는 URA 모듈은 기존 방법보다 더 명시적인(explicit) 가이드 학습 방식이다.

#### ADP 설계 근거
불확실 영역이 전체 saliency 맵의 약 2~5%에 불과해 전역적으로 mask attention을 계산하면 비효율적이므로, sharp한 영역은 세분화하고 blur가 심한 영역은 분할을 멈추는 동적 추론 ADP를 설계했다.

#### 기여 요약
저채도 영역·아티팩트 문제를 불확실성 가이드 기반의 동적·반복적 정제로 최초로 해결했으며, 제안 모듈을 통합한 새 네트워크를 7개 데이터셋에서 정량·정성적으로 검증했다.

# Main Contribution
1. 멀티레벨 feature를 효과적으로 활용해 salient 객체를 정확히 위치시키는 두 모듈, Multilevel Interaction Attention(MIA)과 Scale Spatial Consistent Attention(SSCA)을 설계했다.
2. Boundary guidance의 한계를 완화하기 위해 불확실성 가이던스 학습 접근법을 도입하고, 불확실 영역 인지 능력을 높이는 Uncertainty Refinement Attention(URA) 모듈을 설계했다.
3. 계산 비용을 줄이고 불확실성 정제를 큰 공간 스케일에 적응시키는 Adaptive Dynamic Partition(ADP) 메커니즘을 제안했다.

# Conclusion
본 논문에서는 saliency 예측 맵에서 흔히 발생하는 그림자와 저채도(under-saturated) 영역 문제를 해결하기 위해, fine-grained salient object detection(SOD)을 위한 불확실성 가이던스 학습 접근법을 제안한다. 먼저, Multilevel Interaction Attention(MIA) 모듈과 Scale Spatial-Consistent Attention(SSCA) 모듈을 도입하여 기존 집계(aggregation) 방법들의 한계를 완화하고 고품질의 salient 정보를 추출한다. 그다음, 예측 맵에서 아티팩트와 저채도 영역을 효과적으로 제거하는 Uncertainty Refinement Attention(URA) 모듈을 설계한다. URA 모듈은 불확실성 가이던스를 활용해 모델의 불확실 영역에 대한 인지를 개선하고, 최종적으로 고채도의 fine-grained saliency 예측을 생성한다. 추가로, 모델 성능과 계산 비용 사이의 균형을 맞추기 위해 Adaptive Dynamic Partition(ADP) 메커니즘을 도입한다. 특히, 본 논문의 정제(refinement) 접근법은 다른 방법에도 매끄럽게 통합될 수 있어, 다른 이진 이미지 분할(binary image segmentation) task에도 적용될 유망한 잠재력을 보인다. 마지막으로, 위 구성 요소들을 통합한 새로운 네트워크 UGRAN을 제시한다. 실험 결과는 널리 사용되는 7개 데이터셋에서 제안 네트워크의 우수한 성능을 입증한다.
