---
title: "Edge-Controlled Anomaly Synthesis"
tags: [concept, anomaly-detection, data-synthesis, edge-detection, generative-model]
created: 2026-08-24
updated: 2026-08-24
---

# 정의
이미지 픽셀을 직접 조작하는 대신, edge map(밝기·색상·텍스처의 급격한 변화를 나타내는 object-agnostic 중간 표현)을 제거·교체·병합하는 방식으로 조작한 뒤, edge-to-image generator(conditional GAN)로 사실적인 이미지를 생성해 비지도 이상 탐지 학습용 합성 이상을 만드는 기법. Semantic 영역(SAM 기반)에서 edge를 제거하면 부품 누락 같은 논리적 이상이, 임의 영역에서 다른 이미지의 edge로 교체하면 구조적 결함이 만들어진다.

# 등장 논문
- [[LogicAL]] — 원조. PiDiNet으로 edge 추출, pix2pixHD 기반 edge-to-image generator를 DeepSIM의 TPS warping으로 증강해 학습. Remove/replace/merge 세 조작으로 논리·구조·혼합 이상을 통일된 프레임워크에서 생성.

# 변형/발전
시간 순 정리(등장 논문이 늘어날 때마다 갱신):
- 2024: LogicAL — SAM 기반 semantic 영역 선택+임의 영역 선택을 결합, edge reconstruction을 탐지 네트워크의 auxiliary loss로도 추가 활용.

# 관련 개념
- (아직 없음 — 이 위키에서 anomaly synthesis를 다룬 첫 사례)
