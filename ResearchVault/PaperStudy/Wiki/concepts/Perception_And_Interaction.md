---
title: "Perception-and-Interaction (P&I)"
tags: [concept, object-detection, feature-fusion, small-object-detection]
created: 2026-08-04
updated: 2026-08-04
---

# 정의
Two-stage detector에서 동일한 proposal에 대해 서로 다른 두 FPN 피라미드 레벨(① proposal이 유래한 anchor의 원래 레벨, ② 크기 기반 휴리스틱으로 할당된 레벨)에서 추출한 region feature를 상호 보완적으로 융합하는 기법. 두 feature는 상호(reciprocal) perception 단계(Analytic Perception + Holistic Perception)를 거쳐 서로의 약점을 보완한 뒤, self-attention 형태의 cross interaction으로 최종 판별력 있는 표현을 얻는다.

# 등장 논문
- [[Unc-SOD]] — 이 논문이 최초로 제안. Small object detection에서 두 스테이지가 서로 다른 pyramid level의 feature를 쓰는 "hierarchy-level uncertainty" 문제를 해결하기 위해 도입.

# 변형/발전
- Unc-SOD(2026)가 처음 제안한 구조. 아직 다른 논문에서의 재사용 사례는 확인되지 않음(2026년 발표).
- 구성 요소:
  - Analytic Perception: compact한 상위 레벨 feature(F_a)로 position-dependent kernel을 생성해, 상대적으로 구조 정보가 부족한 원래 레벨 feature(F_o)의 공간적 디테일을 복원.
  - Holistic Perception: F_o를 Global Average Pooling한 뒤 F_a와 결합해, F_a에 변형/노이즈에 대한 강건성을 부여.
  - Cross Interaction: self-attention과 유사하되 query/key/value를 서로 다른 두 perception 결과에서 가져오는 cross-level attention.

# 관련 개념
- [[Gaussian_Box_Uncertainty_Modeling]]
