---
title: "Latent Restoration Regularization"
tags: [concept, super-resolution, remote-sensing, auxiliary-task, latent-space]
created: 2026-08-24
updated: 2026-08-24
---

# 정의
Super-resolution(SR)을 추론 전 이미지/영역/feature 명시적 향상 단계("restoration-first")로 쓰는 대신, 학습 시에만 존재하는 SR 복원 브랜치로 detection과 파라미터를 공유하는 encoder를 정규화하는 데만 사용하는 프레임워크. 추론 시 SR 브랜치는 완전히 제거되며, detection은 이 정규화로 개선된 latent representation에서 직접 수행된다 — "복원이 탐지를 보조해야지 지배해서는 안 된다"는 원칙을 설계에 내재화한다.

# 등장 논문
- [[CoLR-Det]] — 원조. 학습 전용 latent restoration branch(U-Net+Swin decoder)로 encoder를 정규화하고, saliency-guided object-preserving token routing으로 배경 텍스처 오염을 막는다. Detached 변형(encoder gradient 차단)이 baseline과 거의 동일한 성능을 보여, "encoder를 함께 업데이트한다"는 것 자체가 핵심 메커니즘임을 실증.

# 변형/발전
시간 순 정리(등장 논문이 늘어날 때마다 갱신):
- 2026: CoLR-Det — SR reconstruction loss(L1)로 encoder를 정규화, detection-prioritized 2단계 학습(semantic stabilization → collaborative refinement)과 saliency 기반 non-destructive token routing을 결합.

# 관련 개념
- [[Self_Reconstruction_Difference_Map]] — SR-TOD가 재구성 오차 자체를 attention prior로 쓰는 것과 달리, 이 개념은 재구성 오차를 직접 활용하지 않고 "재구성을 학습 목표로 삼는다"는 사실 자체가 encoder를 정규화하는 데만 쓰인다는 점에서 다르다. [[ORFENet]]의 ORB, [[FFSSTDNet]]의 FSR과 같은 "학습시에만 존재하는 auxiliary branch" 계보의 네 번째 변형으로 볼 수 있다.
