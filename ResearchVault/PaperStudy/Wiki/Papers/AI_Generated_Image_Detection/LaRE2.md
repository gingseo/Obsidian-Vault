---
title: "LaRE²: Latent Reconstruction Error Based Method for Diffusion-Generated Image Detection"
authors: [Yunpeng Luo, Junlong Du, Ke Yan, Shouhong Ding]
year: 2024
venue: "CVPR"
jcr_quartile: Q1
task: [ai-generated-image-detection]
direction: [novel-approach]
tags: [paper, ai-generated-image-detection, diffusion-model, reconstruction-error, latent-space, feature-refinement, deepfake-detection]
status: read
user_read: false
added: 2026-08-05
source: "raw/ai-generated-image-detection/2024_CVPR_LaRE2.pdf"
created: 2026-08-05
---

# 한 줄 요약
<mark style="background: #FFF3A3A6;">Diffusion model의 forward process가 닫힌 형태 해를 갖는다는 성질을 이용해, latent space에서 단 한 번의 디노이징 스텝만으로 재구성 오차(LaRE)를 추출하고, 이 오차를 공간·채널 두 관점에서 attention으로 결합해 이미지 feature를 정제함으로써, 기존 방법 대비 8배 빠르면서 정확도는 크게 앞서는 diffusion 생성 이미지 탐지 방법.</mark>


# 문제 정의

### 기존 방법의 한계
- **완전 재구성의 연산 비용**:
  DIRE는 DDIM inversion으로 이미지를 노이즈로 변환한 뒤 다시 생성하는 수십 단계 샘플링 과정을 거쳐야 해, 이미지 1장당 Tesla V100 GPU에서 2초 이상 소요된다 — 실시간 응용에는 부적합.
- **다단계 과정의 오차 누적**:
  Forward/reverse Markov process를 여러 번 거치며 오차가 누적되어, 재구성 실패가 실제 이미지와 가짜 이미지의 차이 때문인지 재구성 과정 자체의 불안정성 때문인지 구분하기 어렵다. DDIM이 결정론적 inversion을 제공한다고 주장하지만 이 신뢰성이 항상 유지되지는 않는다.
- **재구성 오차 단독 사용의 정보 손실**:
  DIRE·SeDID는 재구성 오차만을 유일한 feature로 사용하고, 오차와 원본 이미지 간의 대응 관계(correspondence)를 무시한다 — 오차가 이미지의 어느 부분에서 발생했는지 활용하지 않는다.

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Hand-crafted feature 기반 탐지(GAN 시대)**
- Color cues [23], saturation cues [24], blending artifact [17], co-occurrence feature [25]: GAN 생성 이미지에 특화된 수작업 특징 — diffusion model에는 잘 일반화되지 않음.

**갈래 2 — CNN/주파수 기반 탐지**
- CNN 기반 탐지 [18, 22, 38]: GAN 이미지의 시각적 아티팩트를 학습.
- 주파수 영역 분석 [10, 28]: GAN 특유의 spectrum 아티팩트 탐지 — Corvi et al. [6, 7, 31]은 GAN용 탐지기가 diffusion 이미지에서 성능이 급락하며, diffusion 이미지에도 spectrum 아티팩트가 존재함을 발견.
- Wu et al. [41]: 언어 가이드 contrastive learning으로 합성 이미지 탐지를 식별(identification) 문제로 재구성 — 일반화력은 있으나 재구성 오차라는 신호 자체는 활용하지 않음.

**갈래 3 — Reconstruction error 기반 탐지(diffusion 특화)**
- DIRE [40]: "diffusion 생성 이미지는 diffusion model로 더 쉽게 재구성된다"는 가정에서 재구성 오차를 판별 feature로 사용 — 크로스모델 일반화력이 뛰어나지만 다단계 샘플링 비용이 큼.
- SeDID [21]: DIRE를 확장해 실제/생성 이미지 간 분포 차이를 추가로 활용 — 여전히 다단계 샘플링 필요.

