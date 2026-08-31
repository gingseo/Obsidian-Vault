---
title: "Deformable Sampling Offset"
tags: [concept, cnn, geometric-transformation, attention]
created: 2026-08-24
updated: 2026-08-24
---

# 정의
Convolution이나 attention의 고정된 샘플링 위치(grid)에, 입력 feature로부터 별도 레이어가 예측한 2D offset을 더해 실제 샘플링 위치를 입력 내용에 따라 동적으로 이동시키는 메커니즘. 오프셋은 분수(fractional) 좌표를 가질 수 있어 bilinear interpolation으로 값을 읽는다. 별도 supervision 없이 표준 역전파만으로 학습된다.

# 등장 논문
- [[Deformable_Convolutional_Networks]] — 원조. Convolution의 3×3 grid와 RoI pooling의 spatial bin 위치에 이 메커니즘을 최초로 적용(deformable convolution, deformable RoI pooling).
- [[Deformable-DETR]] — 이 개념을 CNN에서 Transformer attention으로 이식. 각 query가 reference point 주변 K개 학습된 offset 위치만 attend하는 deformable attention module 제안. `K=1,L=1,W'm=I`로 축소하면 원조 deformable convolution과 수식적으로 동일함을 저자들이 직접 증명(Appendix A.1) — 두 메커니즘이 정확한 일반화 관계.

# 변형/발전
시간 순 정리(등장 논문이 늘어날 때마다 갱신):
- 2017: Dai et al.의 원조 deformable convolution — CNN의 고정 grid convolution 샘플링에 적용. Offset만 학습(가중치는 고정 커널 가중치를 그대로 사용).
- 2021: Zhu et al.의 Deformable DETR — offset 예측에 더해 attention weight(softmax 정규화)까지 함께 학습하도록 확장, 이를 통해 deformable convolution에 없던 element 간 relation modeling 능력을 추가. Multi-scale 버전은 레벨별로 별도 offset을 둬 FPN 없이 멀티스케일 feature를 통합.

# 관련 개념
- (아직 없음 — 이후 dynamic query DETR 계열 처리 시 관련성 판단 예정)
