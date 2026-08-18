---
title: "Self-Reconstruction Difference Map"
tags: [concept, object-detection, tiny-object-detection, feature-enhancement, self-supervision]
created: 2026-08-04
updated: 2026-08-04
---

# 정의
탐지 모델의 neck(FPN)에서 나온 저수준 feature map으로부터 원본 입력 이미지를 복원하는 reconstruction head를 붙이고, 복원된 이미지와 원본 이미지의 픽셀 차이("difference map")를 계산하는 기법. 이미지 재구성은 픽셀 변화에 민감한 low-level vision 과제이므로, backbone의 downsampling 과정에서 구조·텍스처 정보가 심하게 손실된 영역일수록 복원이 어려워 difference map에서 큰 값을 갖는다. 이 성질을 이용해, 별도의 supervision 없이(원본 이미지 자체가 재구성 target) 정보 손실이 심한 영역 — 전형적으로 tiny object가 있는 영역 — 을 찾아내는 self-supervised prior로 활용한다. 생성(GAN/super-resolution) 기반 방법과 달리 없는 디테일을 새로 만들어내는 것이 아니라, "어디서 정보가 사라졌는가"를 진단하는 방식이라는 점이 핵심 차별점이다.

# 등장 논문
- [[SR-TOD]] — 이 개념을 최초로 제안. FPN의 P2 feature map에서 reconstruction head로 이미지를 복원해 difference map을 얻고, Difference Map Guided Feature Enhancement(DGFE) 모듈을 통해 difference map을 element-wise attention 형태로 tiny object feature 강화에 사용. Pixel-level difference map 외에 FFT 기반 high-frequency difference map도 실험.

# 변형/발전
- 원조(SR-TOD, ECCV 2024): reconstruction head는 U-Net과 FPN의 구조적 유사성에서 착안한 단순한 Up Block(Transpose Conv + Conv + ReLU) 스택으로 구성되며, MSE reconstruction loss로 학습. Difference map은 학습 가능한 threshold로 이진화(filtration)한 뒤 채널 방향 reweighting과 결합해 attention matrix로 변환.
- 저자들이 논문 내에서 직접 시도한 변형: pixel-level difference map(원본) vs. high-frequency difference map(FFT로 고주파 성분만 추출 후 차이 계산) — 후자가 near-noise 수준의 매우 작은 물체 신호를 일부 더 죽이는 대신 윤곽선이 더 선명해지는 trade-off 존재.
- 이후 다른 논문에서의 확장 사례는 아직 위키에 없음(2026-08-04 기준 SR-TOD 1편에만 등장).

# 관련 개념
- [[Latent_Reconstruction_Error]] — "재구성이 어려운 정도를 판별 신호로 쓴다"는 원리를 공유하는 다른 도메인(AI 생성 이미지 탐지)의 유사 기법. Feature map→원본 이미지 재구성(이 개념)과 사전학습 diffusion model의 노이즈 예측 오차(LaRE)라는 재구성 대상의 차이가 있다.
