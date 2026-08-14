---
title: "Gaussian Box Uncertainty Modeling"
tags: [concept, object-detection, uncertainty, bounding-box-regression]
created: 2026-08-04
updated: 2026-08-04
---

# 정의
바운딩 박스의 각 변(top/left/bottom/right) 좌표를 결정론적 값이 아니라 평균과 표준편차를 갖는 Gaussian 분포로 예측하는 기법. 표준편차(σ)는 모델이 해당 좌표 예측에 대해 갖는 불확실성(confidence)을 나타낸다. GT 박스는 표준편차가 0인 Dirac delta 함수로 취급하고, 예측 분포와 GT 분포 간 KL divergence(또는 negative log-likelihood)를 최소화하는 방식으로 학습한다.

# 등장 논문
- [[Unc-SOD]] — RPN에 위치 불확실성 예측 branch를 추가할 때 이 방식을 그대로 채택(He et al. 2019 기반). 예측된 σ를 instance-level uncertainty로 집계해 동적 sampling 기준으로 활용하는 것이 이 논문의 확장 지점.

# 변형/발전
- 원조: He et al., "Bounding Box Regression with Uncertainty for Accurate Object Detection" (CVPR 2019) — KL divergence 최소화로 학습, 예측 표준편차로 NMS를 개선(낮은 불확실성 박스를 우선).
- Gaussian YOLOv3 — 실시간 detector에 동일 아이디어를 적용해 False Positive 감소.
- GFL(Generalized Focal Loss) — Gaussian 같은 특정 분포 가정 대신, 이산화된 확률분포를 직접 학습하는 방향으로 발전(더 유연한 분포 표현).
- Unc-SOD(2026) — 개별 박스의 불확실성이 아니라, 여러 positive prior의 불확실성을 IoU 기반 비선형 가중합으로 집계해 "인스턴스 단위" 불확실성으로 확장하고, 이를 sampling(assignment) 기준으로 전용(轉用)한 것이 새로운 지점. 기존 연구들이 불확실성을 NMS/regression 품질 개선에만 썼다면, Unc-SOD는 "어떤 prior를 positive로 볼지"를 결정하는 데 사용.

# 관련 개념
- [[perception-and-interaction]]