**갭**: <mark style="background: #FFF3A3A6;">DIRE/SeDID는 "완전한 재구성"이 판별 feature 추출에 필수라고 전제하지만, diffusion model의 forward process가 닫힌 형태 해를 가지고 임의의 timestep에서 디노이징 손실을 계산할 수 있다는 성질을 활용하면 완전 재구성 없이도 동일한 판별 신호를 얻을 수 있다는 가능성은 검증되지 않았다. 또한 재구성 오차를 원본 이미지 feature와 공간적으로 대응시켜 활용하는 방법도 없었다.</mark>

### 이 논문이 풀고자 하는 문제
1. 이미지를 완전히 재구성하지 않고도 판별력 있는 reconstruction 기반 feature를 효율적으로 얻는 방법
2. 재구성 오차와 원본 이미지 feature의 공간적 대응 관계를 활용해 오차 신호를 더 효과적으로 결합하는 방법

# 제안 방법

<mark style="background: #FFF3A3A6;">Diffusion model의 forward process는 닫힌 형태 해를 가지므로 원본 x0에서 임의의 timestep t의 노이즈 이미지 xt를 직접 계산할 수 있고, 모델은 어떤 t에서도 노이즈를 예측하도록 학습되어 있다. 따라서 완전한 다단계 재구성 대신 단일 스텝 디노이징만으로 재구성 오차를 얻을 수 있으며, 이 오차가 실제/생성 이미지를 구별하는 신호로 충분하다는 가설을 latent space에서 검증하고, 이 오차의 공간적 분포(고주파 영역에서 오차가 큼)를 feature 정제에 활용한다.</mark>

### ① Latent Reconstruction Error (LaRE)
- VAE로 이미지를 latent code `x0`로 인코딩한 뒤, 임의의 timestep `t`에서 노이즈 `ε`를 추가해 `xt`를 닫힌 형태로 직접 계산.
- Diffusion U-Net `εθ`로 단 한 번의 디노이징만 수행해 예측 노이즈와 실제 노이즈의 차이(`Lε`)를 구하고, 그 제곱을 LaRE로 정의.
- Noise ensemble(`e`회 반복 후 평균)로 Monte Carlo 추정을 안정화.

> [!example]- 구현 디테일
> ```
> x_t = √ᾱ_t·x_0 + √(1-ᾱ_t)·ε                         (forward, closed-form)
> L_ε = ε - ε_θ(√ᾱ_t·x_0 + √(1-ᾱ_t)·ε, t)               (단일 스텝 디노이징 오차)
> LaRE = (1/e) Σ_i (L_εi ⊙ L_εi)                        (noise ensemble 평균)
> ```
> 최적 하이퍼파라미터: Stable Diffusion V1.5, t=200, e=4(속도-정확도 트레이드오프 최적점, Fig. 5), LaRE 크기 32×32×4(latent 해상도).

<mark style="background: #FFF9D6A6;">완전 재구성이 아니라 닫힌 형태 forward + 단일 스텝 reverse만으로 재구성 오차를 근사할 수 있다는 것이 핵심 통찰이다 — "완전 재구성된 이미지의 최종 오차가 작다"는 DIRE의 가정이 성립한다면, 그 재구성 과정을 이루는 개별 스텝들의 오차도 이미 작아야 한다는 논리로 다단계 샘플링 없이 동일한 판별 신호를 얻는다. 이는 "문제 정의"의 연산 비용·오차 누적 문제를 모두 근본적으로 제거한다.</mark>

### ② Error-Guided Feature Refinement (EGRE)
- LaRE를 원본 이미지에 시각화한 결과(Fig. 2), 손실이 추가 노이즈 크기에 비례할 뿐 아니라 이미지의 국소 주파수와도 양의 상관관계를 보임(고주파 전경에서 손실이 크고 저주파 배경에서 손실이 작음) — 이 공간적 대응 정보를 활용해 feature를 정제.
- **Error-guided Spatial Refinement (ESR)**: LaRE를 adaptive average pooling으로 이미지 feature map과 같은 공간 크기로 정렬한 뒤, multi-head attention의 attention score에 LaRE를 더해(`softmax(QK^T/√dk + E)V`) 공간적으로 중요한 영역을 재가중.
- **Error-guided Channel Refinement (ECR)**: LaRE와 feature map을 각각 전역 벡터로 squeeze한 뒤, LaRE를 학습 가능한 projection과 sigmoid를 거쳐 채널별 게이트로 사용(`sigmoid(ē·W) ⊙ x̄`).
- 두 정제 결과(`xs`, `xc`)와 원본 전역 feature(`xg`)를 concat해 최종 분류 feature 구성.

