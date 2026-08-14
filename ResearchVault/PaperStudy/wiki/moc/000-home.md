---
title: "연구 지도"
tags: [moc]
created: 2026-08-04
updated: 2026-08-12
---

# 연구 지도

## Task별 MOC
- [[small-object-detection-moc]] — 13편, 가장 많이 다룬 분야
- [[anomaly-detection-moc]] — 1편
- [[salient-object-detection-moc]] — 1편
- [[ai-generated-image-detection-moc]] — 1편

## 지금 무엇에 집중하고 있는가
지금까지 읽은 15편 중 13편이 small/tiny object detection에 집중되어 있고, 그중 12편은 "기존 detector에 어떤 plug-in 모듈을 추가할 것인가"(feature 강화, label assignment, 연산 가속, 경량화, 구조 개선)를 다루며, 나머지 1편([[DETR]])은 이 위키에서 처음 다루는 순수 foundational 아키텍처 논문이다. 자세한 흐름과 빈틈은 [[small-object-detection-moc]] 참고.

Anomaly detection, salient object detection, ai-generated-image-detection은 각각 1편씩만 읽은 상태로, 아직 이 분야의 전체 지형을 파악하기엔 이르다. 다만 ai-generated-image-detection의 [[lare2]]가 쓰는 reconstruction error 기반 판별 신호는 small-object-detection의 [[self-reconstruction-difference-map]]과 원리적으로 겹쳐, 두 분야를 넘나드는 교차 관점이 있다.
