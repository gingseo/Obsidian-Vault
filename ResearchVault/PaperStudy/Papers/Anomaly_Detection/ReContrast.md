---
title: "ReContrast: Domain-Specific Anomaly Detection via Contrastive Reconstruction"
authors: [Jia Guo, Shuai Lu, Lize Jia, Weihang Zhang, Huiqi Li]
year: 2023
venue: "NeurIPS"
jcr_quartile: "Q1"
task: [anomaly-detection]
direction: [novel-approach, improvement]
tags: [paper, anomaly-detection, contrastive-learning, feature-reconstruction, domain-adaptation, mvtec-ad]
status: in-progress
added: 2026-08-04
source: "PaperStudy/Raw/Anomaly_Detection/2023_NeurIPS_ReContrast.pdf"
created: 2026-08-04
---

#paper #anomaly-detection #contrastive-learning #feature-reconstruction #domain-adaptation #mvtec-ad

# 한 줄 요약
<mark style="background: #FFF3A3A6;">Feature reconstruction 기반 비지도 이상 탐지(UAD)에서 그동안 pattern collapse를 막기 위해 반드시 고정(frozen)해야 했던 ImageNet 사전학습 encoder를, contrastive learning의 세 요소(global cosine distance, stop-gradient, dual-encoder contrastive pair)를 접목해 decoder와 함께 end-to-end로 안전하게 최적화함으로써 도메인 편향 문제를 해결한 방법.</mark>


# 문제 정의

### 기존 방법의 한계
- **Frozen encoder라는 관행**:
  Feature reconstruction(RD4AD 등 [2;3;4])·feature distillation([2;5;6])·feature memory & modeling([10;11;12;13;14]) 계열 모두, ImageNet 등 대규모 자연 이미지로 사전학습된 encoder를 특징 추출기로 그대로 얼려 쓴다.
- **Pattern collapse 위험**:
  Feature reconstruction에서 encoder까지 함께 학습시키면, decoder가 재구성하기 쉬운 무의미한(indiscriminative) 특징으로 encoder가 수렴해버리는 trivial solution — pattern collapse가 발생한다. 이 때문에 encoder freeze가 "당연한 전제"로 굳어졌다.
- **도메인 편향(semantic gap)**:
  자연 이미지 도메인에서 학습된 frozen encoder가 추출하는 특징은, 산업 결함 탐지·의료 영상처럼 자연 이미지와 거리가 먼 target UAD 도메인에는 잘 들어맞지 않는다(poor transfer ability).

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Frozen-encoder 기반 reconstruction/distillation/memory 계열**
- **Pixel reconstruction**(AE [38;39], VAE [40;41], GAN [7;8;42]): 픽셀 값이 정상과 비슷하거나 이상이 미세하면 이상 영역도 잘 복원해버리는 문제 [2;9].
- **Feature reconstruction**(RD4AD [2], ADTR [3], [4]): pixel 대신 pre-trained encoder의 feature를 재구성하지만, pattern collapse를 막기 위해 encoder를 반드시 고정해야 함.
- **Feature distillation**([2;5;6]): student network가 처음부터 학습되어 pre-trained teacher의 정상 영역 feature를 모방.
- **Feature memory & modeling**(PaDiM [10], PatchCore [11], Patch SVDD [12], [13], CFA [14]): 정상 feature 전체를 기억/모델링하고 테스트 시 매칭 — 연산 비용이 크고 encoder는 여전히 target 도메인에 최적화되지 않음.
- 이 모든 갈래가 "encoder는 손대지 않는다"는 전제를 공유하며 semantic gap을 그대로 안고 간다.

**갈래 2 — 도메인 적응을 시도한 최근 연구**
- **CFA [14], SimpleNet [15]**: frozen encoder 뒤에 학습 가능한 linear layer를 붙여 출력 feature를 task-oriented하게 변환. Encoder 추출 단계에서 이미 손실된 도메인 특화 정보를 뒤늦게 복원하기엔 역부족이며, CT/MRI처럼 자연 이미지와 특히 먼 도메인에서는 한계가 뚜렷.

**갈래 3 — OCC(one-class classification) 쪽의 self-supervised pre-training**
- **PANDA [43], [44], [45]**: 정상 샘플만으로 compact representation space를 자체 학습하지만, UAD에 적용하면 MVTec AD 기준 I-AUROC 90% 미만으로 실용성이 떨어짐.