> [!example]- 구현 디테일
> ```
> ESA(Q,K,V,E) = softmax(QK^T/√d_k + E)V
> MHESA(Q,K,V,E) = Concat(head_1,...,head_h)W^O
> x_s = MHESA(x̄, x, x, e)                    # x̄=전역 query, x=feature map
> x_c = sigmoid(ē·W) ⊙ x̄
> x_EGRE = Concat(x_s, x_c, x_g) → FC → BCE loss
> ```
> Backbone은 CLIP 사전학습 ResNet50. Class-specific 프롬프트("a photo of CLS") 사용 시 추가로 1.4%/0.3% ACC/AP 향상(Table 2, Model E).

<mark style="background: #FFF9D6A6;">LaRE를 단독 feature로 쓰는 대신(Table 3, Model B는 성능이 크게 낮음) 원본 이미지 feature를 "어디를 더 봐야 하는지" 가이드하는 재가중 신호로 활용함으로써, "문제 정의"에서 지적한 재구성 오차-원본 이미지 간 대응 관계 무시 문제를 해결한다. 특히 공간 정렬 후 attention bias로 결합하는 방식(단순 concat 대비 ACC +7.7)은 대응 관계를 명시적으로 보존한다.</mark>

# 실험 결과

### 핵심 결과
| 벤치마크 | 지표 | Before(DIRE, 이전 SOTA) | After(LaRE²) |
|---|---|---|---|
| GenImage(8-generator 평균) | ACC / AP | 67.2 / - | 79.1 / - |
| Feature 추출 속도 | 상대 배율 | 1× (2초/이미지) | 8× 향상 |

> [!note]- 세부 결과 및 Ablation
> #### EGRE 구성요소 기여 (8-generator 평균)
> | 구성 | ESR | ECR | ACC | AP |
> |---|---|---|---|---|
> | Model A(baseline, LaRE 미사용) | | | 73.1 | 92.3 |
> | Model B | ✓ | | 81.7 | 97.6 |
> | Model C | | ✓ | 78.8 | 94.7 |
> | Model D(ESR+ECR) | ✓ | ✓ | 84.5 | 99.5 |
> | Model E(+class-specific prompt) | ✓ | ✓ | 85.9 | 99.8 |
>
> #### 입력 구성 비교 (SDv1.5 학습, 8-subset 테스트 평균)
> | 구성 | ACC | AP |
> |---|---|---|
> | 이미지만 | 73.1 | 92.3 |
> | LaRE만 | 66.2 | 68.3 |
> | 이미지+LaRE(단순 concat) | 76.8 | 93.5 |
> | 이미지+LaRE(EGRE) | 84.5 | 99.5 |
>
> #### 하이퍼파라미터 민감도
> - Noise ensemble `e`: e=4에서 속도-정확도 최적 균형(e를 늘리면 정확도는 소폭 상승하지만 런타임이 선형 증가).
> - Timestep `t`: t∈[150,300]에서 안정적 성능, 이 범위를 벗어나면 성능 저하.
>
> #### Cross-generator 일반화
> - 동일 생성기로 학습·평가 시(대각선) 모든 방법이 높은 성능.
> - BigGAN(GAN 기반)으로 학습한 모델은 diffusion 기반 생성기들에 잘 일반화되지 않는 반면, diffusion 기반 생성기(SDv1.5)로 학습한 모델은 다른 diffusion 계열에 상대적으로 잘 일반화 — 구조적 유사성이 일반화력에 영향.
> - t-SNE 시각화(Fig. 7): 미확인 생성기(MJ, ADM, BigGAN)에서 baseline은 real/fake feature가 크게 겹치는 반면, LaRE²는 겹침이 더 작음.

# Discussion

### 이 아이디어의 잠재적 부작용
- LaRE가 latent space(32×32×4)의 압축된 표현이라 원본 이미지 대비 정보 손실이 있음 → <mark style="background: #FF5582A6;">논문 스스로 LaRE 단독 사용 시 성능이 크게 떨어짐을 실험으로 확인했고(Table 3 Model B), 이를 원본 feature의 보조 신호로만 활용하는 설계로 완화했다고 주장.</mark>
- 일반 프롬프트("a photo") 사용 시 클래스 정보 없이도 성능이 크게 떨어지지 않는 현상 → <mark style="background: #FF5582A6;">논문은 "이미지가 완전히 노이즈로 변환되지 않아 원본 semantic 정보가 일부 남아있기 때문"이라 추측할 뿐 정량적으로 검증하지 않았다.</mark>

