---
title: "Frequency-Domain Feature Attention"
tags: [concept, object-detection, frequency-domain, attention, spectral-analysis]
created: 2026-08-04
updated: 2026-08-04
---

# 정의
이미지 또는 feature map을 공간(spatial) 영역이 아니라 주파수(frequency) 영역으로 변환(2D-DFT, 2D-DCT, wavelet 등)한 뒤, 특정 주파수 대역(주로 고주파=경계·텍스처, 저주파=매끄러운 배경)에 학습 가능한 가중치를 적용해 선택적으로 강조하거나 억제하는 기법. 주파수 변환 자체는 정보 손실이 없는 가역 연산이므로, 성능 향상의 원천은 "새로운 정보"가 아니라 고주파/저주파가 자연스럽게 분리되어 있어 spatial-domain보다 더 유연하고 해석 가능한 방식으로 특징을 조작(필터링)할 수 있다는 데 있다. Spatial attention(예: 채널/공간 attention)이 원본 픽셀·특징 배치에서 직접 가중치를 학습하는 것과 달리, 이 기법은 먼저 신호를 주파수 성분으로 분해한 뒤 그 성분에 대해 가중치를 학습한다는 점이 구별점이다.

# 등장 논문
- [[FANet]] — Multi-Scale Frequency Feature Enhancement Module(MSFFEM, feature map을 patch로 나눠 2D-DFT 적용 후 적응형 주파수 가중치로 tiny object의 contour/texture를 강조하고 배경을 억제)과 Channel Attention-based RoI Enhancement Module(CAREM, RoI feature에 2D-DCT 기반 Gaussian 고주파 필터를 적용한 뒤 채널 attention으로 고주파 응답이 강한 채널을 선택)이라는 두 가지 형태로 이 개념을 확장·적용.

# 변형/발전
- 원류: Discrete Fourier Transform(DFT)/wavelet transform을 이용해 노이즈·조명 변화에 덜 민감한 주파수 성분을 추출하는 고전적 신호처리 접근이 컴퓨터 비전에 도입됨.
- CNN과 Fourier transform을 결합해 spatial 특징과 주파수 정보를 함께 활용하는 연구들(camouflaged object detection, remote sensing semantic segmentation 등)로 확장.
- SpectFormer(Patro et al., 2023) — Vision Transformer의 self-attention을 spectral attention으로 대체해 주파수 영역에서 특징 표현을 학습.
- HS-FPN(Shi et al., AAAI 2025) — tiny object의 고주파 응답을 attention 메커니즘으로 활용하는 high-frequency spatial perception FPN. FANet의 CAREM이 이 방향의 필터 설계(0-1 필터)를 일부 참고.
- FANet(2025) — 기존 연구가 대부분 범용 객체 검출이나 다른 태스크(위장 객체 탐지, segmentation)를 겨냥한 것과 달리, 원격탐사 tiny object에 특화해 (1) feature map 레벨에서 patch 단위 multi-scale 주파수 강화(MSFFEM)와 (2) RoI 레벨에서 고주파 필터+채널 attention 결합(CAREM)이라는 두 지점에 동시에 적용하고, 두 모듈이 상호 보완적으로 작동함을 실험적으로 확인. 또한 patch size가 클수록 medium object에, 작을수록 very tiny object에 유리하다는 스케일-주파수 해상도 trade-off를 정량적으로 분석.

# 관련 개념
-