**갭**: <mark style="background: #FFF3A3A6;">"encoder는 pattern collapse 때문에 반드시 얼려야 한다"는 관행 자체에 도전한 연구가 없었다 — 기존 적응 시도(갈래 2)조차 encoder는 그대로 두고 뒤에 얕은 레이어만 덧붙였을 뿐, encoder 내부를 target 도메인에서 직접 최적화하는 방법은 아무도 제시하지 못했다.</mark> 이 논문은 contrastive learning의 collapse 방지 메커니즘(SimSiam류)을 feature reconstruction 구조에 이식해 이 전제 자체를 깨는 첫 시도다.

### 이 논문이 풀고자 하는 문제
1. Encoder를 포함한 네트워크 전체를 target 도메인에서 end-to-end로 최적화해 도메인 편향(semantic gap)을 줄이는 방법을 찾는다.
2. 이 과정에서 발생하는 세 가지 실패 모드 — pattern collapse, 학습 불안정성, decoder가 정상/이상을 구분 없이 모두 잘 복원해버리는 "identical shortcut" — 없이 안정적으로 학습시킨다.

# 제안 방법

<mark style="background: #FFF3A3A6;">핵심 아이디어: Reverse Distillation(RD4AD)류 feature reconstruction 구조(encoder-bottleneck-decoder)에 positive-pair contrastive learning(SimSiam류)의 세 요소 — ① global 관점의 거리 계산, ② stop-gradient, ③ 두 개의 view(=두 encoder)를 이용한 contrastive pair 구성 — 을 순차적으로 도입해 encoder를 안전하게 학습 가능한 상태로 전환한다.</mark> 논문은 RD4AD(Config.A)에서 출발해 Config.B → C → D → E → 최종 ReContrast(hard-mining 포함)까지 단계적으로 구성 요소를 추가하며 각 요소의 필요성을 실험적으로 논증한다.

### ① Global Cosine Distance
- RD4AD의 원래 손실은 encoder/decoder feature map을 지점별(point-by-point)로 비교하는 regional cosine distance.
- 저자들은 논문 그대로의 regional loss로 재현하면 학습이 극도로 불안정(I-AUROC 95.3%까지 하락)한 반면, RD4AD 공식 코드는 실제로 feature map 전체를 flatten해 비교하는 다른 함수(사실상 버그성 구현)를 쓰고 있음을 발견.
- 이를 global cosine distance로 명시적으로 정식화 — feature map을 1-D로 flatten한 뒤 거리 계산.
- Loss landscape 시각화(Fig. 4) 결과 global 방식이 region 방식보다 더 평탄(flatter) — 개별 지점 과적합에 의한 불안정성 제거.

> [!example]- 구현 디테일
> ```
> M^k(h,w) = 1 − f_E^k(h,w)ᵀ·f_D^k(h,w) / (‖f_E^k(h,w)‖ ‖f_D^k(h,w)‖)     (식 1, regional)
> L_region = Σ_k (1/H^kW^k) Σ_{h,w} M^k(h,w)                              (식 2)
> L_global = Σ_{k=1}^{3} 1 − F(f_E^k)ᵀ·F(f_D^k) / (‖F(f_E^k)‖ ‖F(f_D^k)‖)   (식 3, F=flatten)
> ```
> 이 단계는 아직 encoder를 얼린 채 decoder만 바꾼 것(Config.B).

<mark style="background: #FFF9D6A6;">Encoder를 end-to-end로 학습하려면 먼저 학습 자체가 안정적이어야 한다. Global distance로 loss landscape를 평탄하게 만들어 두면, 이후 encoder를 열었을 때도 최적화가 흔들리지 않을 기반이 마련된다.</mark>

### ② Stop-Gradient
- Encoder와 decoder를 그냥 함께 학습(Config.C)시키면 training loss가 거의 0으로 급락하지만, 이는 encoder feature 다양성이 급격히 무너지는 pattern collapse 때문.
- Contrastive SSL의 predictor를 "reconstruction network", decoder를 "predictor"로 재해석해 SimSiam의 stop-gradient를 이식(Config.D) — decoder→encoder 직접 gradient를 막고, "decoder 최적화 → encoder가 더 domain-specific해짐 → 다시 decoder에 더 많은 최적화 요구"하는 상호 강화 구조로 학습.
- 다만 Config.D는 학습 초반 정점을 찍은 뒤 성능이 재하락 — decoder가 정상/이상 모두 잘 복원해버리는 "identical shortcut"에 빠지기 때문(Fig. 5a).

