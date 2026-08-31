---
title: "연구 지도"
tags: [moc]
created: 2026-08-04
updated: 2026-08-24
---

# 연구 지도

## Task별 MOC
- [[Small_Object_Detection_Moc]] — 26편, 가장 많이 다룬 분야
- [[Anomaly_Detection_Moc]] — 2편
- [[Salient_Object_Detection_Moc]] — 2편
- [[AI_Generated_Image_Detection_Moc]] — 1편
- [[Instance_Segmentation_Moc]] — 1편
- [[Visual_Grounding_Moc]] — 1편, 새로 생긴 분야

## 지금 무엇에 집중하고 있는가
지금까지 읽은 28편 중 대다수가 small/tiny object detection에 집중되어 있다. 그중 다수는 "기존 detector에 어떤 plug-in 모듈을 추가할 것인가"(feature 강화, label assignment, 연산 가속, 경량화, 구조 개선)를 다루고, 6편은 Deformable DETR을 baseline으로 삼아 object query의 개수·구성을 이미지 내용에 따라 동적으로 조정하는 "dynamic query DETR" 계열([[DQ-DETR]], [[Density-Aware-DETR]], [[IG-DETR]], [[PaQ-DETR]], [[DQA-DETR]], [[DQP-DETR]])이며, 나머지 3편([[DETR]], [[Deformable_Convolutional_Networks]], [[Deformable-DETR]])은 이 흐름들이 기반하는 순수 foundational 아키텍처 논문이다. 자세한 흐름과 빈틈은 [[Small_Object_Detection_Moc]] 참고.

Salient object detection은 [[AIMRINet]](원격탐사, 다중 레벨 feature 통합)이 추가되어 2편이 되었고, 자연 이미지 반복 정제([[Uncertainty_Guided_Refinement]])와 서로 다른 접근을 대표한다. Anomaly detection도 [[LogicAL]](edge 조작 기반 논리적/구조적 이상 합성)이 추가되어 2편이 되었으며, [[ReContrast]](탐지 아키텍처 개선)와 "어떻게 탐지할지" vs "무엇으로 학습시킬지"라는 상호보완적 축을 형성한다. Visual grounding은 [[VGRSS]]로 새로 열린 분야로, 원격탐사 선박 영상에서 자연어로 특정 객체를 지목하는 과제를 다룬다 — "모든 객체를 찾는다"는 small-object-detection의 완전성 문제와 달리 "특정 객체 하나만 정확히 찾는다"는 특정성 문제라는 점에서 이 위키에 새로운 축을 더한다. Ai-generated-image-detection, instance-segmentation은 각각 1편씩만 읽은 상태로, 아직 전체 지형을 파악하기엔 이르다. 다만 ai-generated-image-detection의 [[LaRE2]]가 쓰는 reconstruction error 기반 판별 신호는 small-object-detection의 [[Self_Reconstruction_Difference_Map]]과 원리적으로 겹치고, instance-segmentation의 [[Reconstruction_Error_Guided_Instance_Segmentation]]은 이 원리를 직접 detection에서 segmentation으로, 가시광에서 적외선 도메인으로 확장한 사례라 세 분야를 넘나드는 교차 관점이 뚜렷해지고 있다.
