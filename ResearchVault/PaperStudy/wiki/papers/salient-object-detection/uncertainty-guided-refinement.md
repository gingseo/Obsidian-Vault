---
title: "Uncertainty-Guided Refinement for Fine-Grained Salient Object Detection"
authors: [Yao Yuan, Pan Gao, Qun Dai, Jie Qin, Wei Xiang]
year: 2025
venue: "IEEE TIP"
jcr_quartile: null
task: [salient-object-detection]
direction: [novel-approach, improvement]
tags: [paper, salient-object-detection, uncertainty, attention, fine-grained-prediction, dynamic-inference]
status: read
user_read: false
added: 2026-04-16
source: "raw/salient-object-detection/2025_TIP_Uncertainty-Guided-Refinement.pdf"
created: 2026-08-04
---

# 한 줄 요약
<mark style="background: #FFF3A3A6;">경계 가이던스(boundary guidance) 대신 예측 saliency map 자체에서 결정론적으로 유도한 불확실성 맵으로 attention을 마스킹해, 저채도(unsaturated)·아티팩트 영역을 반복적으로 정제하는 UGRAN(Uncertainty Guided Refinement Attention Network)을 제안한 논문.</mark>


# 문제 정의

### 기존 방법의 한계
- **저수준 feature 집계 기반 방법의 한계**:
  DSS·NLDF류처럼 저수준(low-level) feature를 끌어와 경계 근처 fine-grained 예측을 보강하는 방법은, 저수준 feature 자체에 노이즈와 non-salient 객체 정보가 많이 섞여 있어 실제로 경계 예측에 유의미하게 기여하는 정보가 적다.
- **Boundary guidance 기반 방법의 한계**:
  EGNet[32], PoolNet[33], F3Net[34], VST[28] 등은 경계 정보로 fine-grained 예측을 가이드하지만 (1) non-salient 객체에서 온 경계 정보가 오히려 모델을 혼란시킬 수 있고(특히 복잡한 배경), (2) 고정된 prior 지식에 의존해 모델이 실제로 어디를 저채도로 예측하는지에 적응적으로 반응하지 못하며, (3) Transformer의 전역 모델링 능력으로 localization 자체는 이미 잘 풀린 문제가 되어 boundary guidance의 한계효용이 작다.

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Feature enhancement/aggregation 계열**
- PoolNet[33]: FPN 구조로 고수준 의미 정보를 저수준 feature map에 전달 — top-down 신호 희석 문제는 완화하나 스케일 간 일관성 통합은 소홀.
- MiNet[37]: 인접 레벨 feature끼리 상호작용 — 업/다운샘플링 필요.
- ICON[38]: 서로 다른 모양의 convolution kernel을 섞어 feature 다양성 확보.
- BBRF[30]: 다양한 수용영역의 switch-path 디코더로 크고 작은 객체를 함께 처리.

**갈래 2 — Boundary guidance 계열**
- VST[28]: 멀티태스크 디코더로 경계 정보를 localization에 활용.
- EGNet[32]: salient object feature와 boundary feature를 별도 추출해 상호 보완.
- PoolNet[33]: 경계 검출과의 joint training.
- F3Net[34]: 경계 픽셀에 더 큰 손실 가중치.
- Amulet[10]: 경계 정보에서 예측 결과로의 short connection.
- 이 갈래는 "기존 방법의 한계"에서 서술한 세 가지 구조적 결함(non-salient 노이즈, 고정 prior, 이미 해결된 localization에 대한 낮은 한계효용)을 공유.

**갈래 3 — Uncertainty guidance 계열 (아직 충분히 탐구되지 않음)**
- UCF[41]: 불확실한 convolutional feature 학습으로 견고성 확보.
- ISPRN[42]: SOD에 불확실성 맵 생성을 최초 도입, 전경/배경과 섞어 문맥 정보로 활용.
- RCSBN[43]: 불확실한 픽셀에 더 큰 손실 가중치(F3Net과 유사).

**갭**: <mark style="background: #FFF3A3A6;">Uncertainty guidance 갈래는 존재하지만 대부분 불확실성을 문맥 정보로 "섞거나"(ISPRN) 손실 가중치로 "반영"(RCSBN)하는 암묵적 방식에 머물러 있었다. 이 논문은 불확실성 맵을 attention 마스크로 직접 사용해 모델이 불확실 영역에만 명시적으로(explicit) 집중하도록 강제하는 첫 시도이며, boundary supervision 없이도 fine-grained 정제가 가능함을 보인다.</mark>

