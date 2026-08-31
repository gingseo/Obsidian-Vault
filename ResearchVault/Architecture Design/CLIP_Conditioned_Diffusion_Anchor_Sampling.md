---
title: "CLIP-Conditioned Diffusion Anchor Sampling"
tags: [architecture, diffusion, vision-language-model, sampling]
created: 2026-08-28
updated: 2026-08-28
---

# 역할
객체 탐지에서 "어디를 양성(positive) 샘플로 볼지" 결정하는 anchor 샘플링 포인트 자체를, 순수 기하학적 기준(거리·IoU)이 아니라 **DDPM/DDIM 기반 diffusion denoising 과정**으로 반복 정제하는 기법. 이때 denoising의 각 단계에 CLIP의 이미지-텍스트 의미 임베딩을 조건(condition)으로 주입해, "기하학적으로 그럴듯한 위치"뿐 아니라 "의미적으로 실제 객체일 법한 위치"로 샘플이 수렴하도록 편향시킨다.

# 구조

## 입력/출력
- 입력: GT 박스에서 Gaussian sampler로 생성한 초기 샘플 포인트 `x_0` (개수는 GT 박스 면적에 비례), CLIP 이미지 인코더의 계층적 feature `{F¹_clip, ..., F^L_clip}` (L=12), CLIP 텍스트 인코더가 만든 클래스 프롬프트 임베딩(768차원).
- 출력: 노이즈로부터 반복적으로 정제된 최종 샘플링 포인트 위치 — 이후 spatial calibration(deformable conv)으로 feature map 격자에 맞게 보정되어 detection head에 입력된다.

## 내부 동작
1. **Forward process(노이즈 주입)**: 초기 샘플 `x_0`에 timestep `t`만큼 점진적으로 가우시안 노이즈를 섞는다.
   `x_t = √ᾱ_t · x_0 + ε√(1-ᾱ_t)`, `ε ~ N(0, I)` — `ᾱ_t`는 timestep별로 미리 정해진 noise schedule의 누적곱.
2. **CLIP 조건 결합**: 텍스트 임베딩 `F_text`를 공간적으로 복제해 이미지 해상도에 맞추고(`f_ta`), 이미지 CLIP feature는 MLP로 투영해(`f_is`) 두 modality-specific 표현을 만든 뒤, `F_fusion = σ(W_text⊙f_ta) ⊕ (W_image⊙f_is)`로 융합한다. 이 융합 결과를 diffusion U-Net의 conditional encoding stream에 residual 방식으로 단계마다 주입한다.
3. **Reverse process(디노이징, DDIM 가속)**: 매 timestep 전부를 순차적으로 밟지 않고 DDIM으로 건너뛰며 역방향 샘플을 근사한다.
   `x_p = √ᾱ_p·((x_t - √(1-ᾱ_t)·ε_t)/√ᾱ_t) + √(1-ᾱ_p-σ²)·ε_t + σε`
   이 과정을 학습 시 3회 반복 수행.
4. **Spatial calibration**: 디노이징 후 위치가 원래 feature map 격자와 어긋나므로, 미리 정한 균일 샘플링 참조점과 디노이즈된 점 사이의 위치 편차를 측정해 deformable convolution의 offset으로 변환, feature map 자체를 그 편차만큼 보정한다.

> [!example]- 수식 요약
> ```
> x_t = √ᾱ_t·x_0 + ε√(1-ᾱ_t)                              # forward
> σ² = η·(1-ᾱ_p)/(1-ᾱ_t)·(1-ᾱ_t/ᾱ_p)                       # DDIM 분산
> x_p = √ᾱ_p·((x_t-√(1-ᾱ_t)ε_t)/√ᾱ_t) + √(1-ᾱ_p-σ²)ε_t + σε # reverse (DDIM)
> F_fusion = σ(W_text⊙f_ta) ⊕ (W_image⊙f_is)                # CLIP 조건 융합
> ```

# 왜 이렇게 되는가
- **왜 diffusion인가(단순 랜덤 샘플링이 아니라)**: Gaussian sampler로 뽑은 초기 포인트는 GT 박스 형태를 반영할 뿐 "실제로 이 위치가 객체를 잘 대표하는지"는 모른다. Diffusion의 반복적 denoising 구조를 빌리면, 노이즈로부터 원래 분포를 복원하는 과정 자체를 "덜 신뢰할 만한 포인트를 더 신뢰할 만한 위치로 점진적으로 옮기는" 정제 절차로 재해석할 수 있다 — 매 step이 조건(CLIP 임베딩)을 참고하며 위치를 조금씩 고쳐나간다.
- **왜 CLIP을 조건으로 주입하는가**: RFLA류의 Gaussian receptive field 매칭은 anchor와 GT 사이의 거리·형태 유사도만 본다 — 배경의 강한 산란체(SAR)나 시각적으로 애매한 소형 객체를 "기하학적으로는 그럴듯하지만 실제로는 아닌" 위치로 잘못 판단할 수 있다. CLIP은 이미지-텍스트 대조학습으로 "이 영역이 [CLASS]와 의미적으로 얼마나 부합하는지"를 이미 알고 있으므로, 이 신호를 diffusion 조건으로 얹으면 기하학적 정제와 의미적 정제를 같은 파이프라인에서 동시에 수행할 수 있다.
- **비용**: 1000 timestep짜리 전체 diffusion을 그대로 쓰면 추론이 매우 느려지므로 DDIM으로 스텝 수를 줄여 근사한다 — 그래도 반복 없이 한 번에 위치를 예측하는 방식보다는 여러 스텝의 U-Net forward가 필요해 연산 비용이 늘어나는 트레이드오프가 있다.

# 등장 논문
- [[CDATOD-Diff]] — GT 박스 면적 비례 개수의 Gaussian 샘플을 생성한 뒤, CLIP 이미지·텍스트 임베딩을 조건으로 한 DDIM 기반 diffusion denoising으로 anchor 샘플링 포인트를 정제. RFLA의 계층적 Gaussian receptive field 매칭과 결합해 소형 객체의 양성 샘플 부족 문제를 완화하는 데 사용.