> [!example]- 구현 디테일
> ```
> L_global = Σ_k 1 − sg(F(f_E^k))ᵀ·F(f_D^k) / (‖sg(F(f_E^k))‖ ‖F(f_D^k)‖)   (식 4)
> ```
> `sg`=stop-gradient. Fig. 5(b): Config.D의 feature 표준편차가 Config.C처럼 무너지지 않고 유지됨.

<mark style="background: #FFF9D6A6;">Encoder에 직접 gradient가 들어가지 못하면 decoder만 보고 손쉽게 답을 맞추는 지름길이 사라지고, encoder는 간접 압력만 받으므로 특징이 무의미하게 붕괴하지 않는다 — 즉 stop-gradient가 있어야 비로소 "encoder를 열어도 안전"해진다.</mark>

### ③ Contrastive Pairs (Dual Encoder)
- 일반적인 contrastive learning은 이미지 증강으로 두 view를 만들지만, UAD에서는 어떤 증강(flip/cutout/color jitter)도 잠재적 이상을 만들 수 있고 공간적 증강(shift/rotate/scale)은 지점별 대응관계를 깨뜨려 쓸 수 없음.
- 증강 없이 `x=x'`로 두면 predictor가 자기 입력을 그대로 예측하는 self-reconstruction으로 퇴화 — Config.D와 같은 identical shortcut 위험이 남음.
- 대신 frozen encoder(사전학습 상태 고정)와 domain-specific encoder(target 도메인에서 학습) 두 개를 두어, 하나의 입력에서 augmentation-free한 두 view를 구성(Config.E).
- Decoder+bottleneck이 양방향(domain-specific→frozen, frozen→domain-specific) cross-reconstruction — 6쌍의 feature map 거리 계산.

<mark style="background: #FFF9D6A6;">두 encoder의 출력이 서로 다른 도메인 view이므로 decoder는 더 이상 입력을 그대로 베끼는 지름길을 쓸 수 없고, frozen encoder가 계속 자연 이미지 도메인의 기준점 역할을 하며 domain-specific encoder를 붙잡아준다 — identical shortcut과 도메인 표류를 동시에 억제한다.</mark> Fig. 5(b)에서 Config.E의 feature 다양성은 무너지지 않고 오히려 소폭 증가.

### ④ Hard-Normal Mining
- 정상 영역이라도 edge·디테일이 많은 부분(hard-normal)은 평범한 영역(easy-normal)보다 재구성 오차가 원래 더 크다.
- 이 intrinsic error가 실제 이상에 의한 epistemic error와 혼동되면 탐지 정밀도가 떨어짐.
- 미니배치 평균·표준편차보다 거리가 작은("쉬운") feature point의 gradient를 끊어(stop-gradient) 학습에서 배제하고, hard-normal 영역 최적화에 학습 예산을 집중.

> [!example]- 구현 디테일
> ```
> f_D^k(h,w) = sg(f_D^k(h,w))   if M^k(h,w) < μ(M^k(h,w)) + α·σ(M^k(h,w))
>            = f_D^k(h,w)        otherwise                                (식 5)
> ```
> α는 학습 초반 -3에서 1로 선형 증가(discard rate 0%→~84%).
>
> **최종 모델 하이퍼파라미터**: encoder=WideResNet50(ImageNet 사전학습). AdamW(β=(0.9,0.999), wd=1e-5), encoder lr=1e-5, decoder/bottleneck lr=2e-3(encoder를 훨씬 느리게 학습시켜 표류 억제). MVTec AD/ISIC2018 2,000 iter, VisA 3,000 iter, APTOS/OCT2017 1,000 iter, batch 16(산업)/32(의료). 일부 카테고리(Toothbrush, Leather, Grid, Tile, Wood, Screw / VisA cashew, pcb1 / OCT2017)는 encoder BN을 eval 모드로 둬야 안정적이었음.

