---
title: "LaRE²: Latent Reconstruction Error Based Method for Diffusion-Generated Image Detection (원문 요약)"
tags: [paper-source]
created: 2026-08-05
---

[[lare2]]의 원문 요약. 분석·평가는 담지 않는다 — 순수 번역/발췌.

# Abstract
Diffusion Model의 발전은 이미지 생성 품질을 극적으로 향상시켜, 실제 이미지와 생성 이미지를 구별하는 것을 점점 더 어렵게 만들고 있다. 이러한 발전은 인상적이지만 동시에 심각한 프라이버시와 보안 우려를 제기한다. 이에 대응해, 우리는 diffusion 생성 이미지를 탐지하기 위한 새로운 Latent REconstruction error 기반 feature REfinement 방법(LaRE²)을 제안한다. 우리는 생성 이미지 탐지를 위한 latent space 기반 최초의 reconstruction-error feature인 Latent Reconstruction Error(LaRE)를 고안했다. LaRE는 실제와 가짜를 구별하는 데 필요한 핵심 단서를 보존하면서도 feature 추출 효율성 면에서 기존 방법을 능가한다. LaRE를 활용하기 위해, LaRE의 가이드로 이미지 feature를 정제해 feature의 판별력을 강화하는 Error-Guided feature REfinement 모듈(EGRE)을 제안한다. 우리의 EGRE는 align-then-refine 메커니즘을 활용해, 공간(spatial)과 채널(channel) 두 관점 모두에서 생성 이미지 탐지를 위한 이미지 feature를 효과적으로 정제한다. 대규모 GenImage 벤치마크에서의 광범위한 실험은 우리 LaRE²의 우수성을 입증하며, 8개의 서로 다른 이미지 생성기에서 평균 ACC/AP 기준 최고 SoTA 방법을 최대 11.9%/12.1% 능가한다. LaRE는 또한 feature 추출 비용 측면에서도 기존 방법을 능가해, 인상적인 8배 속도 향상을 제공한다. 코드는 공개되어 있다.

# Introduction

#### Diffusion Model 발전과 새로운 우려
모델 구조·학습 전략·샘플링 방법의 개선으로 diffusion model이 상상을 초월하는 품질의 이미지를 생성하게 되면서, 생성 이미지의 프라이버시·보안 문제(유해 콘텐츠·허위정보 확산)가 커지고 있다.

#### 기존 reconstruction 기반 탐지의 한계
DIRE는 diffusion 생성 이미지가 diffusion model로 더 쉽게 재구성된다는 가정에 기반해 재구성 오차를 판별 feature로 쓰지만, DDIM inversion에 수십 단계의 샘플링이 필요해 이미지당 2초 이상 소요되고, 다단계 과정에서 오차가 누적되며, 재구성 오차만 단독 feature로 쓰고 원본 이미지와의 대응관계는 무시한다.

#### 이 논문이 던지는 두 질문
이미지를 완전히 재구성해야만 판별 feature를 얻을 수 있는가, 그리고 재구성 오차를 생성 이미지 탐지에 결합하는 더 나은 방법이 있는가라는 두 질문에서 출발한다.

#### 탐색적 실험 결과
Diffusion model의 forward process가 닫힌 형태(closed-form) 해를 가지므로 임의의 timestep에서 단일 단계 디노이징만으로도 재구성 오차를 얻을 수 있으며, 실제 이미지와 생성 이미지의 손실 값에 뚜렷한 격차가 있음을 확인했다. 또한 재구성 손실이 원본 이미지의 국소 정보 주파수(고주파 영역일수록 손실이 큼)와 양의 상관관계를 보였다.

#### 제안 방법 개요
Latent space에서 단일 단계 재구성으로 LaRE를 추출하고, 이를 공간·채널 두 관점에서 이미지 feature 정제에 활용하는 EGRE 모듈을 결합한 LaRE²를 제안한다.

#### 실험 결과 요약
GenImage 벤치마크에서 LaRE는 기존 방법 대비 8배 빠른 feature 추출 속도를 보이며, LaRE²는 이전 SoTA 대비 최대 11.9%/12.1% ACC/AP 향상을 달성했다.

#### 기여 요약
Latent space 기반 최초의 reconstruction-error feature 제안, 재구성 손실의 효과성을 정성적으로 분석해 고안한 EGRE 모듈, 그리고 대규모 벤치마크에서의 우수한 성능을 기여로 제시한다.

# Main Contribution
1. **새로운 feature**: Generated-image 탐지를 위해 latent space에서의 재구성 오차를 최초로 제안했다. 기존 방법 대비 탐지에 필요한 핵심 정보를 보존하면서도 feature 추출 비용을 현저히 줄였다.
2. **새로운 모듈**: 재구성 손실이 효과적인 이유를 정성적으로 분석하고, 이를 바탕으로 이미지 feature의 판별력을 강화하는 error-guided feature refinement를 수행하는 새로운 모듈 EGRE를 고안했다.
3. **우수한 성능**: 광범위한 실험으로 방법의 효과성을 입증했다. 대규모 GenImage 벤치마크에서 SoTA 방법을 크게 능가하는 11.9%/12.1% ACC/AP 향상을 달성했다.

# Conclusion
이 논문에서는 LaRE²라는 새로운 reconstruction 기반 diffusion 생성 이미지 탐지 방법을 제안했다. 우리는 latent space에서 이미지를 재구성함으로써 더 효율적인 reconstruction 기반 feature인 LaRE를 고안했다. 특히 LaRE는 기존 reconstruction 기반 방법 대비 8배 빠르다. LaRE를 Error-guided Feature Refinement 모듈(EGRE)과 결합함으로써, 우리의 LaRE²는 diffusion 생성 이미지 탐지에서 우수한 성능을 달성하며 state-of-the-art 성능을 입증한다.