### 한계
- <mark style="background: #FF5582A6;">Stable Diffusion 기반으로 LaRE를 추출하므로, 학습에 쓰인 diffusion model 구조와 크게 다른 생성기(BigGAN 등 GAN 계열)에는 일반화력이 떨어진다(Fig. 4 cross-validation 결과).</mark>
- <mark style="background: #FF5582A6;">정확한 이미지 설명 프롬프트를 활용해 성능을 더 끌어올리는 방향은 향후 연구로 남겨두었다고 명시 — 현재는 범용 프롬프트만 사용.</mark>

### 생각할 점
- <mark style="background: #A6E3A1A6;">"재구성이 어려운 정도를 판별 신호로 쓴다"는 핵심 아이디어는 이 위키의 [[Self_Reconstruction_Difference_Map]](SR-TOD)와 구조적으로 동일하다 — SR-TOD는 정보 손실이 심한 영역(tiny object)을 찾는 데, LaRE는 생성 이미지 여부를 찾는 데 같은 원리를 쓴다는 점에서 도메인은 다르지만 원리가 겹친다.</mark>
- <mark style="background: #A6E3A1A6;">[[ReContrast_Dual_Encoder_Contrastive_Reconstruction]](anomaly-detection)도 reconstruction 기반 이상 탐지라는 점에서 유사 계열 — 다만 ReContrast는 encoder 자체를 학습시키는 반면, LaRE²는 사전학습된 diffusion model을 고정한 채 오차만 추출한다는 차이가 있다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">Diffusion model의 "닫힌 형태 forward + 단일 스텝 reverse"라는 효율화 전략은, [[SR-TOD]]가 쓰는 self-reconstruction head보다 더 정교한 사전학습 생성 모델 기반 신호 추출 방식으로 확장 가능성이 있어 보인다 — 다만 tiny object detection에는 별도의 대규모 diffusion 사전학습 모델이 필요하다는 진입장벽이 있다.</mark>
- <mark style="background: #A6E3A1A6;">AI-TOD 등 원격탐사 데이터셋 자체가 향후 diffusion 모델로 합성될 가능성을 고려하면, small-object-detection 논문들이 다루는 실제 위성/드론 영상과 생성 이미지를 구분하는 것도 데이터 품질 관리 차원에서 교차 관련성이 있을 수 있다.</mark>

# 관련 개념
- [[Latent_Reconstruction_Error]] — 이 논문의 핵심 기여

# 관련 문서
(아직 없음 — ai-generated-image-detection task의 첫 논문)

# 읽어볼 만한 논문
- 참고문헌 기반: Z. Wang, J. Bao, W. Zhou, W. Wang, H. Hu, H. Chen, H. Li, "DIRE for diffusion-generated image detection" (2023) [40] — 이 논문이 직접 비교·극복하는 baseline. LaRE²의 모든 개선점이 DIRE 대비 설명되므로 배경 이해에 필수.
- 참고문헌 기반: R. Rombach, A. Blattmann, D. Lorenz, P. Esser, B. Ommer, "High-resolution image synthesis with latent diffusion models" (LDM, CVPR 2022) [32] — LaRE 추출에 사용되는 Stable Diffusion의 기반 논문. Latent space 재구성의 원리 이해에 필수.
- 참고문헌 기반: M. Zhu, H. Chen, Q. Yan, X. Huang, G. Lin, W. Li, Z. Tu, H. Hu, J. Hu, Y. Wang, "GenImage: A million-scale benchmark for detecting AI-generated image" (2023) [47] — 이 논문의 실험 전체가 기반하는 벤치마크. 8개 생성기의 구성과 평가 프로토콜 이해에 필요.
- 자유 추천(검증 필요): DIRE 이후 diffusion 생성 이미지 탐지의 2025년 최신 후속 연구 — 검색 키워드: `diffusion generated image detection reconstruction error 2025 generalization`
