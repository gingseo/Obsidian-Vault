---
title: "Feature Information Driven Position Gaussian Distribution Estimation for Tiny Object Detection"
authors: [Jinghao Bian, Mingtao Feng, Weisheng Dong, Fangfang Wu, Jianqiao Luo, Yaonan Wang, Guangming Shi]
year: 2025
venue: "CVPR"
jcr_quartile: "Q1"
task: [small-object-detection]
direction: [novel-approach, improvement]
tags: [paper, small-object-detection, feature-enhancement, information-entropy, gaussian-mixture, plug-and-play]
status: in-progress
added: 2026-07-01
source: "PaperStudy/Raw/Small_Object_Detection/2025_CVPR_Feature-Information-Driven-Position-Gaussian.pdf"
created: 2026-08-04
---

#paper #small-object-detection #feature-enhancement #information-entropy #gaussian-mixture #plug-and-play

# 한 줄 요약
<mark style="background: #FFF3A3A6;">Tiny object의 극도로 약한 feature 표현을 "픽셀 단위 정보량"이라는 정보이론적 관점에서 비지도로 찾아낸 information map과, 객체 위치·크기 기반 Gaussian Mixture로 지도학습되는 position distribution map, 이 두 가지로 동시에 강화하는 plug-and-play feature enhancement 모듈.</mark>


# 문제 정의

### 기존 방법의 한계
- **Scale-aware feature fusion 계열(FPN, BiFPN, DetectoRS 등)**:
  서로 다른 깊이의 feature를 융합해 spatial-semantic gap을 메우지만, tiny object는 반복적인 다운샘플링으로 애초에 activation 자체가 거의 사라진 상태이므로 단순 융합만으로는 회복이 안 된다.
- **Attention 기반 방법(SCRDet, AFF-SSD, KB-RANN 등)**:
  heuristic한 attention map 생성에 의존하는데, tiny object는 픽셀 수가 워낙 적어 local patch 안에서 background가 attention map을 지배해버려 신뢰도가 낮다.
- **Mimic learning 계열(Perceptual GAN, MT-GAN 등)**:
  큰 인스턴스의 고품질 feature로 작은 인스턴스 표현을 보완하려 하지만, 이 역시 "정보 손실이 일어난 영역이 어디인지"를 직접 식별하지는 않는다.

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Sample-oriented / label assignment 전략**
- Kisantal et al. [17]: 작은 객체 이미지 oversample + copy-paste 증강 — feature 표현 자체의 정보 손실은 안 건드림.
- NWD-RKA [47]: IoU 대신 Normalized Wasserstein Distance와 ranking 기반 assignment — 동일.
- RFLA [48]: Gaussian 수용영역-GT 유사도로 positive sample 부족 완화 — 동일.

**갈래 2 — Scale-aware / attention 기반 feature enhancement**
- FPN [25], BiFPN [38], DetectoRS [31], Gong et al. [12]: multi-scale fusion으로 tiny object 표현 개선.
- SCRDet [50], AFF-SSD [29], KB-RANN [52]: attention으로 중요 영역 강조.
- SR-TOD [6]: 복원 이미지와 원본의 difference map으로 information loss 영역 탐색 — 문제의식은 가장 가깝지만 복원 이미지 품질에 의존하고, 복원 과정의 다운샘플링이 difference map 정보를 다시 훼손.

**갭**: <mark style="background: #FFF3A3A6;">선행 연구들은 "정보 손실이 일어난 영역"을 픽셀 단위 정보량(amount of information) 관점에서 직접 정의·측정한 적이 없다 — SR-TOD조차 간접적인 재구성 difference에 의존한다. 이 논문은 Shannon entropy 기반 encoding cost로 feature map 레벨에서 직접 정보량을 추정하는 최초 시도라고 주장한다.</mark>

### 이 논문이 풀고자 하는 문제
1. Tiny object의 feature가 왜/어디서 약해지는지를 정보량 관점에서 정량적으로 식별하는 것 (복원 이미지 같은 간접 신호 없이)
2. 식별된 정보 손실 영역 중에서도 tiny object 위치에 더 집중하도록 유도하는 것 (일반 크기 객체와의 구분)
3. 위 두 가지를 특정 detector 구조에 종속되지 않는 plug-and-play 모듈로 구현하는 것

# 제안 방법

