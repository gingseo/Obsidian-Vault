---
title: "Feature Information Driven Position Gaussian Distribution Estimation for Tiny Object Detection (원문 요약)"
tags: [paper-source]
created: 2026-08-04
---

[[feature-info-driven-gaussian]]의 원문 요약. 분석·평가는 담지 않는다 — 순수 번역/발췌.

# Abstract
Tiny object detection은 일반 detector들의 성공에도 불구하고 여전히 어려운 문제로 남아 있다. 일반 detector가 tiny object에서 성능이 크게 저하되는 주된 이유는 극도로 제한된 픽셀 수로 인한 약한 표현(representation) 때문이다. 이 문제를 해결하기 위해, 우리는 소멸된(extinguished) 영역을 강화하는 plug-and-play 아키텍처를 제안한다. 우리는 최초로 픽셀 단위 정보량(pixel-wise amount of information)이라는 관점에서 강화해야 할 영역을 탐색한다.

구체적으로, Information Entropy loss를 최소화함으로써 전체 이미지 픽셀의 feature information을 모델링하여, 약하게 활성화된 영역을 비지도 방식으로 attentively 강조하는 information map을 생성한다. 위 단계가 tiny object에 더 주의를 기울이도록 효과적으로 돕기 위해, 우리는 이어서 Position Gaussian Distribution Map을 도입한다. 이는 각 Gaussian 성분의 파라미터가 객체 인스턴스 라벨의 위치와 크기에 의존하는 Gaussian Mixture distribution으로 명시적으로 모델링되며, 추가적인 feature 강화를 위한 supervision 역할을 한다. Information map을 prior 지식으로 삼아, 우리는 multi-scale position Gaussian distribution map prediction 모듈을 구성하여, 학습 중 information map과 distribution map을 동시에 조절함으로써 tiny object에 집중하도록 한다.

세 개의 공개 tiny object 데이터셋에 대한 광범위한 실험은 현재 state-of-the-art 경쟁 기법들 대비 우리 방법의 우수성을 입증한다.

# Introduction

#### SOD의 역할과 현재 성능 격차
딥러닝 기반 아키텍처는 일반 object detection을 크게 발전시켰지만, tiny object(AI-TOD 기준 2–32px) 탐지에서는 성능이 급격히 저하되며, 교통 감시·해상 수색·야생동물 조사 등 실세계 적용에서 이 격차를 메우는 것이 중요하다.

#### 선행 접근 두 갈래 소개
이를 해결하기 위한 선구적 시도로 scale-aware feature fusion(계층적 feature 융합으로 spatial-semantic gap 해소)과 attention 기반 방법(feature map importance weight 모델링)이라는 두 갈래가 제시되었다.

#### 기존 연구 한계 지적
그러나 tiny object의 픽셀 수가 극히 제한된 경우 이런 방법들은 여전히 비효율적이다. 다운샘플링 반복으로 feature activation이 배경과 거의 구분 안 되게 억제되고, attention 기반 방법도 희소한 픽셀 탓에 local patch 내에서 배경이 attention map을 지배해 신뢰도가 낮아진다.

#### 제안 방법 개요
정보 손실을 겪는 비식별적 영역 강화가 핵심이라 보고, Information Entropy loss로 픽셀 단위 정보량을 비지도로 추정하는 information map과, 객체 위치·크기 기반 Position Gaussian Distribution Map(Gaussian Mixture)을 information map을 prior로 삼아 지도학습 예측하는 plug-and-play feature enhancement 프레임워크를 제안한다.

#### 기여 요약
세 데이터셋에 대한 광범위한 실험으로 우수성을 검증하며, 논문의 주요 기여를 요약해 제시한다.

# Main Contribution
1. 픽셀 단위 정보량 관점에서 tiny object의 약한 표현을 강화하는 최초의 시도. Attentive information map은 Information Entropy loss를 비지도 방식으로 최소화해 얻는다.
2. Information map을 prior guide로 삼아 multi-scale feature로 Position Gaussian Distribution Map을 예측해 tiny object를 더 부각시킨다.
3. 어떤 FPN 계열 detector에도 통합 가능한 plug-and-play 모듈이며, 실험 결과 일관된 성능 향상과 기존 SOTA 대비 우수성을 확인했다.

# Conclusion
이 논문은 픽셀 수준 feature information에 기반해 tiny object detection에서 약한 영역을 강화하는 새로운 접근을 제시한다. 우리는 Information Entropy loss를 비지도 방식으로 최소화하여, 더 많은 정보량을 가진 salient한 영역일수록 높은 값을 갖는 attentive information map을 생성한다. 그런 다음, information map이 tiny object를 더욱 부각시키도록 하기 위해, information map의 guidance 아래 지도학습 방식으로 Position Gaussian Distribution Map을 예측한다. 이 두 attentive map을 이용해 우리는 기존에 비식별적이었던 tiny object feature를 강화하며, 세 데이터셋에 대한 실험 결과는 우리 방법의 효과성을 검증한다.
