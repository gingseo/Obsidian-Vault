---
title: "AI-Generated Image Detection MOC"
tags: [moc]
task: ai-generated-image-detection
created: 2026-08-05
updated: 2026-08-05
---

# 이 분야가 다루는 핵심 질문
- Diffusion model이 만든 이미지는 점점 더 실제 이미지와 구별하기 어려워지고 있다 — 실제/생성 이미지를 가르는 신뢰할 수 있는 판별 신호는 무엇인가?
- Diffusion 모델 특유의 "재구성(reconstruction)" 성질을 판별에 활용한다면, 그 재구성 과정을 완전히 수행하지 않고도 같은 신호를 저비용으로 얻을 수 있는가?
- 특정 생성기(GAN/특정 diffusion 아키텍처)로 학습한 탐지기가 본 적 없는 생성기의 이미지에도 일반화될 수 있는가?

# 지금까지 다룬 흐름
아직 1편만 읽어 흐름을 서술하기에는 이르다. 다음 논문이 들어오면 이 절을 채운다.
- [[lare2]] — Diffusion model의 forward process가 닫힌 형태 해를 갖는다는 성질을 이용해, 완전 재구성 없이 latent space에서 단일 스텝 디노이징만으로 재구성 오차(LaRE)를 얻고, 이를 공간·채널 attention으로 원본 feature에 결합해 판별력을 높이는 접근. DIRE(다단계 샘플링 기반)의 비효율을 정면으로 겨냥한 첫 시도.

# 이 분야를 관통하는 개념
- [[latent-reconstruction-error]] — lare2의 핵심 기여. Small-object-detection의 [[self-reconstruction-difference-map]]과 "재구성 난이도를 판별 신호로 쓴다"는 원리를 공유하는 타 도메인 사례.

# 비교 문서
(아직 없음 — 비교할 두 번째 논문이 들어오면 생성)

# 아직 못 채운 빈틈
- lare2가 직접 비교하는 baseline DIRE(다단계 DDIM 기반 reconstruction error) 논문 자체가 아직 위키에 없음 — 이 분야의 출발점이므로 우선순위가 높음.
- GAN 생성 이미지 탐지(CNNSpot, Spec, F3Net 등 lare2가 비교한 방법들)의 원 논문도 아직 없음 — diffusion 이전 시대의 접근법과 비교하려면 필요.
- 이 분야와 [[small-object-detection-moc]]의 reconstruction 기반 feature 강화 계열(sr-tod 등)이 원리적으로 겹친다는 점을 발견했으나, 실제로 두 도메인을 교차 비교하는 논문은 아직 없다.

# 관련 MOC
- [[000-home]]
- [[small-object-detection-moc]] — reconstruction 기반 신호 활용이라는 원리가 겹침