<mark style="background: #FFF3A3A6;">핵심 아이디어: (1) Shannon entropy 기반 encoding cost를 최소화해 픽셀별 정보량을 비지도로 추정하는 Pixels Feature Information Modeling(PFIM)으로 information map σ를 얻고, (2) 객체의 위치·크기로부터 만든 Gaussian Mixture 분포(Position Gaussian Distribution Map)를 σ를 prior로 삼아 지도학습으로 예측하는 Position Gaussian Distribution Prediction(PGDP)으로 position map을 얻어, 두 맵으로 FPN 최하위 레벨 feature P2를 동시에 강화한다.</mark>

### ① Pixels Feature Information Modeling (PFIM) — information map σ
- Feature y(=P2)를 가산 uniform noise로 quantization해 미분가능한 이산 feature를 얻고, 각 픽셀을 평균 μ·표준편차 σ의 fully factorized Gaussian으로 모델링.
- Encoding cost(= -log 확률, bits)의 합을 Information Entropy loss `L_IE`로 정의하고 최소화하도록 μ, σ를 CNN으로 예측.
- Salient/tiny 영역일수록 발생확률이 낮아 encoding cost(정보량)가 커지고, 예측된 σ가 이 정보량과 양의 상관을 가지므로 σ를 "information map"으로 채택.
- 채널 평균한 σ로 `y1 = y ⊗ (1+σ)` 형태의 1차 feature 강화 수행 (1을 더해 σ≈0 영역의 맥락 정보 보존).

> [!example]- 구현 디테일
> ```
> p(ŷ|μ,σ) = ∏_i (N(μ_i, σ_i²) * U(-1/2,1/2))(ŷ_i)
> R_ŷi = -log2 p(ŷ_i|μ_i,σ_i)          (encoding cost, bits)
> L_IE = Σ_i R_ŷi                       (Information Entropy loss)
> ```

<mark style="background: #FFF9D6A6;">왜 문제 1(정보 손실 영역의 직접 식별)을 해결하는가: 기존 attention 기반 방법은 heuristic한 importance weight를 만들 뿐 "얼마나 많은 정보가 손실됐는가"를 정량적 근거로 갖지 않았고, SR-TOD의 difference map은 복원 이미지 품질에 의존하는 간접 신호였다. PFIM은 encoding cost(=정보량)라는 정보이론적으로 정의된 양을 직접 최소화해서 얻은 σ를 쓰므로, 별도의 복원 네트워크나 heuristic 없이 feature map 레벨에서 바로 정보량을 측정한다.</mark>

### ② Position Gaussian Distribution Prediction (PGDP) — position distribution map
- 각 GT box i를 중심 `(x_i,y_i)`, 공분산 `diag((w_i/α_i)², (h_i/α_i)²)`인 2D Gaussian으로 모델링.
- 스케일링 factor α는 AI-TOD 크기 정의에 따라 very tiny(2–8px)=4, tiny(8–16px)=6, small(16–32px)=8, general=10 — 객체가 작을수록 α가 작아 분포가 더 뾰족하고 값이 크게 만듦.
- N개 인스턴스의 Mixture를 합산하고 threshold 기반 후처리로 foreground-background 대비를 강화해 GT map `M_GT` 생성.
- Prediction 단계는 information map σ를 prior로 각 FPN 레벨 입력에 더해(`P4+σ/4, P3+σ/2, P2+σ`), skip connection 있는 multi-scale conv/deconv 네트워크로 `M_pd2, M_pd3, M_pd4` 예측, weighted MSE(`L_pred`, fg:bg=10:0.1)로 지도학습.
- `M_pd2`로 `y2 = y ⊗ (1+M_pd2)` 2차 강화 수행.

> [!example]- 구현 디테일
> ```
> f(p) = (1/N) Σ_i N(p | μ_i^box, Σ_i^box)
> M_GT = (Sign(N·f(p) - th) + 1) × 0.25 + N·f(p)
> ```

<mark style="background: #FFF9D6A6;">왜 문제 2(tiny object에 대한 추가 집중)를 해결하는가: information map σ는 정보량이 큰 영역 전반(일반 크기 객체 포함)을 균등하게 강조할 뿐 tiny object를 특별 취급하지 않는다. Position Gaussian map은 크기별로 다른 α를 써서 "작을수록 분포를 더 뾰족하고 값을 크게" 만들도록 설계했으므로, 같은 foreground라도 tiny object가 더 큰 값을 갖는다. σ를 prior로 넣어 예측을 유도하면 두 맵이 서로 강화하는 상호작용이 생겨, 어느 한쪽만으로는 얻기 힘든 tiny-object 특화 신호를 만든다.</mark>

