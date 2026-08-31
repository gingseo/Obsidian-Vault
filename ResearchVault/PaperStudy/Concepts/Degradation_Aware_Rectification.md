---
title: "Degradation-aware Rectification (Degradation-then-Rectification Paradigm)"
tags: [concept, small-object-detection, restoration, degradation-modeling, mixture-of-experts, multi-task-learning]
created: 2026-08-04
updated: 2026-08-04
---

# 정의
Small object의 열화(degradation)를 먼저 명시적으로 학습한 뒤, 그 지식을 조건으로 삼아 탐지(detection) 지향적으로 feature를 교정(rectify)하는 2단계 패러다임. 핵심 원칙은 "무엇이 열화를 일으키는지 알아야, 어떻게 교정할지 안다(knowing what degrades, knowing how to rectify)"이다.

기존 restoration 기반 SOD 방법들이 좁은 범위의 synthetic corruption으로 restoration 모듈을 학습시켜 추론 시 distribution shift를 겪고, pixel-level fidelity(restoration 목표)와 semantic discriminability(detection 목표) 간 task conflict에 시달리는 문제를 해결하기 위해 고안되었다. 구체적으로 두 요소로 구성된다:

1. **Degradation-aware Learning**: 학습 가능한 degradation basis 집합(B_d)을 두고, 원본 feature에 이 basis를 modulation해 실제 열화 feature를 모사하도록 학습시킨다. 단, pixel 단위 정합이 아니라 prediction-level(classification KL divergence, box IoU) 정합을 핵심 supervision으로 삼아 task-relevant한 열화 패턴만 선택적으로 포착하고, InfoNCE 기반 representation constraint와 basis 직교 정규화로 basis의 collapse를 방지한다.
2. **Task-oriented Rectification**: 학습된 degradation basis를 Mixture-of-Experts(MoE) 스타일의 task-specific router가 참조해, 입력 위치별로 지배적인 열화 유형에 맞는 rectification prompt를 동적으로 합성한다. 이 prompt는 경량 cross-attention과 per-pixel modulation을 결합한 rectification block을 반복 통과하며 feature를 점진적으로 정제한다.

일반적인 restoration-then-detection 파이프라인과 달리, degradation을 "미리 알고 있는 지식"으로 명시화해 조건부 프롬프트를 만든다는 점, 그리고 rectification 자체가 pixel 복원이 아니라 detection-friendly한 feature 교정을 직접 목표로 한다는 점이 핵심 차별점이다.

# 등장 논문
- [[Detection_Oriented_Rectification]] — 이 개념을 최초로 제안. DORA(Detection-Oriented RectificAtion) 프레임워크로 구현하여 degradation basis 기반 학습, MoE 기반 task-oriented rectification, entity reconstruction, self-correction term을 결합해 다양한 detector에 plug-in 형태로 적용. SODA-D/SODA-A/AITOD-R/COCO/VisDrone 5개 벤치마크에서 일관된 성능 향상을 입증.

# 변형/발전
현재는 [[Detection_Oriented_Rectification]] 1편에서만 제안된 개념으로, 이후 변형·확장 사례는 아직 없다.

# 관련 개념
- (없음 — 현재 위키 내 관련 concept 미등록)
