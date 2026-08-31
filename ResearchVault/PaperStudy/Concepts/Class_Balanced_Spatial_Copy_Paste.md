---
title: "Class-Balanced Spatial Copy-Paste"
tags: [concept, long-tailed-detection, data-augmentation, remote-sensing]
created: 2026-08-24
updated: 2026-08-24
---

# 정의
데이터셋 전체를 분석해 샘플 비율이 낮은 tail class를 식별한 뒤, 그 클래스의 개별 인스턴스를 원본 위치 주변(좌우 인접, 박스 너비의 일정 비율만큼 떨어진 거리)에 조건부로 복제해 클래스 불균형을 완화하는 augmentation 기법. 무작위 위치에 복제하는 random copy-paste와 달리, 복제된 인스턴스가 원본과 동일한 공간적 문맥(도로 위 차량 근처엔 차량만 복제 등)을 상속해 semantic 모순(예: 자동차를 바다 위에 복제)을 피한다.

# 등장 논문
- [[YOFOR]] — 원조. Class Balance Module(CBM)로 이 개념을 제안. VisDrone에서 7% 미만 비율의 클래스를 tail class로 식별, tail class 박스 너비의 1/4 거리로 좌우 조건부 복제. Random copy-paste 대비 일관된 우위를 실험으로 확인(Table 10).

# 변형/발전
시간 순 정리(등장 논문이 늘어날 때마다 갱신):
- 2026: YOFOR — 좌우 방향으로만 복제, tail class 판정은 데이터셋별 수동 분석(고정 임계값 7%) 기반.

# 관련 개념
- (아직 없음 — 이 위키에서 long-tailed detection을 다룬 첫 사례)
