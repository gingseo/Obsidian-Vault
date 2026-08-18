---
title: "Salient Object Detection MOC"
tags: [moc]
task: salient-object-detection
created: 2026-08-04
updated: 2026-08-04
---

# 이 분야가 다루는 핵심 질문
- Salient object detection에서 예측이 애매한(저채도/경계 불명확) 영역을 어떻게 식별하고, 그 영역만 반복적으로 정제(refine)할 것인가?

# 지금까지 다룬 흐름
아직 1편만 읽어서 "흐름"을 서술하기엔 이르다. 다음 논문이 추가되면 이 섹션을 갱신한다.
- [[Uncertainty_Guided_Refinement]] — 별도의 경계 라벨/가이던스 없이, 예측 saliency map 자체에서 불확실성 맵을 만들어 attention을 마스킹하는 방식으로 반복 정제.

# 이 분야를 관통하는 개념
- [[Uncertainty_Masked_Refinement_Attention]] — uncertainty-guided-refinement의 핵심 기여. Small object detection 쪽의 [[Gaussian_Box_Uncertainty_Modeling]]과 이름은 비슷하지만, 학습된 분포 파라미터가 아니라 예측값 자체에서 결정론적으로 유도되는 맵이라는 점에서 메커니즘이 다르다(각 개념 문서에 상호 링크 있음).

# 비교 문서
(아직 없음)

# 아직 못 채운 빈틈
- 이 task 역시 단일 논문 상태라 계열 비교가 아직 불가능.

# 관련 MOC
- [[000-Home]]
