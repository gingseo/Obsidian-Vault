---
title: "Anomaly Detection MOC"
tags: [moc]
task: anomaly-detection
created: 2026-08-04
updated: 2026-08-24
---

# 이 분야가 다루는 핵심 질문
- 비지도 이상 탐지(UAD)의 reconstruction 기반 접근은 흔히 ImageNet 사전학습 encoder를 고정(frozen)해서 쓴다 — 이걸 풀어서(trainable) 함께 학습시키면서도 pattern collapse(encoder와 decoder가 서로 같아져 이상까지 잘 복원해버리는 문제)를 피할 수 있는가?
- 실제 이상 샘플이 희소한 상황에서, 탐지기 학습에 쓸 합성 이상을 어떻게 사실적이면서도 다양하게(특히 구조적 결함뿐 아니라 논리적 제약 위반까지) 만들어낼 것인가?

# 지금까지 다룬 흐름
2편을 읽었으며, 서로 다른 개입 지점(탐지 아키텍처 자체 vs 학습 데이터 합성)을 대표한다.
- [[ReContrast]] — contrastive learning의 dual-encoder 구조를 도입해 encoder를 end-to-end로 학습 가능하게 만든 논문. UAD의 "encoder는 고정해야 한다"는 통념 자체를 문제 삼는 접근.
- [[LogicAL]] — 탐지 아키텍처가 아니라 학습용 합성 이상 자체를 개선하는 접근. 이미지 픽셀 대신 edge map을 중간 표현으로 조작(제거·교체·병합)해 사실적이면서 논리적 제약을 위반하는 이상(부품 누락·개수 초과 등)까지 합성. 기존 방법들이 구조적 결함(scratch, dent)에 편향되어 있다는 점을 정면으로 지적한 첫 사례.

두 논문은 "어떻게 탐지할 것인가"(ReContrast)와 "무엇으로 학습시킬 것인가"(LogicAL)라는 서로 다른 질문에 답한다는 점에서 상호보완적 — 실제로 결합 가능성이 있다(LogicAL의 합성 이상으로 ReContrast류 아키텍처를 학습시키는 방향).

# 이 분야를 관통하는 개념
- [[ReContrast_Dual_Encoder_Contrastive_Reconstruction]] — recontrast의 핵심 기여.
- [[Edge_Controlled_Anomaly_Synthesis]] — LogicAL의 핵심 기여. Edge 조작을 통해 논리적·구조적 이상을 통일된 프레임워크로 합성.

# 비교 문서
(아직 없음 — 두 논문이 서로 다른 개입 지점(아키텍처 vs 데이터 합성)을 다뤄 직접 비교축이 명확하지 않다. 세 번째 논문이 추가되면 재검토)

# 아직 못 채운 빈틈
- 이 task의 대표 baseline(예: RD4AD 등 recontrast가 참조하는 선행 reconstruction 기반 UAD 논문, Draem·GCAD·SLSG 등 LogicAL이 반복 비교하는 논문들)이 아직 위키에 없어, 두 논문이 실제로 무엇을 개선했는지 비교할 대상이 부족하다.
- LogicAL의 합성 이상 생성 방식(edge 조작)과 ReContrast의 dual-encoder 아키텍처를 실제로 결합한 사례가 아직 이 위키에 없다 — MOC 상에서만 결합 가능성을 언급한 상태.

# 관련 MOC
- [[000-Home]]