### 이 논문이 풀고자 하는 문제
1. 멀티레벨 feature를 집계할 때, 서로 다른 공간 스케일에 걸친 salient 정보를 일관성 있게 통합하는 문제 (localization 정확도 개선)
2. 예측된 saliency map에 남는 저채도(undersaturated) 영역·그림자·아티팩트를, 고정된 boundary prior가 아니라 현재 예측에 적응적인(adaptive) 방식으로 줄이는 문제
3. 위 정제를 큰 공간 스케일에서도 감당 가능한 계산 비용으로 수행하는 문제

# 제안 방법

<mark style="background: #FFF3A3A6;">핵심 아이디어: 멀티레벨 feature를 두 단계(MIA→SSCA)로 상호작용·통합해 salient 정보를 정교화한 뒤, 매 정제 단계마다 현재 saliency 예측 자체로부터 "0.5(결정 경계)에 가까울수록 불확실하다"는 결정론적 공식으로 불확실성 맵을 만들고, 이를 attention의 마스크로 사용해 저수준 feature와의 상호작용을 불확실 영역에만 강제로 집중시킨다. 계산량 문제는 불확실 영역 비율에 따라 윈도우를 재귀적으로 나누거나 멈추는 Adaptive Dynamic Partition(ADP)으로 해결한다.</mark>

### ① Multilevel Interaction Attention (MIA)
- 백본(ResNet/Res2Net/SwinTransformer)의 멀티레벨 feature F0~F4 각각을 channel attention으로 먼저 정제(F̂i).
- F̂i를 Query, 한 단계 상위 레벨 F(i+1)을 Key/Value로 하는 cross-attention 적용 — 상위 레벨의 전역 salient 표현으로 하위 레벨의 non-salient 노이즈를 줄임.
- 기존 feature enhancement 방법(ASPP, PSP, RFB, AIM)과 달리 업/다운샘플링 없이 그대로 attention에 사용해 정보 손실 회피.

> [!example]- 구현 디테일
> ```
> F̂i = χ(Fi ⊙ ψ(GeLU(ψ(GAP(Fi)))))
> Q = ψ(F̂i), K = ψ(Fi+1), V = ψ(Fi+1)
> F_i^I = BN(χ(F̂i + Attention(Q, K, V)))
> ```

<mark style="background: #FFF9D6A6;">저수준 feature를 그 자체로만 쓰거나 단순히 상위 레벨과 더하던 기존 방식과 달리, MIA는 이미 정제된 상위 레벨의 전역 salient 표현을 K/V로 삼아 저수준 feature가 "어디를 봐야 하는지"를 attention으로 직접 알려주므로 노이즈가 걸러진 채로 활용된다.</mark>

### ② Scale Spatial-Consistent Attention (SSCA)
- MIA로 정제된 feature들을 채널 축소(2C→C)로 합친 뒤, r×r strided convolution(r=2^(3-i))으로 저해상도 K/V를 만들어 self-attention 수행.
- 각 SSCA 출력에서 1×1 conv로 saliency 예측을 뽑아 multilevel supervision에 사용.

> [!example]- 구현 디테일
> ```
> F̂i^C = BN(χ_{r×r,r}(Fi^C))
> Q = ψ(Fi^C), K = ψ(F̂i^C), V = ψ(F̂i^C)
> F̂i^S = Fi^C + Attention(Q, K, V)
> Fi^S = BN(χ(F̂i^S + χ(GeLU(χ(F̂i^S)))))
> ```
> r은 레벨에 따라 F3와 같은 해상도가 되도록 설정.

<mark style="background: #FFF9D6A6;">단순 self-attention(VST 방식)은 스케일 간 salient 정보 불일치를 무시하고 비용도 크다. SSCA는 저해상도 K/V로 전역 salient 표현을 저비용으로 얻어 스케일 간 통합을 겨냥한 설계이며("풀고자 하는 문제 1"), local detail 복원은 다음 단계인 URA로 넘겨 자신은 localization에만 집중한다.</mark>