<mark style="background: #FFF9D6A6;">①~③이 "encoder를 열어도 무너지지 않게" 만들었다면, hard-mining은 그 위에서 "정상 중에서도 원래 어려운 부분"에 학습 예산을 더 태워 진짜 이상만 도드라지게 한다.</mark> Fig. 5(c): hard-mining 적용 시 easy-normal 과적합도 함께 완화.

# 실험 결과

### 핵심 결과

| 벤치마크 | 지표 | RD4AD (baseline) | Ours (ReContrast) |
|---|---|---|---|
| MVTec AD (단일 클래스, I-AUROC) | All Avg. | 98.5% | **99.5%** |

> [!note]- 세부 결과 및 Ablation
> #### 설정
> - **산업 결함 탐지**: MVTec AD(15개 카테고리, 정상 3,629장/테스트 1,725장), VisA(12개 카테고리, 정상 9,621장/이상 1,200장)
> - **의료 영상**: OCT2017, APTOS(안저·당뇨망막병증), ISIC2018(피부 병변)
> - **지표**: I-AUROC(image-level), P-AUROC/AUPRO(pixel-level), 의료 데이터셋은 F1/ACC 병행
>
> #### MVTec AD — anomaly segmentation (Table 2, P-AUROC/AUPRO)
> | 방법 | All Avg. |
> |---|---|
> | RD4AD [2] | 97.8 / 93.9 |
> | PatchCore [11] | 98.1 / 93.4 |
> | **Ours** | **98.4 / 95.2** |
>
> #### VisA (Table 3)
> | 지표 | RD4AD [2] | PatchCore [11] | **Ours** |
> |---|---|---|---|
> | I-AUROC | 96.0 | 95.1 | **97.5** |
> | P-AUROC | 90.1 | 98.8 | 98.2 |
> | AUPRO | 70.9 | 91.2 | **92.6** |
>
> #### Multi-class unified 모델 (Table 4, 단일 모델로 전 카테고리 처리)
> | 데이터셋 | RD4AD [2] | UniAD [9] | **Ours** |
> |---|---|---|---|
> | MVTec AD I-AUROC | 95.8 | 96.5 | **98.2** |
> | VisA I-AUROC | 92.7 | 91.5 | **95.1** |
>
> #### 의료 영상 (Table 5)
> | 데이터셋 | 지표 | 2위 방법 | **Ours** |
> |---|---|---|---|
> | APTOS | I-AUROC | CFA [14] 94.21 | **97.51** |
> | OCT2017 | I-AUROC | PatchCore [11] 99.61 | 99.60 (comparable best) |
> | ISIC2018 | I-AUROC | AE-flow [32] 87.79 | **90.15** |
>
> APTOS·ISIC2018은 전 지표(I-AUROC/F1/ACC)에서 최고, OCT2017은 비교 가능한 최고 수준. PatchCore 대비 MVTec AD 오차(100%−AUROC)는 0.9%→0.5%로 상대적 44% 감소, VisA AUPRO는 RD4AD 대비 +1.3%p.
>
> #### Ablation (Table 6, MVTec AD / APTOS, last/best iteration)
> | 구성 | `L_global` | encoder 최적화 | stop-grad | contrastive pairs | hard mining | MVTec I-AUROC | MVTec AUPRO | APTOS I-AUROC |
> |---|---|---|---|---|---|---|---|---|
> | Config.A (RD4AD, `L_region`) | | | | | | 95.31/97.55 | 93.34/94.05 | 90.12/90.50 |
> | Config.B (+`L_global`) | ✓ | | | | | 98.86/99.07 | 94.51/94.59 | 92.49/93.62 |
> | Config.C (+encoder 최적화) | ✓ | ✓ | | | | 91.54/95.96 (pattern collapse로 하락) | 88.24/92.14 | 90.71/91.06 |
> | Config.D (+stop-grad) | ✓ | ✓ | ✓ | | | 94.64/97.07 | 84.11/87.38 | 93.06/95.66 |
> | Config.C+cp (stop-grad 없이 contrastive pairs만) | ✓ | ✓ | | ✓ | | 97.59/97.88 | 93.76/93.92 | 92.68/92.68 |
> | Config.E (stop-grad+contrastive pairs) | ✓ | ✓ | ✓ | ✓ | | 99.13/99.34 | 94.59/94.60 | 97.32/97.43 |
> | **Ours (+hard-mining)** | ✓ | ✓ | ✓ | ✓ | ✓ | **99.45/99.52** | **95.20/95.29** | **97.51/97.51** |
>
> #### 세부 발견
> - Config.C가 frozen baseline(Config.B)보다도 크게 떨어지는 것이 pattern collapse의 직접적 증거.
> - Stop-gradient와 contrastive pairs는 각각 단독으로도 Config.C를 일부 회복시키지만(Config.D, Config.C+cp), 두 요소를 모두 넣은 Config.E에서야 frozen encoder baseline(Config.B)을 확실히 능가.
> - Hard-mining은 baseline reconstruction(Config.B+hm: 99.00/99.12)에도 별도로 도움이 되는 독립적인 개선.
> - **α 민감도(Table A4)**: discard rate 50~93% 구간에서 안정적(I-AUROC 99.32~99.45), α=1(discard rate 84.1%)이 최적.
> - **Backbone 일반화(Table A5)**: ResNet18/ResNet50/WideResNet50 전부 RD4AD 대비 일관 개선. ResNet18 기반 Ours(98.7)가 WideResNet50 기반 RD4AD(98.5)와 comparable.
> - **Feature map 조합(Table A6)**: 추론 시 domain-specific encoder feature만 써도(99.38%) frozen encoder만 쓸 때(99.16%)보다 낫고 default(99.45%)와 거의 동등 — dual-encoder가 단순 앙상블이 아니라 domain-specific encoder 자체가 핵심 기여임을 시사. APTOS는 trained-only(97.73%)가 default(97.51%)보다 근소하게 높음.
> - **MVTec LOCO(Table A7, 부록 E)**: structural anomaly에서 GCAD[49](80.6) 대비 우수(90.7)하지만, logical anomaly에서는 GCAD(86.0)에 못 미침(73.4).

