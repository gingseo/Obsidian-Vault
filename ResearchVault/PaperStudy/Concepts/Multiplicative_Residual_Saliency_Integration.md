---
title: "Multiplicative Residual Saliency Integration"
tags: [concept, feature-fusion, salient-object-detection, residual-connection]
created: 2026-08-24
updated: 2026-08-24
---

# 정의
서로 다른 레벨(얕은/중간/깊은)의 feature를 element-wise 곱으로 결합해 공통으로 나타나는 saliency 정보만 추출(비공통 정보·노이즈는 곱셈 과정에서 자연히 억제)한 뒤, 이 공통 정보를 각 레벨의 원본 feature에 residual로 다시 더해 레벨별 고유 디테일 정보를 보존하는 다중 레벨 feature 통합 기법.

# 등장 논문
- [[AIMRINet]] — 원조. MRFI(Multiple Residual Feature Integration) 모듈로 이 개념을 제안. 세 레벨(X/Y/Z) 곱으로 공통 saliency `F_m`을 얻고, 각 레벨에 residual로 더한 뒤 concat+최종 residual로 마무리. Ablation에서 이 모듈의 단독 기여가 attention 기반 SAI 모듈보다 큼을 확인.

# 변형/발전
시간 순 정리(등장 논문이 늘어날 때마다 갱신):
- 2026: AIMRINet — 곱셈(공통점 추출)과 다중 residual(고유 정보 보존)을 명시적으로 분리한 설계. 중간 residual block 제거, 전체 residual 제거 각각의 ablation으로 두 구성요소 모두의 기여를 정량 검증.

# 관련 개념
- (아직 없음 — 이 위키에서 원격탐사 salient object detection을 다룬 첫 사례. 소형 객체 탐지 계열의 "다중 소스 동적 가중합" 패턴(예: ORFENet의 MRFAFEM)과 상위 목표는 유사하나 결합 연산(학습 가중합 vs 곱셈+residual)이 다름)