### ③ Uncertainty Refinement Attention (URA)
- 이전 단계 saliency 예측 S로부터 불확실성 맵을 결정론적으로 생성: 예측값이 0/1에 가까우면 확실, 0.5에 가까우면 불확실.
- 불확실성 맵을 이진 마스크로 변환해 attention score에 더하는 masked attention 적용.
- 정제 대상 feature(Query)와 저수준 feature F0(Key/Value)로 불확실 영역에서만 attention이 유효 — local semantic 정보를 불확실 영역에만 선택적으로 채움.
- 이 refinement를 3회 연속 적용. 학습 시 feature map 크기 고정, 추론 시에는 attention의 공간 스케일 불변성을 이용해 점진적으로 업샘플링하며 재적용해 원본 해상도까지 정제.

> [!example]- 구현 디테일
> ```
> Û = t − |S − t|          (t = 0.5)
> U = Gaussian(k=7, σ=1)(Û)
> M(x,y) = 0        if U(x,y) > 0.01
>        = −∞       otherwise
> Q = ψ(F_i^R), K = ψ(F0), V = ψ(F0)
> F_{i+1}^R = BN(χ(F_i^R + MaskAttention(M, Q, K, V)))
> ```
> Mask2Former[52]의 mask attention과 형태는 비슷하지만, 카테고리별 마스크 예측용 feature extraction이 아니라 반복적으로 예측을 다듬는 post-processing 성격.

<mark style="background: #FFF9D6A6;">Boundary guidance는 훈련·추론 내내 고정된 prior만 참조해 모델이 실제로 어디를 저채도로 예측하는지 반영 못한다는 것이 문제 정의의 핵심이었다. URA의 불확실성 맵은 매 단계마다 "현재" 예측 S로부터 다시 계산되어 모델의 실시간 예측 상태에 맞춰 적응적으로 바뀐다. 또한 attention을 아예 마스킹하는 것은 F3Net/RCSBN류의 손실 가중치보다 훨씬 명시적(explicit)이며, ablation(Table VII)에서 boundary guidance 대비 우위로 검증된다.</mark>

### ④ Adaptive Dynamic Partition (ADP)
- 불확실 영역은 평균적으로 전체 픽셀의 2~5%에 불과 — 전역 attention은 이 작은 비율을 위해 매번 전체 이미지 크기 연산을 하는 셈이라 비효율적.
- 윈도우 단위로 불확실 영역 비율 p를 계산해, p < p_t(=0.2)이면 재귀적으로 더 세분화, p ≥ p_t(blur한 윈도우)이면 분할 중단.
- p_t=0이면 전역 attention, p_t=1이면 Swin[40] 스타일 고정 window attention으로 퇴화하는 일반화된 설계.

> [!example]- 구현 디테일
> 최소 분할 크기는 입력 이미지의 1/32 (Algorithm 1).

<mark style="background: #FFF9D6A6;">"풀고자 하는 문제 3"(큰 공간 스케일에서의 계산 비용)을 정면으로 겨냥한다. 확실한(sharp) 영역은 잘게 쪼개 계산량을 아끼고, 불확실한(blur) 영역은 분할을 멈춰 문맥을 보존한다 — 균일하게 잘게 쪼개면(Table VIII) 불확실 영역이 파편화되어 정제 효과가 떨어지는데, ADP는 그 "덩어리"를 보존해 이를 피한다.</mark>

> [!example]- 손실 함수 및 하이퍼파라미터
> ```
> L_tot = Σ_{i=1}^3 (L_bce(S_i,G) + L_iou(S_i,G))
>       + Σ_{i=1}^3 (L_bce(R_i,G) + L_iou(R_i,G))
>       + Σ_{i=1}^2 L_sc(S_i, Ŝ_{i+1})
> ```
> S는 SSCA 예측, R은 URA 정제 예측, L_sc는 SSCA 단계 간 예측 일관성을 강제하는 spatial consistency loss(gradient stop 적용). F3Net류의 픽셀별 가중 손실(weighted BCE/IoU, TRACER의 L_api 포함)도 시도했으나 오히려 성능 저하(Table IX) — "URA가 이미 충분한 명시적 가이던스를 주므로 손실 가중치까지 더하면 복잡한 영역에 과도하게 치우친다"고 해석.
>
> 주요 하이퍼파라미터: threshold t=0.5(고정), Gaussian smoothing k=7·σ=1, 마스크 임계값 0.01, p_t=0.2, 최소 partition 크기=1/32, URA 반복 횟수 3회, 채널 C=64.
>
> 학습 설정: DUTS-TR(10,553장), 384×384 입력, batch size 8, 60 epoch, Adam(lr 1e-4, 백본은 1/10), poly decay + 12,000 iteration warm-up, RTX 3090.

