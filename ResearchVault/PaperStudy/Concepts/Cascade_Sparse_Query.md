---
title: "Cascade Sparse Query (CSQ)"
tags: [concept, small-object-detection, sparse-convolution, inference-acceleration, feature-pyramid]
created: 2026-08-05
updated: 2026-08-05
---

# 정의
Feature pyramid 기반 detector에서, 저해상도 feature map의 예측으로 고해상도 feature map에서 소형 객체가 있을 만한 대략적 위치(query)를 먼저 찾고, 그 위치 주변에만 sparse convolution으로 detection head를 적용해 dense 연산을 피하는 coarse-to-fine 가속 기법. 한 레벨에서 다음 레벨로 query 위치를 직접 매핑하지 않고, 인접한 레벨끼리 순차적으로(cascade) 위치를 좁혀가며 넘긴다는 점이 핵심 — 단일 레벨에서 여러 단계 아래로 한번에 매핑하면 레벨 차가 커질수록 후보 위치 수가 지수적으로 폭증하는데, cascade 구조는 이를 막는다. 별도의 학습 가능한 gating network나 sparsity loss 없이, GT 박스로부터 유도한 이진 target map만으로 query head를 학습시킨다는 점이 다른 sparse 연산 기법과의 차별점이다.

# 등장 논문
- [[QueryDet]] — 이 기법의 원조 제안. RetinaNet/FCOS/Faster R-CNN에 적용해 고해상도 P2/P3 연산 비용을 약 74%에서 1%로 줄이면서 정확도 손실 없이 COCO 3.0배, VisDrone 2.3배 추론 속도 향상.

# 변형/발전
아직 원조 논문 1편에서만 등장. 향후 다른 논문이 이 기법을 채택하거나 변형하면 여기에 추가.

# 관련 개념
(아직 없음)
