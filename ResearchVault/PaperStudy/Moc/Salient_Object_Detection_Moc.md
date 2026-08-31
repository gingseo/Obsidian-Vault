---
title: "Salient Object Detection MOC"
tags: [moc]
task: salient-object-detection
created: 2026-08-04
updated: 2026-08-24
---

# 이 분야가 다루는 핵심 질문
- Salient object detection에서 예측이 애매한(저채도/경계 불명확) 영역을 어떻게 식별하고, 그 영역만 반복적으로 정제(refine)할 것인가?
- 원격탐사 이미지 특유의 극심한 스케일 변화·복잡한 배경 속에서, 전역 문맥(semantic)과 국소 디테일(경계·텍스처) 사이의 균형을 어떻게 잡을 것인가?

# 지금까지 다룬 흐름
2편을 읽었으며, 서로 다른 도메인(자연 이미지 vs 원격탐사)과 접근(반복 정제 vs 다중 레벨 feature 통합)을 대표한다.
- [[Uncertainty_Guided_Refinement]] — 자연 이미지 SOD. 별도의 경계 라벨/가이던스 없이, 예측 saliency map 자체에서 불확실성 맵을 만들어 attention을 마스킹하는 방식으로 반복 정제.
- [[AIMRINet]] — 원격탐사(ORSI) SOD. PVT-v2 backbone의 얕은/깊은 feature에 점진적 그룹 spatial attention(SAI)을 적용하고, 다중 레벨 feature의 공통 saliency를 곱셈으로 추출한 뒤 residual로 고유 정보를 보존하는 통합(MRFI)을 결합. Uncertainty_Guided_Refinement와 달리 반복적 정제가 아니라 단일 forward pass에서 다중 레벨 feature 상호작용으로 완결되는 구조.

두 논문은 "무엇이 애매한 영역인지 어떻게 식별할 것인가"(URA)와 "여러 레벨의 정보를 어떻게 결합해야 완전한 경계를 얻는가"(AIMRINet)라는 서로 다른 질문에 답한다는 점에서 상호보완적이다.

# 이 분야를 관통하는 개념
- [[Uncertainty_Masked_Refinement_Attention]] — uncertainty-guided-refinement의 핵심 기여. Small object detection 쪽의 [[Gaussian_Box_Uncertainty_Modeling]]과 이름은 비슷하지만, 학습된 분포 파라미터가 아니라 예측값 자체에서 결정론적으로 유도되는 맵이라는 점에서 메커니즘이 다르다(각 개념 문서에 상호 링크 있음).
- [[Progressive_Grouped_Spatial_Attention_Interaction]] — AIMRINet의 SAI 핵심 기여. Channel shuffle+그룹 분할 순차 spatial attention.
- [[Multiplicative_Residual_Saliency_Integration]] — AIMRINet의 MRFI 핵심 기여. 곱셈으로 공통 saliency 추출, residual로 레벨별 고유 정보 보존.

# 비교 문서
(아직 없음 — 2편이 도메인·접근이 상당히 달라(자연 이미지 반복 정제 vs 원격탐사 다중 레벨 통합) 비교표를 만들기엔 이르다. 3편째가 추가되면 검토)

# 아직 못 채운 빈틈
- 원격탐사 SOD([[AIMRINet]])와 자연 이미지 SOD([[Uncertainty_Guided_Refinement]])를 같은 프레임워크에서 직접 비교한 논문이 없어, 두 도메인 간 접근법의 실질적 차이(전이 가능성)를 아직 검증할 수 없다.
- AIMRINet이 참고문헌에서 반복 인용하는 다수의 최신 ORSI-SOD 논문(TSCNet, UDCNet-R, LGIPNet 등)이 아직 이 위키에 없어, 이 서브필드의 전체 지형을 파악하기엔 이르다.

# 관련 MOC
- [[000-Home]]
- [[Small_Object_Detection_Moc]] — 원격탐사 도메인을 공유하는 인접 분야. AIMRINet의 MRFI(곱셈+residual 통합)와 [[ORFENet]]의 MRFAFEM(동적 가중합)이 "여러 소스를 결합하되 원본 정보를 잃지 않는다"는 유사한 상위 패턴을 공유한다.