### ③ 최종 융합 및 손실 함수
- y1, y2는 각각 CBAM(Convolutional Block Attention Module)을 거쳐 element-wise 덧셈으로 합쳐져 최종 강화 feature P2'를 만들고, 기존 P2를 대체해 detection head로 전달.
- FPN 기반 어떤 detector에도 꽂을 수 있는 plug-and-play 모듈로 설계.

> [!example]- 구현 디테일
> ```
> L = L_det + λ1·L_IE + λ2·L_pred     (λ1=0.01, λ2=1.0)
> ```

# 실험 결과

### 핵심 결과

| 벤치마크 | 지표(구성) | Before | After |
|---|---|---|---|
| AI-TOD | AP (DetectoRS) | 14.6 | 24.3 (+9.7, 논문 내 최대 gain) |
| VisDrone2019 | AP (RFLA, 전체 SOTA 중 최고) | 27.2 | 29.0 |

> [!note]- 세부 결과 및 Ablation
> #### 설정
> - **데이터셋**: VisDrone2019(드론뷰 10class, 10,209장), AI-TOD(항공 8class, 28,036장, 평균 인스턴스 크기 12.8px), AI-TODv2(정제판, 평균 12.7px)
> - **구현**: MMDetection, ResNet50-FPN, RTX 4090 1장, SGD(momentum 0.9, wd 0.0001), batch 2, 12 epoch, lr 0.005(8/11 epoch decay)
> - **지표**: AP, AP0.5, AP0.75, AP_vt(very tiny), AP_t(tiny), AP_s(small)
>
> #### Baseline 대비 개선 (plug-and-play 적용, AP 기준)
> | 벤치마크 | Baseline | Before | After | Gain | 비고 |
> |---|---|---|---|---|---|
> | VisDrone2019 | Faster R-CNN | 23.9 | 26.8 | +2.9 | AP_t 6.5→12.3(+5.8, 최대 gain) |
> | VisDrone2019 | Cascade R-CNN | 25.2 | 28.1 | +2.9 | |
> | VisDrone2019 | DetectoRS | 26.3 | 28.3 | +2.0 | AP_vt 0.1→3.5(+3.4) |
> | VisDrone2019 | RFLA | 27.2 | 29.0 | +1.8 | AP_vt 7.4, 2위 대비 +2.0 |
> | AI-TOD | Faster R-CNN | 11.7 | 20.6 | +8.9 | |
> | AI-TOD | Cascade R-CNN | 14.0 | 22.6 | +8.6 | |
> | AI-TOD | DetectoRS | 14.6 | 24.3 | +9.7 | AP0.5 31.8→54.4(+22.6) |
> | AI-TOD | RFLA | 21.7 | 22.6 | +0.9 | AP_vt는 8.3→8.2로 소폭 하락(-0.1) |
> | AI-TODv2 | Cascade R-CNN | 14.9 | 23.7 | +8.8 | |
> | AI-TODv2 | DetectoRS | 16.1 | 25.5 | +9.4 | AP0.5 35.5→58.2(+22.7), 전체 최고 |
> | AI-TODv2 | RFLA | 22.8 | 23.9 | +1.1 | AP_vt는 7.9→7.3으로 하락(-0.6) |
>
> - VisDrone2019에서 RFLA+ours는 SR-TOD(27.3), Salience DETR(28.4) 등 다른 SOTA 대비 전 지표 최고.
> - SR-TOD와 직접 비교 시 이 논문의 개선폭이 더 크며, 저자는 "difference map은 정보 손실 영역의 일부만 포착하기 때문"이라 설명.
>
> #### Ablation — 모듈 기여도 (VisDrone2019, DetectoRS 기준)
> | 구성 | AP | AP0.5 | AP_vt | AP_t | AP_s |
> |---|---|---|---|---|---|
> | Baseline | 26.3 | 43.9 | 0.1 | 7.5 | 23.3 |
> | + PFIM만 | 28.2 | 48.4 | 3.3 | 12.2 | 26.0 |
> | + PGDP만 | 27.6 | 47.3 | 3.4 | 11.2 | 24.6 |
> | + PFIM+PGDP | 28.3 | 48.5 | 3.5 | 12.6 | 26.1 |
>
> #### Ablation — Position 분포 설계 방식 비교
> | 설정 | AP | AP0.5 | AP_vt | AP_t | AP_s |
> |---|---|---|---|---|---|
> | 고정 α=1 | 27.9 | 47.9 | 3.4 | 11.8 | 25.6 |
> | Binary mask | 27.9 | 47.6 | 3.4 | 11.8 | 25.1 |
> | Self-attention | 27.7 | 47.5 | 3.0 | 11.7 | 25.1 |
> | 제안 방식(크기 비례 α) | 28.3 | 48.5 | 3.5 | 12.6 | 26.1 |
>
> #### 세부 발견
> - **σ 주입 방식**(Table 6): 단순 `⊗σ`는 AP/AP_s 최고지만 AP_vt(2.7)가 크게 떨어짐 — σ≈0 영역의 맥락 정보가 사라지기 때문. Concat은 노이즈 유발(σ는 정보량, feature는 공간·의미 정보로 성격이 달라 단순 결합이 안 맞음). `⊗(1+σ)`가 대체로 안정적이며 논문 채택 방식.
> - **y1, y2 융합 방식**(Table 7): element-wise addition(28.3)이 multiplication(27.5)·concat(27.8)보다 우수 — multiplication은 값을 과도하게 키우거나 줄여 정보 손실, concat은 y1·y2가 이미 유사해 중복만 늘림.
> - Bits-per-pixel(bpp) 분석(Fig. 7): 인스턴스 밀도가 높은 장면일수록 평균 bpp가 증가 — `L_IE`가 실제로 salient/dense 영역에 더 많은 encoding cost를 할당함을 뒷받침.
> - 다수 detector(Faster R-CNN, Cascade R-CNN, DetectoRS, RFLA)에 이식해도 일관된 개선 — plug-and-play 범용성 확인.