# 실험 결과

- **벤치마크**: DUT-OMRON·DUTS-TE·ECSSD·HKU-IS·PASCAL-S·SOD 6개 + SOC(복잡 실세계 장면, 9개 속성 서브셋).
- **비교 대상**: CNN 기반 SOTA 26편(RAS, NLDF, EGNet, F3Net, MiNet, ICON, RCSB, BBRF 등) + Transformer 기반 SOTA 5편(VST, ISPRNet, SRformer 등).

### 핵심 결과 (Table I, DUTS-TE)

| 지표 | ICON-R(이전 최고, ResNet) | Ours-S (SwinTransformer) |
|---|---|---|
| MAE↓ | .032 | .022 |
| Fβ^w↑ | .819 | .931 |

> [!note]- 세부 결과 및 Ablation
> #### Baseline 대비 개선 (Table I, 전체 벤치마크)
> | 벤치마크 | 지표 | ICON-R(ResNet) | Ours-R (ResNet50) | Ours-S (Swin) |
> |---|---|---|---|---|
> | DUTS-TE | MAE↓ | .032 | .033 | .022 |
> | DUTS-TE | Fβ^w↑ | .819 | .906 | .931 |
> | ECSSD | MAE↓ | .029 | .025 | .02 |
> | ECSSD | Fβ^w↑ | .89 | .922 | .946 |
> | HKU-IS | MAE↓ | .025 | .025 | .02 |
> | HKU-IS | Fβ^w↑ | .875 | .906 | .932 |
> | PASCAL-S | MAE↓ | .055 | .055 | .046 |
> | PASCAL-S | Fβ^w↑ | .82 | .84 | .863 |
> | SOD | MAE↓ | .084 | .084 | .075 |
> | SOD | Fβ^w↑ | .78 | .776 | .818 |
> | DUT-O | MAE↓ | .057 | .061 | .045 |
> | DUT-O | Fβ^w↑ | .74 | .776 | .802 |
>
> - 6개 데이터셋 모두에서 Fβ^w(shadow·저채도에 특히 민감한 지표) 기준 이전 SOTA를 앞섬. Ours-R은 MAE가 일부 데이터셋에서 근소하게 뒤지지만 Fβ^w 개선폭이 큼.
> - SwinTransformer backbone(Ours-S)이 전반적으로 우수하며, 다수의 attention 모듈을 포함하고도 44 fps 실시간 추론 속도 유지.
> - **SOC 데이터셋**(Table II, CNN 기반 SOTA 17편과 비교): 전반적으로 최고 성능, 특히 motion blur(MB) 속성에서 개선폭이 큼(blur 처리 능력 직접 검증). 반대로 shape complexity(SC) 속성에서는 상대적으로 부진 — ADP의 영역 분할이 형태가 복잡한 객체에서 파편화(fragmentation)를 일으키기 때문으로 저자들은 추정.
>
> #### 모듈별 기여도 (Table III, ResNet50 backbone)
> | 구성 | DUTS MAE↓ | DUTS Fβ^w↑ | ECSSD MAE↓ | ECSSD Fβ^w↑ | HKU-IS MAE↓ | HKU-IS Fβ^w↑ |
> |---|---|---|---|---|---|---|
> | Baseline(UNet류) | .046 | .776 | .047 | .871 | .037 | .866 |
> | +MIA | .041 | .812 | .037 | .898 | .032 | .891 |
> | +MIA+SSCA | .038 | .844 | .034 | .909 | .031 | .897 |
> | +MIA+SSCA+URA | .033 | .865 | .029 | .922 | .025 | .917 |
>
> #### Uncertainty guidance vs boundary guidance (Table VII, MIA+SSCA 기준 URA 입력만 교체)
> | Guidance | DUTS Fβ^w | ECSSD Fβ^w | HKU-IS Fβ^w |
> |---|---|---|---|
> | None(마스킹 없음) | .862 | .921 | .912 |
> | Boundary(EGNet 방식 경계 추출) | .863 | .920 | .914 |
> | Uncertainty(제안) | .865 | .922 | .917 |
>
> Boundary guidance는 개선이 미미하거나 일부 지표(ECSSD)는 오히려 하락. Uncertainty guidance만 일관되게 개선 — 선행 연구 갭에서 제기한 핵심 가설을 직접 검증.
>
> #### ADP 효과 (Table VIII, DUTS)
> - 균일한 small partition: MAE .04 / Fβ^w .857
> - Random partition(50% 확률): .038 / .854
> - 제안 ADP(p_t=0.2): .033 / .865 — 불확실 영역의 과도한 파편화를 막기 때문으로 해석
> - p_t=0.1: .035/.857, p_t=0.4: .037/.857 — 둘 다 p_t=0.2보다 열세
>
> #### 세부 발견
> - MIA는 기존 feature enhancement 기법(DFA, ASPP, PSP, RFB, AIM)보다 전 지표에서 우수(Table IV) — 업/다운샘플링 없는 상호작용 attention이 정보 손실을 줄이기 때문.
> - MIA의 interaction 방향은 "상위 레벨→하위 레벨"이 "하위 레벨→상위 레벨"보다 우수하고(Table V), 인접 레벨끼리 상호작용하는 것이 먼 레벨보다 낫다.
> - SSCA는 기존 feature aggregation 기법(SIM, AFM)보다 우수(Table VI) — attention 기반 전역 모델링이 convolution의 제한된 수용영역보다 유리.
> - 가중 손실(weighted BCE/IoU/L_api)을 추가하면 오히려 성능 저하(Table IX) — URA가 이미 충분한 명시적 가이던스를 제공하기 때문으로 해석.

