---
title: "Instance Segmentation MOC"
tags: [moc]
task: instance-segmentation
created: 2026-08-19
updated: 2026-08-19
---

# 이 분야가 다루는 핵심 질문
- Instance segmentation은 픽셀 단위 마스크까지 요구해 detection보다 backbone의 정보 손실에 더 민감하다 — 소형 객체의 content 압축과 복잡구조 객체의 edge 단편화를 어떻게 동시에 보완할 것인가?
- 저해상도·저대비 센서 도메인(적외선 등)에서, 별도 라벨 없이 self-supervised 신호만으로 이런 정보 손실을 진단하고 feature를 보강할 수 있는가?

# 지금까지 다룬 흐름
- [[Reconstruction_Error_Guided_Instance_Segmentation]] — 이 task의 첫 논문. Backbone feature로부터 원본 이미지를 재구성(ORD)하고, 그 재구성 오차(difference map)를 전역 cross-attention(DFE)으로 backbone feature에 재주입해 소형·복잡구조 객체를 동시에 강화하는 model-agnostic 프레임워크. Small_Object_Detection의 [[SR-TOD]]가 제안한 [[Self_Reconstruction_Difference_Map]] 원리를 object detection에서 instance segmentation으로, 가시광/드론 영상에서 적외선 영상으로 확장한 사례라는 점에서 두 task 사이의 중요한 다리 역할을 한다.

# 이 분야를 관통하는 개념
- [[Self_Reconstruction_Difference_Map]] — reconstruction-error-guided-instance-segmentation의 핵심 원리가 기반하는 개념(원조는 Small_Object_Detection의 SR-TOD). "재구성이 어려운 영역=정보가 손실된 영역"이라는 self-supervised 진단 신호를 이 task에서는 multi-level decoder + 전역 cross-attention 형태로 확장해 사용한다.

# 비교 문서
(아직 없음 — 이 task에 논문이 더 쌓이면 작성)

# 아직 못 채운 빈틈
- 이 task는 현재 1편만 있어 흐름이라 부를 만한 것이 아직 없다. 적외선이 아닌 일반 도메인의 instance segmentation, 또는 Mask R-CNN/SOLO 계열 자체의 foundational 논문이 아직 위키에 없다.
- Small_Object_Detection task와의 경계가 모호할 수 있다 — 소형 객체를 다루는 instance segmentation 논문이 더 들어오면, 두 task 중 어디에 배정할지 기준(태스크가 detection이냐 segmentation이냐)을 계속 일관되게 적용해야 한다.

# 관련 MOC
- [[000-Home]]
- [[Small_Object_Detection_Moc]] — [[Self_Reconstruction_Difference_Map]] 개념을 공유하는 인접 분야. 소형 객체 정보 손실이라는 문제의식이 겹친다.
