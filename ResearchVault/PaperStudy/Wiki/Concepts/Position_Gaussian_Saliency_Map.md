---
title: "Position Gaussian Saliency Map"
tags: [concept, object-detection, feature-enhancement, gaussian-mixture, attention]
created: 2026-08-04
updated: 2026-08-04
---

# 정의
객체의 위치(중심)와 크기를 2D Gaussian 성분의 평균·공분산으로 인코딩하고, 이미지(또는 feature map) 전체 인스턴스에 대한 Gaussian Mixture를 합산해 "위치 기반 saliency map"을 만드는 기법. 박스 회귀의 불확실성을 표현하는 것이 목적이 아니라, 어느 공간 위치가 객체(특히 tiny object)인지를 나타내는 supervision/attention prior를 생성하는 것이 목적이다. 객체 크기가 작을수록 공분산을 더 작게(분포를 더 뾰족하고 값이 크게) 만들어, tiny object가 일반 크기 객체보다 맵 상에서 더 두드러지도록 설계하는 것이 핵심 트릭이다. 이렇게 만든 맵은 (a) 지도학습의 supervision target으로 network가 이 saliency map을 예측하도록 학습시키거나, (b) feature map에 곱/덧셈으로 결합되는 attention prior로 사용된다.

# 등장 논문
- [[Feature_Info_Driven_Gaussian]] — 각 GT box를 중심 (x,y), 공분산 diag((w/α)²,(h/α)²)인 Gaussian으로 모델링(α는 very tiny/tiny/small/general 크기별로 4/6/8/10 다르게 적용해 tiny일수록 뾰족한 분포). N개 인스턴스의 Mixture를 합산·threshold 후처리해 GT map을 만들고, multi-scale network(P2~P4)가 이를 예측하도록 지도학습. 예측된 맵을 feature에 `y⊗(1+M_pd)` 형태로 곱해 tiny object 영역의 feature activation을 강화하는 데 사용.

# 변형/발전
- 원조/최초 등장: Feature Information Driven Position Gaussian Distribution Estimation (CVPR 2025) — 이 논문에서 처음 "Position Gaussian Distribution Map"이라는 이름으로 도입. 크기 의존적 covariance scaling(scaling factor α)이 핵심 설계 포인트로, 논문 자체 ablation에서 고정 α, binary mask, self-attention 기반 대안보다 우수함을 확인.
- 관련 아이디어의 계보: RFLA(ECCV 2022)의 "Gaussian receptive field" 개념 및 Wang et al. 2020(center probability map)에서 "객체를 2D Gaussian으로 표현"하는 아이디어 자체는 선행 연구에 있으나, 이들은 주로 label assignment(어떤 anchor/prior를 positive로 볼지)에 사용됨. 이 논문은 동일한 "Gaussian으로 객체 위치 인코딩" 아이디어를 label assignment가 아니라 feature enhancement(attention prior/supervision)라는 다른 목적에 적용했다는 점이 차별점.
- 아직 이 논문 1편에서만 등장 — 향후 다른 논문에서 유사하게 재사용되는지 지켜볼 필요.

# 관련 개념
- [[Gaussian_Box_Uncertainty_Modeling]] — 둘 다 "객체를 Gaussian으로 모델링"하지만, 전자는 박스 회귀 좌표의 예측 불확실성(NMS/sampling 개선용)을, 이 개념은 위치 기반 saliency/attention(feature enhancement용)을 표현한다는 점에서 용도가 명확히 다르다.
