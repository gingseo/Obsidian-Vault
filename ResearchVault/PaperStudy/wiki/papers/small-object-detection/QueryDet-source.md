---
title: "QueryDet: Cascaded Sparse Query for Accelerating High-Resolution Small Object Detection (원문 요약)"
tags: [paper-source]
created: 2026-08-05
---

[[QueryDet]]의 원문 요약. 분석·평가는 담지 않는다 — 순수 번역/발췌.

# Abstract
FPN 기반 CNN을 이용한 최신 객체 탐지 방법은 큰 발전을 이루었지만, 고해상도 입력에서 소형 객체(small object)를 탐지하는 것은 여전히 어려운 문제다. 이 간극은 고해상도 feature map에서의 값비싼 연산에서 비롯된다. 이 문제를 해결하기 위해 우리는 새로운 query 기반 detection framework인 QueryDet을 제안한다. 이는 cascaded sparse query 메커니즘을 통해 feature pyramid의 추론 속도를 가속화한다. 저해상도 feature에서의 예측을 사용해, 대응하는 고해상도 영역에서 소형 객체가 있을 것 같은 대략적인 위치를 빠르게 찾아낸다. 이런 식으로, dense하고 연산 비용이 큰 detection head를 고해상도 feature map 전체에 적용하는 대신, sparse하고 계산 효율적인 head만 예측된 소형 객체 영역에 적용할 수 있다. FPN 기반 모델을 사용해, 우리의 방법은 소형 객체 탐지의 정확도를 유지하거나 향상시키면서 COCO와 VisDrone 데이터셋에서 각각 1.8배와 3.0배의 추론 속도 향상을 달성한다. 특히 VisDrone에서, 우리의 방법은 mAP 33.9%로 최신 최고 성능을 달성하면서 기존 최고 성능 방법보다 5.5배 빠르다. 코드는 공개되어 있다.

# Introduction

#### 소형 객체 탐지의 기존 접근과 그 비용
소형 객체 탐지는 고해상도 feature map(넓은 receptive field, 미세한 정보 보존)에 의존해 왔지만, 이는 연산량과 메모리 소모를 크게 증가시킨다.

#### FPN 구조의 트레이드오프
FPN은 고/저해상도 feature를 병합해 다양한 스케일의 객체를 탐지하지만, 고해상도 레벨(P2 등)에 dense detection head를 적용하는 비용이 여전히 크다.

#### 기존 가속화 연구의 한계
Cascade RPN이나 sparse convolution 기반 가속 기법들은 후보 영역을 줄이려 하지만, 여전히 고해상도 feature map 자체를 dense하게 계산하거나 별도의 학습 복잡도를 필요로 한다.

#### 이 논문의 핵심 관찰
저해상도 feature map에서의 예측이 고해상도 feature map에서 소형 객체가 있을 만한 위치를 대략적으로 가리킬 수 있다는 관찰에서 출발한다.

#### 제안 방법 개요
저해상도 레벨의 예측으로부터 고해상도 레벨에서 조사할 sparse한 위치(query)를 생성하고, 그 위치에만 detection head를 sparse하게 적용하는 cascaded sparse query 메커니즘 QueryDet을 제안한다.

#### 실험 결과 요약
COCO와 VisDrone에서 정확도를 유지·향상하면서 각각 1.8배, 3.0배 추론 속도를 향상시켰고, VisDrone에서는 최고 성능 대비 5.5배 빠르면서 SOTA 정확도(33.9% mAP)를 달성했다.

#### 기여 요약
Cascaded sparse query 메커니즘, 이를 위한 sparse convolution 기반 효율적 구현, 그리고 두 벤치마크에서의 속도·정확도 개선을 기여로 제시한다.

# Main Contribution
1. 저해상도 feature map의 예측을 이용해 고해상도 feature map에서 조사할 sparse한 위치를 예측하는 cascaded sparse query 메커니즘을 제안한다.
2. 이 메커니즘을 효율적으로 구현하기 위해 sparse convolution을 활용한 detection head 설계를 제시한다.
3. COCO와 VisDrone 데이터셋에서 정확도를 유지·향상하면서 각각 1.8배, 3.0배의 추론 속도 향상을 달성했으며, VisDrone에서는 기존 최고 성능 방법 대비 5.5배 빠르면서 33.9% mAP로 새로운 SOTA를 달성했다.

# Conclusion
이 논문에서는 고해상도 입력에서의 소형 객체 탐지를 가속화하기 위한 새로운 query 기반 detection framework인 QueryDet을 제안했다. Cascaded sparse query 메커니즘은 저해상도 feature map의 예측을 이용해 고해상도 feature map에서 소형 객체가 있을 만한 위치를 찾아내고, 이 sparse한 위치에만 연산 비용이 큰 detection head를 적용함으로써 dense한 연산을 피한다. COCO와 VisDrone 데이터셋에서의 실험 결과는 이 방법이 정확도를 유지하거나 향상시키면서 추론 속도를 크게 개선함을 보여준다.