# Discussion

### 이 아이디어의 잠재적 부작용
- **"확신도"가 실제 정확도와 어긋날 위험**:
  불확실성을 예측값이 0.5에 가까운 정도로만 정의하므로, 모델이 극단값(0 또는 1)으로 강하게 예측했지만 틀린 경우(확신에 찬 오답)는 "불확실"로 잡히지 않아 URA의 정제 대상에서 원천적으로 제외된다. <mark style="background: #FF5582A6;">논문이 명시한 한계와 정확히 일치한다 — "불확실성 맵은 saliency map 자체에서 파생되므로 salient object의 잘못된 localization을 교정할 수 없다"(Limitations 절). 저자들도 인지하고 있으나 해결책은 제시하지 않았다.</mark>
- **반복 정제가 자기 자신의 오류를 재생산할 위험**:
  URA를 3회 연속 적용하고 추론 시 점진적 업샘플링까지 반복하는 구조라, 초기 단계의 오류(위 항목의 "확신에 찬 오답")가 다음 단계에서도 "확실 영역"으로 분류되어 그대로 유지·증폭될 가능성이 있다. <mark style="background: #FF5582A6;">논문은 이 문제를 직접 실험으로 검증하지 않았다</mark> — ablation(Table III·VIII)은 반복 정제의 순효과만 보여줄 뿐, "특정 실패 사례가 반복될수록 악화되는지"는 별도 분석되지 않는다.

### 한계
- <mark style="background: #FF5582A6;">불확실성 맵이 현재 saliency 예측 자체에서 파생되므로, salient object의 위치 파악(localization) 자체가 잘못된 경우는 교정할 수 없다(저자 명시).</mark>
- <mark style="background: #FF5582A6;">윈도우 분할(ADP)의 유연성에 한계가 있어 형태가 복잡한 객체의 경우 불확실 영역이 조각날 수 있다</mark> — SOC의 shape complexity(SC) 서브셋 성능 저하와 저자들이 직접 연결지어 설명.
- t=0.5, k=7/σ=1, p_t=0.2 같은 하이퍼파라미터는 이 논문의 실험 세팅에 맞춰진 값 — 전혀 다른 도메인(예: 클래스 수가 많은 semantic segmentation)에서는 재검증 필요(저자 명시는 아님, 방법론 구조상 자연스러운 추정).