# Discussion

### 이 아이디어의 잠재적 부작용
- **정보량이 큰 영역이 항상 tiny object는 아닐 위험**:
  텍스처가 복잡한 배경(나뭇잎, 자갈, 노이즈 영역)도 σ 값이 커질 수 있음. <mark style="background: #FF5582A6;">논문은 Position Gaussian map으로 보완한다고 주장하지만, PGDP 자체가 GT 박스 위치에 지도학습되므로 학습 시 못 본 배경 텍스처에 대한 σ 오탐 가능성은 정량 검증되지 않았다 — 정성적 시각화(Fig. 6)만 제시, false positive 분석 없음.</mark>
- **두 맵의 상호 의존이 만드는 학습 불안정 가능성**:
  σ가 PGDP 예측의 prior로 들어가고 `L_pred` 최적화가 다시 더 나은 σ를 만드는 상호 보강 구조는, 초기 σ 추정이 부정확하면 PGDP도 저품질 prior로 시작하는 악순환 위험을 내포한다. <mark style="background: #FF5582A6;">논문은 초기화·warmup 전략이나 안정성 분석을 제시하지 않는다.</mark>

### 한계
- <mark style="background: #FF5582A6;">RFLA와 결합 시 다른 baseline 대비 gain 폭이 뚜렷이 작고(AI-TOD +0.9, AI-TODv2 +1.1 vs DetectoRS +9.7/+9.4), AP_vt는 AI-TOD·AI-TODv2 둘 다 RFLA 단독보다 하락(-0.1, -0.6)한다.</mark> 왜 RFLA와 시너지가 약한지 구체적 분석 없음.
- <mark style="background: #FF5582A6;">σ·Position Gaussian map 계산에 따른 추가 연산/파라미터 오버헤드(latency, FLOPs, 메모리) 정량 비교가 전혀 없다.</mark> Plug-and-play를 내세우는 논문 특성상 실무 적용 판단에 중요한데 누락.
- P2(FPN 최하위 레벨)에만 강화 적용, 다른 레벨(P3~P5) 확장 효과는 미다룸.
- 세 데이터셋 모두 항공/드론뷰 벤치마크 — 지상 시점 tiny object(SODA-D류)에 대한 검증 없음.

