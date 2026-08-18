---
title: "Dual-Encoder Contrastive Reconstruction (ReContrast)"
tags: [concept, anomaly-detection, contrastive-learning, feature-reconstruction, domain-adaptation, stop-gradient]
created: 2026-08-04
updated: 2026-08-04
---

# 정의
Feature reconstruction 기반 비지도 이상 탐지(UAD)에서, 전통적으로 pattern collapse를 피하기 위해 반드시 고정(frozen)해야 했던 사전학습 encoder를 decoder와 함께 end-to-end로 학습 가능하게 만드는 프레임워크. Positive-pair contrastive self-supervised learning(SimSiam류)의 세 가지 요소 — ① 전역(global) 관점의 거리 계산, ② stop-gradient 연산, ③ 두 개의 view를 이용한 contrastive pair 구성 — 을 feature reconstruction 구조에 이식해서 구현한다. 이때 두 번째 view는 이미지 증강(augmentation) 대신, 사전학습 상태로 고정된 frozen encoder와 target 도메인에서 학습되는 domain-specific encoder라는 "서로 다른 두 encoder"로 대체한다. Decoder는 두 encoder의 특징을 상호(cross) 재구성하도록 학습되어, encoder가 domain-specific 정보를 흡수하면서도 무의미한 특징으로 붕괴(pattern collapse)하거나 정상/이상을 구분 없이 잘 재구성해버리는 "identical shortcut"에 빠지지 않는다.

핵심 구성 요소:
- **Global cosine distance**: point-by-point regional distance 대신 feature map 전체를 flatten해 계산하는 cosine distance. Loss landscape를 평탄하게 만들어 학습 안정성을 높인다.
- **Stop-gradient**: decoder(predictor 역할)에서 encoder로 직접 흐르는 gradient를 차단하고, encoder-decoder 간 "상호 강화"로 학습해 encoder feature의 diversity 붕괴를 방지한다.
- **Dual-encoder contrastive pair**: frozen encoder(사전학습 도메인 view)와 domain-specific encoder(target 도메인 view) 두 개를 두어 augmentation 없이도 contrastive pair를 구성한다.
- **Hard-normal mining**: 정상 영역 중 재구성이 원래 어려운 edge/디테일 영역(hard-normal)의 gradient에 집중시켜, 이 intrinsic error가 실제 이상에 의한 epistemic error와 혼동되는 것을 완화한다.

# 등장 논문
- [[ReContrast]] — 이 개념을 최초로 제안한 논문. RD4AD(Reverse Distillation)를 baseline으로 삼아 위 네 요소를 단계적으로 도입하며, MVTec AD/VisA 등 산업 결함 탐지와 OCT2017/APTOS/ISIC2018 등 의료 영상 UAD에서 SOTA를 달성.

# 변형/발전
- 원조: 이 프레임워크 자체가 ReContrast(NeurIPS 2023)에서 처음 제안됨. Reverse Distillation(RD4AD, CVPR 2022)의 frozen-encoder feature reconstruction 구조를 출발점으로 삼고, SimSiam(CVPR 2021)의 stop-gradient 기반 collapse 방지 아이디어를 결합해 만들어졌다.
- 아직 이 논문 1편에서만 등장. 추후 다른 논문이 이 프레임워크를 확장·재사용하면 여기에 갱신.

# 관련 개념
- [[Latent_Reconstruction_Error]] — 마찬가지로 reconstruction 기반이지만, 사전학습된 diffusion model을 고정한 채 오차만 추출하는 방식이라 encoder 자체를 학습시키는 이 개념과 대조된다.
