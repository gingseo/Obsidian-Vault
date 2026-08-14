---
title: "Unc-SOD: An Uncertainty Learning Framework for Small Object Detection (원문 요약)"
tags: [paper-source]
created: 2026-08-04
---

[[Unc-SOD]]의 원문 요약. 분석·평가는 담지 않는다 — 순수 번역/발췌.

# Abstract
작은 물체 탐지(SOD)는 크기가 제한된 인스턴스에 내재된 정보 영역이 협소하다는 점에서 비롯되는, 주목할 만하지만 대단히 다루기 어려운 과제다. 이는 현재의 2단계 탐지기가 감당하는 수준을 넘어서는 심화된 불확실성을 촉발한다. 구체적으로, 작은 물체에 내재된 모호성은 기존의 지배적인 sampling 방식들을 훼손하고, 모델이 알아볼 수 없는 대상에 헛되이 노력을 쏟도록 오도할 수 있으며, 두 단계에서 탐지에 활용되는 feature들 간의 불일치는 계층적(hierarchical) 불확실성을 추가로 드러낸다.

본 논문에서 우리는 작은 물체 탐지를 위한 불확실성 학습 프레임워크, Unc-SOD를 개발한다. 기존 Region Proposal Network(RPN)에 보조 불확실성 브랜치를 통합함으로써, 우리는 인스턴스 수준의 불확정성을 모델링하며, 이는 이후 sampling을 위한 대리 기준(surrogate criterion)으로 작용해 다양한 정도의 불확실성에 기반해 동적으로 충분한 후보들을 발굴하고 proposal network의 학습을 촉진한다. 이와 병행하여, 풍부하고 판별력 있는 표현을 포착하기 위해 Perception-and-Interaction 전략을 고안했으며, 이는 원래 pyramid와 할당된 pyramid에서의 영역 feature들의 본질적 속성을 최적화함으로써 이루어지고, 이때 지각(perceptual) 과정은 상호(mutual) 패러다임으로 전개된다.

SOD 과제에서 불확실성을 모델링한 최초의 시도로서, 우리의 Unc-SOD는 두 개의 대규모 작은 물체 탐지 벤치마크인 SODA-D와 SODA-A에서 state-of-the-art 성능을 달성하며, COCO, VisDrone, Tsinghua-Tencent 100K를 포함한 여러 SOD 지향 데이터셋에서의 결과 역시 baseline detector 대비 향상을 보인다. 이는 우리 접근법의 효능과, 작은 인스턴스를 다룰 때 기존 탐지기들에 대한 우수성을 뒷받침한다.

# Introduction

#### SOD의 역할과 현재 우위 접근
작은 물체 탐지(SOD)는 일반 물체 탐지의 하위 과제로, 자율주행·의료 영상 분석·비전 기반 감시 시스템 등에서 핵심적인 역할을 한다. 최근 일반 물체 탐지는 DETR 계열이 주도하지만, 2단계 CNN 기반 패러다임이 SOD에서는 여전히 우위를 보인다.

#### 기존 연구 한계 지적 — sampling의 데이터 수준 문제
RPN 기반 prior-to-proposal 파이프라인은 작은 인스턴스에서 예상치 못한 장애물에 부딪힌다. Fig. 1에서 회귀 성능이 5번째 epoch부터 포화되는데, 이는 부분 가림·모션 블러 등으로 인한 데이터 수준의 내재적 불확실성 때문이다.

#### 기존 연구 한계 지적 — 획일적 sampling 기준
선행 연구는 박스를 Gaussian 분포로 모델링해 회귀 분산을 다뤘지만, 심하게 변형되거나 극단적으로 작은 물체는 다루지 못한다. 게다가 기존 proposal 패러다임은 고정 IoU 기준(≥0.7)을 모든 물체에 동일 적용해, 입력 IoU 0.8 이상인 prior조차 학습이 진행될수록 타겟에서 멀어지는 역설이 발생한다(Fig. 1(a)).

#### 기존 연구 한계 지적 — 계층 구조 문제
2단계 탐지기는 크기 의존적 매핑으로 pyramid 레벨을 지정하는데, SOD에서는 영역 표현이 최하위 레벨(P2)에서만 나와 다른 레벨은 암묵적으로만 갱신된다.