### 생각할 점
- <mark style="background: #A6E3A1A6;">불확실성을 "예측값의 0.5로부터의 거리"로 정의하는 방식은 단순·학습 불필요라는 장점이 있지만 "예측 신뢰도 = 예측 정확도"라는 가정에 기대고 있다 — calibration이 안 된 모델일수록 이 가정이 깨진다. Temperature scaling으로 먼저 calibration을 개선한 뒤 같은 공식을 적용하면 "확신에 찬 오답" 문제를 부분적으로 완화할 수 있을지 검증해볼 만하다.</mark>
- 저자들이 언급하듯 이 정제 접근은 다른 이진 이미지 분할 task에도 이식 가능해 보이나 논문에서 직접 검증하지는 않았다 — 다중 클래스로 확장하려면 "0.5" 임계값을 confidence 분포로 어떻게 일반화할지가 관건.

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">[[gaussian-box-uncertainty-modeling]]과의 대비가 흥미롭다 — 이 논문의 불확실성은 결정론적·비학습(별도 파라미터·loss 없이 예측값 자체의 함수)이고 픽셀 단위인 반면, gaussian-box-uncertainty-modeling은 학습된 분포 파라미터(σ)와 KL loss를 쓰고 박스 단위다. SOD에도 "학습된" 불확실성(saliency map을 Gaussian/Beta 분포로 모델링해 픽셀별 σ를 직접 예측)을 도입하면 "확신에 찬 오답을 못 잡는" 한계를 완화할 수 있을지 검증해볼 가치가 있다 — 학습된 불확실성은 모델의 실제 오차 패턴에서 신호를 얻으므로 다른 종류의 불확실성을 포착할 가능성이 있다.</mark>
- <mark style="background: #A6E3A1A6;">[[uncertainty-masked-refinement-attention]]의 ADP(불확실 영역 비율 기반 재귀적 윈도우 분할)는 SOD 외에 dense prediction 전반(depth estimation, anomaly detection의 픽셀 단위 이상 스코어 맵 등)에서 "불확실/이상 영역에만 비용을 쓰는" 동적 추론 패턴으로 재사용할 수 있어 보인다.</mark>

# 관련 개념
- [[uncertainty-masked-refinement-attention]] — 이 논문이 제안하는 핵심 기법(URA). 예측 saliency map에서 결정론적으로 생성한 불확실성 맵으로 attention을 마스킹해 반복적으로 저채도 영역을 정제하는 방식.
- [[gaussian-box-uncertainty-modeling]] — 마찬가지로 "불확실성"을 명시적으로 모델링해 학습을 가이드하지만, 대상이 바운딩 박스 좌표 회귀(학습되는 σ + KL divergence)인 반면 이 논문은 픽셀 단위 saliency 값 자체(0.5로부터의 거리)로부터 결정론적으로 불확실성을 유도하고 attention 마스크로 즉시 사용한다는 점에서 메커니즘이 명확히 다르다. 학습 가능한 분포 파라미터도, 별도의 uncertainty loss도 없다.

# 관련 문서
(현재 위키에 직접 비교할 다른 salient-object-detection 논문 노트가 없어 comparison 문서는 아직 만들지 않음. RFLA·CFINet 등은 이 논문과 무관한 task이므로 링크하지 않음.)

# 읽어볼 만한 논문
- 참고문헌 기반: J. Zhao, J.-J. Liu, D.-P. Fan, Y. Cao, J. Yang, and M.-M. Cheng, "EGNet: Edge guidance network for salient object detection" [32] (ICCV 2019) — 이 논문이 대비축으로 삼는 boundary guidance 계열의 대표 논문이자, URA의 ablation(Table VII "Boundary" 설정)에서 경계 추출 방식으로 직접 채택된 baseline. Boundary guidance의 실제 구현과 한계를 이해하는 데 필수적.
- 참고문헌 기반: T. Kim, K. Kim, J. Lee, D. Cha, J. Lee, and D. Kim, "Revisiting image pyramid structure for high resolution salient object detection" (ISPRN) [42] (ACCV 2022) — SOD에 불확실성 맵 생성을 최초로 도입한 논문. 이 논문(UGRAN)이 "불확실성을 문맥 정보로 섞는" 기존 방식과 "attention 마스크로 명시적으로 쓰는" 자신의 방식을 대비할 때 기준으로 삼는 선행 연구라 배경 이해에 도움.
- 참고문헌 기반: B. Cheng, I. Misra, A. G. Schwing, A. Kirillov, and R. Girdhar, "Masked-attention mask transformer for universal image segmentation" (Mask2Former) [52] (CVPR 2022) — URA의 masked attention 계산식(식 5)이 형태적으로 참조한 원조 기법. Mask2Former는 카테고리 마스크 예측에, UGRAN은 반복적 post-processing 정제에 쓴다는 차이를 이해하려면 원조 메커니즘을 먼저 볼 필요가 있음.
- 자유 추천(검증 필요): saliency/segmentation 모델의 confidence calibration(예: temperature scaling)을 다룬 연구 — 검색 키워드: `confidence calibration semantic segmentation temperature scaling`. 위 Discussion에서 제기한 "확신에 찬 오답은 불확실성 맵이 못 잡는다"는 문제를 완화할 수 있는지 검증할 때 참고할 만한 방향.