# Discussion

### 이 아이디어의 잠재적 부작용
- **Encoder를 열면 학습이 불안정해질 위험**:
  <mark style="background: #FF5582A6;">논문 스스로 이 우려가 현실임을 인정한다 — 일부 카테고리에서 encoder BN을 train 모드로 두면 불안정해져, 저자들은 해당 카테고리(Toothbrush, Leather, Grid, Tile, Wood, Screw / VisA cashew, pcb1 / OCT2017)에서 BN을 eval 모드로 바꿔 우회했다.</mark> "encoder를 열어도 안전하다"는 주장이 무조건적이지 않고 데이터셋별 완화책이 필요한 조건부 안전이다.
- **Dual-encoder pairing이 완전히 collapse를 막지는 못할 가능성**:
  두 encoder의 초기 표현이 우연히 매우 가까운 도메인이라면(자연 이미지와 target 도메인 격차가 작은 경우) contrastive 신호 자체가 약해져 identical shortcut에 가까워질 위험을 이론적으로 배제하기 어렵다. 논문은 이 경우를 별도로 실험하지 않았다.

### 한계
- <mark style="background: #FF5582A6;">적용 범위: 정상/이상이 "같은 객체의 국소적 결함 유무"로 구분되는 UAD에 초점을 맞추며, 정상/이상이 의미론적으로 아예 다른 OCC는 다루지 않는다.</mark>
- <mark style="background: #FF5582A6;">학습 불안정성(저자 명시): validation set이 없는 UAD 특성상 마지막 iteration이 loss spike 구간에 걸리는지가 random seed에 좌우된다.</mark> 원인은 (1) 단일 카테고리 배치에서 BN 분산이 0에 가까워지는 문제, (2) Adam의 gradient 제곱 과거 추정치 관련 불안정성([50;51]). 완화책(BN 분산을 running_var로 대체, Adam 상태 500 iter마다 리셋)으로 MVTec 99.41%/VisA 97.28%까지 회복했지만 <mark style="background: #FF5582A6;">"BN 모드 수동 선택이 불필요한" 완전한 해결에는 이르지 못했다고 자인한다.</mark>
- <mark style="background: #FF5582A6;">Logical anomaly: MVTec LOCO에서 structural anomaly는 GCAD 대비 우수(90.7 vs 80.6)하지만, 부품 누락·배치 오류 같은 logical anomaly는 GCAD(86.0)에 못 미친다(73.4)</mark> — 국소적 재구성 오차 기반이라 "전체적 배치/논리" 이상 포착에는 원천적으로 약함.