#### 이 논문이 다루는 문제 — 계층 간 불일치
proposal은 여러 pyramid의 prior에서 진화하는데, Faster RCNN proposal의 95% 이상이 P3~P5 같은 상위 레벨에서 비롯된다(Fig. 3). 이 불일치가 "계층 수준의 불확실성" 문제다.

#### 제안 방법 개요
Unc-SOD는 RPN을 보조 예측 브랜치로 확장해 인스턴스 수준 위치 불확실성을 모델링하고, 이를 후보 확보의 대리 기준으로 삼는다(Fig. 4). 계층 불확실성에는 원래 레벨과 할당된 레벨 간 상관관계를 활용하는 Perception-and-Interaction 전략(analytic perception + holistic perception + cross-interaction)으로 대응한다.

# Main Contribution
저자들이 원문에서 명시한 기여는 다음 세 가지다.
1. 표본 부족을 완화하는 Uncertainty-aware Sampling 전략. RPN에 새 브랜치를 설치해 GT 인스턴스의 불확실성을 모델링하고, 이를 후보 확보의 대리 기준으로 삼아 충분한 proposal을 만들면서 알아볼 수 없는 인스턴스의 오도 위험을 최소화한다.
2. 원래 레벨과 할당된 레벨 간 상관관계를 활용해 판별력 있는 표현을 얻는 Perception-and-Interaction 방식.
3. SODA 벤치마크에서 state-of-the-art 달성, COCO/VisDrone/TT100K에서도 일반성과 강건성 확인.

# Conclusion
본 논문에서 우리는 작은 물체 탐지를 위한 불확실성 학습 프레임워크, Unc-SOD를 제시했다. 먼저 우리는 각 인스턴스의 불확정성을 확보하는 Uncertainty-aware Sampling 전략을 개발했으며, 이는 이후 작은 물체에 대한 잠재적 표본을 발굴하는 대리 기준으로 작용해 표본 부족을 완화하고 고도로 모호한 인스턴스의 저품질 prior에 의해 최적화가 오염되는 것을 방지한다. 다음으로, 계층 수준의 불확실성에서 비롯되는 두 단계 간 feature 불일치를 다루기 위해 기존에 활용 가능한 단서를 활용하는 Perception-and-Interaction 파이프라인을 고안했다. 구체적으로 우리는 상호 Perception 방식을 구성한 뒤 cross Interaction을 이어 붙여, 원래 레벨과 할당된 pyramid의 영역 feature에 내재된 본질적 상관관계를 활용함으로써 크기 제한 인스턴스에 내재된 정보 부족을 보완하는 동시에 판별력 있는 표현을 도출했다.

대규모 작은 물체 탐지 벤치마크인 SODA-D와 SODA-A, 그리고 COCO·VisDrone·Tsinghua-Tencent 100K를 포함한 여러 SOD 지향 데이터셋에서의 광범위한 실험은 SOD 과제에서 우리의 불확실성 모델링 패러다임의 우수성을 뒷받침한다. 나아가, 제안된 불확실성 학습 프레임워크는 현재 내재적인 데이터 수준(aleatoric) 불확실성에만 초점을 맞추고 있는데, 딥러닝 모델이 어려운 상황에 직면했을 때 높은 모델 수준(epistemic) 불확실성을 보일 수 있다는 것은 널리 알려져 있다. 따라서 유망한 향후 방향은, aleatoric uncertainty를 활용해 고품질 표본을 선별하는 동시에 epistemic uncertainty를 활용해 가치 있는 표본을 강조하거나 최종 탐지를 위한 신뢰도 보정 신호를 제공함으로써 과신(overconfident)된 false positive를 효과적으로 억제하는, 더 포괄적이고 통합된 불확실성 학습 파이프라인을 구축하는 것이다. 이는 불확실성에 대한 총체적 이해를 갖춘 강건한 작은 물체 탐지 패러다임에 기여할 것이며, 우리는 이 유망한 방향을 향후 연구로 남겨둔다.