### 생각할 점
- <mark style="background: #A6E3A1A6;">"정보량 기반 saliency"는 detection을 넘어, segmentation의 경계 모호 영역이나 super-resolution의 디테일 복원 영역 탐색에도 적용 가능해 보인다 — 공통적으로 "픽셀이 적은/약한 신호 영역 식별"이 핵심이기 때문.</mark>
- Position Gaussian map의 스케일링 factor α(4,6,8,10)는 AI-TOD 크기 구간에 고정된 하이퍼파라미터 — 데이터셋마다 크기 분포가 다르면 분위수 기반 자동 추정이 더 일반화될 수 있을지 검토해볼 만함.

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">이 논문의 정보량 기반 σ는 [[Unc-SOD]]의 instance-level uncertainty와 접근 축이 다르지만(전자는 feature 강화용 attention prior, 후자는 sampling 기준) 상호 보완 가능성이 있다 — "σ가 작아 특징이 흐릿한 영역의 proposal일수록 sampling 기준을 더 관대하게" 만드는 식으로 결합하면 feature 강화 축과 label assignment 축을 잇는 다리가 될 수 있다.</mark> [[Small_Object_Detection_Approaches]]에서 두 계열이 직교적으로 분류된 것과 맞닿아 있다.
- [[SR-TOD]]와의 직접 비교에서 이 논문이 더 크게 이긴 이유가 "difference map은 정보 손실의 일부만 포착"이라는 서술뿐 — σ와 difference map의 공간적 correlation을 직접 시각화·정량 비교하면 결합 지점을 더 구체적으로 찾을 수 있을 것이다.

# 관련 개념
- [[Position_Gaussian_Saliency_Map]] — 이 논문이 "Position Gaussian Distribution Map"이라는 이름으로 처음 도입한, 객체 위치·크기를 Gaussian Mixture로 인코딩해 feature enhancement의 supervision/attention prior로 쓰는 기법.
- [[Gaussian_Box_Uncertainty_Modeling]] — 같은 "Gaussian으로 객체를 모델링"하는 계열이지만 용도가 다름(박스 회귀 좌표의 예측 불확실성 모델링 vs. 이 논문의 위치 saliency/attention prior). 서로 다른 개념으로 유지.

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]] — feature 강화 계열(정보이론+위치 축)로 분류
- [[SR-TOD]] — 이 논문이 직접 비교 대상으로 삼는 가장 가까운 선행 연구(difference map 기반 정보 손실 탐지)
- [[Unc-SOD]] — 같은 task를 다른 축(label assignment/sampling)에서 개선하는 논문. Discussion 참고

# 읽어볼 만한 논문
- 참고문헌 기반: C. Xu, J. Wang, W. Yang, H. Yu, L. Yu, G.-S. Xia, "RFLA: Gaussian receptive field based label assignment for tiny object detection" [48] (ECCV 2022) — 이 논문의 실험에서 가장 강력한 baseline(RFLA+ours가 VisDrone SOTA)이자, "Gaussian으로 객체를 모델링"하는 아이디어를 label assignment에 적용한 대표 논문. Position Gaussian map과의 개념적 차이(assignment용 vs saliency prior용)를 이해하는 데 직접적으로 도움됨.
- 참고문헌 기반: B. Cao, H. Yao, P. Zhu, Q. Hu, "Visible and clear: Finding tiny objects in difference map" (SR-TOD) [6] (ECCV 2024 / arXiv 2405.11276) — 이 논문이 본문에서 직접 비교·차별화하는 가장 가까운 선행 연구. 이미 [[SR-TOD]]로 위키에 있으므로 재확인 겸 두 접근의 실제 차이를 다시 짚어볼 때 참고.
- 참고문헌 기반: J. Ballé, V. Laparra, E. P. Simoncelli, "Density modeling of images using a generalized normalization transformation" [2] (arXiv 2015) — PFIM의 quantization·fully factorized density modeling·GDN(Generalized Divisive Normalization) 설계가 직접 기반하는 원조 논문(image/learned compression 분야). Information Entropy loss의 수학적 배경을 제대로 이해하려면 필수.
- 참고문헌 기반: D. Minnen, J. Ballé, G. D. Toderici, "Joint autoregressive and hierarchical priors for learned image compression" [30] (NeurIPS 2018) — PFIM에서 "Gaussian density를 unit uniform distribution과 convolve해 실제 marginal distribution에 맞춘다"는 기법의 근거로 직접 인용됨. 학습된 압축(learned compression) 분야의 density modeling 기법이 detection의 feature enhancement에 어떻게 전용됐는지 배경 이해에 도움.
- 자유 추천(검증 필요): entropy/rate-distortion 관점을 다른 dense prediction task(예: segmentation의 경계 영역 saliency)에 적용한 연구 — 검색 키워드: `information entropy loss feature map saliency segmentation learned compression`. Discussion의 "생각할 점"에서 언급한 task 간 이식 가능성을 검증할 때 참고할 만한 방향.