### 생각할 점
- <mark style="background: #A6E3A1A6;">"Frozen encoder가 당연하다"는 전제를 깬 이 접근은, UAD를 넘어 다른 self-supervised/frozen-backbone 파이프라인 전반(예: 세그멘테이션의 frozen ViT backbone, few-shot classification의 frozen feature extractor)에도 "왜 얼려야 한다고 생각했는지" 재검토를 촉발하는 사례로 볼 수 있다.</mark>
- Dual encoder를 "frozen + domain-specific" 대신 서로 다른 두 사전학습 도메인(예: ImageNet vs 의료영상 특화)으로 구성하면 domain gap을 더 세밀하게 조정할 수 있을지도 궁금한 지점 — 논문은 frozen encoder를 항상 ImageNet 고정 기준점으로만 사용.

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">[[ReContrast_Dual_Encoder_Contrastive_Reconstruction]]의 hard-normal mining 아이디어(정상 중에서도 재구성이 원래 어려운 영역에 학습을 집중시키는 방식)는, [[Gaussian_Box_Uncertainty_Modeling]]에서 다루는 "라벨 자체가 원래 애매한 영역"을 다루는 방식과 구조적으로 유사하다 — 두 방법 모두 "원래 어려운 것"과 "진짜 이상 신호"를 구분하려는 시도라는 공통점이 있어, uncertainty 관점에서 두 방법을 통합적으로 이해할 여지가 있다.</mark>
- 학습 불안정성 문제(BN·Adam 관련)는 이 논문이 완전히 풀지 못했다고 밝힌 채 남겨둔 지점이므로, Adam의 대안 optimizer나 BN 대신 LayerNorm/GroupNorm으로 이 구조를 다시 학습시켰을 때 안정성이 개선되는지 검증해볼 가치가 있다.

# 관련 개념
- [[ReContrast_Dual_Encoder_Contrastive_Reconstruction]] — 이 논문의 핵심 기여인, contrastive learning 요소(global distance, stop-gradient, dual encoder)를 feature reconstruction UAD에 결합해 encoder까지 end-to-end로 최적화하는 프레임워크.

# 관련 문서
(아직 이 논문과 직접 비교할 만한 다른 anomaly-detection 노트가 위키에 없음 — 향후 RD4AD, PatchCore, UniAD 등이 추가되면 비교 문서 작성 검토)

# 읽어볼 만한 논문
- 참고문헌 기반: H. Deng and X. Li, "Anomaly Detection via Reverse Distillation from One-Class Embedding" (CVPR 2022) [2] — 이 논문의 직접적인 baseline이자 출발점(Config.A). ReContrast의 모든 단계적 개선이 이 구조를 기준으로 설명되므로 먼저 읽어야 논문의 논증 흐름이 이해된다.
- 참고문헌 기반: X. Chen and K. He, "Exploring simple Siamese representation learning" (SimSiam, CVPR 2021) [16] — ReContrast가 stop-gradient 아이디어를 직접 차용한 원조 논문. Collapse 방지 메커니즘 자체를 더 깊이 이해하려면 필수.
- 참고문헌 기반: Z. You et al., "A unified model for multi-class anomaly detection" (UniAD, NeurIPS 2022) [9] — multi-class unified 세팅에서 ReContrast가 비교하는 SOTA. 단일 모델로 여러 카테고리를 처리하는 방향은 산업 배포 관점에서 실용적 함의가 크다.
- 참고문헌 기반: P. Bergmann et al., "Beyond dents and scratches: Logical constraints in unsupervised anomaly detection and localization" (GCAD, IJCV 2022) [49] — ReContrast가 유일하게 뒤처지는 logical anomaly 영역에서 특화 설계된 방법. ReContrast의 한계(구조적 vs 논리적 이상)를 보완하는 방향을 구체적으로 이해하는 데 도움.
- 자유 추천(검증 필요): Adam optimizer의 불안정성을 다루는 최신 개선 연구 — 검색 키워드: `Adam optimizer instability large-scale training second moment estimate fix`. ReContrast가 부록 E에서 스스로 미해결로 남긴 학습 불안정성 문제(원인으로 지목한 [50;51]의 후속 연구)를 확인할 때 참고할 만하다.
