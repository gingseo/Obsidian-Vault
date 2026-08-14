---
title: "End-to-End Object Detection with Transformers (원문 요약)"
tags: [paper-source]
created: 2026-08-12
---

[[DETR]]의 원문 요약. 분석·평가는 담지 않는다 — 순수 번역/발췌.

# Abstract
우리는 객체 탐지를 직접적인 집합 예측(direct set prediction) 문제로 보는 새로운 방법을 제시한다. 우리의 접근은 탐지 파이프라인을 간소화해, 이 과제에 대한 사전 지식을 명시적으로 인코딩하는 비최대 억제(non-maximum suppression) 절차나 anchor 생성 같은 다수의 수작업 설계 구성요소의 필요성을 효과적으로 제거한다. DEtection TRansformer, 즉 DETR이라 불리는 새 프레임워크의 핵심 요소는 이분 매칭(bipartite matching)을 통해 고유한 예측을 강제하는 집합 기반 전역 손실(set-based global loss)과 transformer encoder-decoder 아키텍처다. 학습된 소수의 고정된 object query 집합이 주어지면, DETR은 객체 간의 관계와 전역 이미지 컨텍스트에 대해 추론해 최종 예측 집합을 병렬로 직접 출력한다. 새 모델은 개념적으로 단순하며, 다른 많은 최신 탐지기들과 달리 특화된 라이브러리를 필요로 하지 않는다. DETR은 어려운 COCO 객체 탐지 데이터셋에서 잘 확립되고 고도로 최적화된 Faster R-CNN baseline과 대등한 정확도·실행시간 성능을 보인다. 나아가 DETR은 통합된 방식으로 panoptic segmentation을 생성하도록 쉽게 일반화될 수 있다. 우리는 이것이 경쟁력 있는 baseline들을 크게 능가함을 보인다. 학습 코드와 사전학습 모델은 공개되어 있다.

# Introduction

#### 기존 객체 탐지 파이프라인의 간접성
최신 탐지기는 집합 예측 과제를 대규모 proposal, anchor, window center에 대한 대리(surrogate) 회귀·분류 문제로 간접적으로 다루며, 후처리·anchor 설계·타겟 할당 휴리스틱에 성능이 크게 좌우된다.

#### End-to-end 철학의 미적용 영역
End-to-end 철학은 기계 번역이나 음성 인식 같은 복잡한 구조적 예측 과제에서 큰 발전을 이끌었지만, 객체 탐지에서는 아직 그렇지 못했다 — 기존 시도들은 추가적인 사전 지식을 필요로 하거나 강력한 baseline과 경쟁력이 입증되지 않았다.

#### 제안 방법 개요
탐지를 직접적인 집합 예측 문제로 재정의해, transformer encoder-decoder 아키텍처와 이분 매칭 기반 집합 손실 함수로 end-to-end 학습되는 DETR을 제안한다.

#### 핵심 메커니즘
DETR은 모든 객체를 한 번에 예측하며, 예측과 GT 객체 간 이분 매칭을 수행하는 집합 손실 함수로 end-to-end 학습된다. Anchor나 non-maximal suppression 같은 사전 지식을 인코딩하는 수작업 구성요소를 제거해 탐지 파이프라인을 단순화한다.

#### 실험 결과 요약
COCO에서 매우 경쟁력 있는 Faster R-CNN baseline과 대등한 성능을 달성했으며, 특히 대형 객체에서 유의미하게 더 나은 성능을 보이지만 소형 객체에서는 더 낮은 성능을 보인다.

#### 학습 특성
DETR은 표준 탐지기와 여러 면에서 다른 학습 설정을 요구한다 — 매우 긴 학습 스케줄이 필요하며 transformer의 보조 디코딩 손실(auxiliary decoding loss)로부터 이득을 얻는다.

#### 기여 요약
탐지를 직접 집합 예측으로 보는 새로운 접근, 이분 매칭 손실과 병렬 디코딩 transformer의 결합, panoptic segmentation으로의 손쉬운 확장성을 기여로 제시한다.

# Main Contribution
1. 객체 탐지를 직접적인 집합 예측 문제로 취급해, non-maximum suppression이나 anchor 생성 같은 다수의 수작업 설계 구성요소를 제거하는 새로운 프레임워크 DETR을 제안한다.
2. 이분 매칭을 통해 고유한 예측을 강제하는 집합 기반 전역 손실과, 병렬 디코딩이 가능한 transformer encoder-decoder 아키텍처를 핵심 요소로 결합했다.
3. COCO에서 고도로 최적화된 Faster R-CNN baseline과 대등한 정확도·실행시간 성능을 보이며, 특히 대형 객체에서 유의미하게 더 나은 성능을 달성한다.
4. 사전학습된 DETR 위에 단순한 segmentation head를 학습시켜 panoptic segmentation으로 자연스럽게 확장할 수 있으며, 경쟁력 있는 baseline들을 유의미하게 능가한다.

# Conclusion
우리는 transformer와 이분 매칭 손실에 기반한 직접적인 집합 예측을 위한 객체 탐지 시스템의 새로운 설계인 DETR을 제시했다. 이 접근은 어려운 COCO 데이터셋에서 최적화된 Faster R-CNN baseline과 대등한 결과를 달성한다. DETR은 구현이 단순하며, panoptic segmentation으로 쉽게 확장 가능한 유연한 아키텍처를 갖는다. 또한 self-attention이 수행하는 전역 정보 처리 덕분으로 보이는, Faster R-CNN보다 유의미하게 나은 대형 객체 성능을 달성한다. 이 새로운 탐지기 설계는 특히 학습, 최적화, 소형 객체 성능과 관련해 새로운 과제를 동반한다. 기존 탐지기들은 유사한 문제를 다루기 위해 수년간의 개선이 필요했으며, 우리는 향후 연구가 DETR에 대해서도 이를 성공적으로 다룰 것으로 기대한다.
