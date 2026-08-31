---
title: "Latent Reconstruction Error (LaRE)"
tags: [concept, ai-generated-image-detection, diffusion-model, reconstruction-error, latent-space]
created: 2026-08-05
updated: 2026-08-05
---

# 정의
사전학습된 diffusion model의 forward process가 닫힌 형태(closed-form) 해를 갖는다는 성질을 이용해, 완전한 다단계 재구성(수십 스텝의 노이즈 추가·제거) 없이 latent space에서 단 한 번의 디노이징 스텝만으로 재구성 오차를 근사하는 기법. 원본 이미지를 VAE로 latent code `x0`로 인코딩하고, 임의의 timestep `t`에서 노이즈 `ε`를 더한 `xt`를 직접 계산한 뒤, diffusion U-Net으로 단일 스텝 디노이징을 수행해 예측 노이즈와 실제 노이즈의 차이를 오차로 삼는다. "diffusion 생성 이미지는 diffusion model로 더 쉽게 재구성된다"는 가정 아래, 완전 재구성의 최종 오차가 작다면 그 과정을 이루는 개별 스텝의 오차도 이미 작다는 논리로, 다단계 샘플링(DIRE류) 없이도 동일한 판별 신호를 훨씬 적은 연산으로 얻는다.

# 등장 논문
- [[LaRE2]] — 이 기법의 원조 제안. Latent space + 단일 스텝 디노이징으로 DIRE 대비 8배 빠른 feature 추출을 달성하면서 GenImage 벤치마크에서 SOTA 정확도(ACC/AP +11.9%/+12.1%) 확보.

# 변형/발전
아직 원조 논문 1편에서만 등장. 향후 다른 논문이 이 기법을 채택하거나 변형하면 여기에 추가.

# 관련 개념
- [[Self_Reconstruction_Difference_Map]] — "재구성이 어려운 정도를 판별 신호로 쓴다"는 원리를 공유하지만, 대상 도메인(tiny object detection vs 생성 이미지 탐지)과 재구성 대상(feature map→원본 이미지 vs 사전학습 diffusion model의 노이즈 예측)이 다르다.
- [[ReContrast_Dual_Encoder_Contrastive_Reconstruction]] — 마찬가지로 reconstruction 기반이지만, encoder를 직접 학습시키는 방식이라 사전학습 모델을 고정한 채 오차만 추출하는 LaRE와 대조된다.
