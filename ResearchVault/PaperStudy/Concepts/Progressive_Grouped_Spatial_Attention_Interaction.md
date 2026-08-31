---
title: "Progressive Grouped Spatial Attention Interaction"
tags: [concept, attention-mechanism, salient-object-detection, feature-fusion]
created: 2026-08-24
updated: 2026-08-24
---

# 정의
Feature map을 channel shuffle로 채널을 뒤섞은 뒤 여러 그룹으로 균등 분할하고, 각 그룹에 spatial attention을 순차적으로 적용하되 이전 그룹의 강화 결과를 다음 그룹 처리에 더해가며 정보를 점진적으로 전파시키는 기법. 전체 feature에 한 번에 attention을 적용하는 대신 그룹별로 순차 처리해, 네트워크가 서로 다른 부분공간(subspace)의 상보적 feature를 학습하도록 유도한다.

# 등장 논문
- [[AIMRINet]] — 원조. SAI(Spatial Attention Interaction) 모듈로 backbone의 가장 얕은/깊은 feature에 4분할 순차 spatial attention을 적용. Ablation에서 4분할이 2/8분할보다 우수함을 확인(분할이 과도하면 그룹당 채널 수 부족으로 표현력 저하).

# 변형/발전
시간 순 정리(등장 논문이 늘어날 때마다 갱신):
- 2026: AIMRINet — channel shuffle + 4그룹 순차 spatial attention. Channel attention으로 대체 시 성능 저하를 확인해 spatial attention의 상대적 중요성을 실증.

# 관련 개념
- (아직 없음 — 이 위키에서 원격탐사 salient object detection을 다룬 첫 사례)
