---
title: "Anomaly Detection MOC"
tags: [moc]
task: anomaly-detection
created: 2026-08-04
updated: 2026-08-04
---

# 이 분야가 다루는 핵심 질문
- 비지도 이상 탐지(UAD)의 reconstruction 기반 접근은 흔히 ImageNet 사전학습 encoder를 고정(frozen)해서 쓴다 — 이걸 풀어서(trainable) 함께 학습시키면서도 pattern collapse(encoder와 decoder가 서로 같아져 이상까지 잘 복원해버리는 문제)를 피할 수 있는가?

# 지금까지 다룬 흐름
아직 1편만 읽어서 "흐름"을 서술하기엔 이르다. 다음 논문이 추가되면 이 섹션을 갱신한다.
- [[ReContrast]] — contrastive learning의 dual-encoder 구조를 도입해 encoder를 end-to-end로 학습 가능하게 만든 논문. UAD의 "encoder는 고정해야 한다"는 통념 자체를 문제 삼는 접근.

# 이 분야를 관통하는 개념
- [[ReContrast_Dual_Encoder_Contrastive_Reconstruction]] — recontrast의 핵심 기여.

# 비교 문서
(아직 없음 — 비교할 다른 논문이 추가되면 작성)

# 아직 못 채운 빈틈
- 이 task의 대표 baseline(예: RD4AD 등 recontrast가 참조하는 선행 reconstruction 기반 UAD 논문)이 아직 위키에 없어, recontrast가 실제로 무엇을 개선했는지 비교할 대상이 부족하다.
- Small object detection 쪽과 달리 이 task는 아직 단일 논문이라 계열 분류 자체가 불가능한 상태.

# 관련 MOC
- [[000-Home]]
