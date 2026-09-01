---
pm-task: true
projectId: "paperwiki-anomaly-detection"
parentId:
id: "t-recontrast-rkme5u3rtk"
title: "ReContrast: Domain-Specific Anomaly Detection via Contrastive Reconstruction"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-08-04"
due:
progress: 0
assignees: []
tags: []
customFields:
  "poo7xxb0mtck1e8q": 2023
  "0sgdhb2bmtck1e8r": "NeurIPS"
subtaskIds: []
dependencies: []
year: 2023
venue: "NeurIPS"
jcr_quartile: "Q1"
task: [anomaly-detection]
direction: [novel-approach, improvement]
paper_tags: [paper, anomaly-detection, contrastive-learning, feature-reconstruction, domain-adaptation, mvtec-ad]
source: "Projects/논문_pdf/Anomaly_Detection/2023_NeurIPS_ReContrast.pdf"
source_type: personal
createdAt: "2026-08-18T11:00:00.000Z"
updatedAt: "2026-08-18T11:00:00.000Z"
---

Project: [[논문_Anomaly_Detection|Anomaly Detection]]
#paper #anomaly-detection #contrastive-learning #feature-reconstruction #domain-adaptation #mvtec-ad

> [!quote] 원제
> **ReContrast: Domain-Specific Anomaly Detection via Contrastive Reconstruction**
> Jia Guo, Shuai Lu, Lize Jia, Weihang Zhang, Huiqi Li — Beijing Institute of Technology (School of Information and Electronics / School of Medical Technology), NeurIPS 2023
> https://arxiv.org/abs/2306.02602

# 한 줄 요약
<mark style="background: #FFF3A3A6;">Feature reconstruction 기반 비지도 이상 탐지(UAD)에서 그동안 pattern collapse를 막기 위해 반드시 고정(frozen)해야 했던 ImageNet 사전학습 encoder를, contrastive learning의 세 요소(global cosine distance, stop-gradient, dual-encoder contrastive pair)를 접목해 decoder와 함께 end-to-end로 안전하게 최적화함으로써 도메인 편향 문제를 해결한 방법.</mark>

> [!info] 내 메모
> 

# 정리

## 기존 방법의 한계
- **Frozen encoder의 도메인 편향**:
  Feature reconstruction·distillation·memory 계열 UAD는 모두 ImageNet 등 자연 이미지로 사전학습된 encoder를 특징 추출기로 얼려 쓴다. 산업 결함·의료 영상처럼 자연 이미지와 거리가 먼 target 도메인에서는 이 frozen feature가 잘 들어맞지 않는다(semantic gap).
- **Encoder를 열면 발생하는 pattern collapse**:
  Encoder를 decoder와 함께 학습시키면, decoder가 재구성하기 쉬운 무의미한(indiscriminative) 특징으로 encoder가 수렴해버리는 trivial solution(pattern collapse)이 발생한다. 이 때문에 "encoder freeze"가 당연한 전제로 굳어졌다.

## 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Frozen-encoder reconstruction/distillation/memory**
- feature reconstruction(RD4AD[2], ADTR[3], [4]), feature distillation([2;5;6]), feature memory & modeling(PaDiM[10], PatchCore[11], Patch SVDD[12], [13], CFA[14]) — 전부 encoder는 손대지 않는다는 전제를 공유
- **타겟/해결**: Frozen encoder의 도메인 편향(문제①) — encoder를 아예 건드리지 않으므로 이 문제를 정면으로 다루지 않고 우회한다.

**갈래 2 — 도메인 적응 시도**
- <mark style="background: #FFF3A3A6;">CFA[14], SimpleNet[15]: frozen encoder 뒤에 학습 가능한 linear layer만 붙여 출력 feature를 task-oriented하게 변환.</mark>
- **타겟/해결**: <mark style="background: #FFF3A3A6;">Frozen encoder의 도메인 편향(문제①) — encoder 자체는 그대로 두고 뒤에 얕은 레이어만 덧붙였을 뿐, encoder 내부를 직접 최적화하는 방법은 없었다. 특히 CT/MRI처럼 자연 이미지와 먼 도메인에서는 이미 추출 단계에서 손실된 도메인 정보를 얕은 레이어로 복원하기 역부족이다.</mark>

**갈래 3 — Positive-pair contrastive SSL / OCC self-supervised pretraining**
- <mark style="background: #FFF3A3A6;">Positive-pair contrastive SSL(SimSiam[16], BYOL[17]): stop-gradient로 collapse 방지.</mark>
- OCC self-supervised pretraining(PANDA[43], [44], [45]): 정상 샘플만으로 compact representation space를 자체 학습하지만 UAD 적용 시 MVTec AD I-AUROC 90% 미만으로 실용성 부족.
- **타겟/해결**: <mark style="background: #FFF3A3A6;">Encoder를 열면 발생하는 pattern collapse(문제②) — contrastive learning은 collapse 방지 메커니즘을 이미 갖고 있지만, 이를 feature reconstruction 구조에 직접 이식한 시도가 없었다.</mark>

**갭**: <mark style="background: #FFF3A3A6;">"encoder는 pattern collapse 때문에 반드시 얼려야 한다"는 관행 자체에 도전한 연구가 없었다 — 기존 적응 시도(갈래 2)조차 encoder는 그대로 두고 뒤에 얕은 레이어만 덧붙였을 뿐이었고, contrastive SSL(갈래 3)의 collapse 방지 메커니즘을 reconstruction UAD 구조에 이식한 시도도 없었다.</mark>

## 이 논문이 풀고자 하는 문제
1. Encoder를 포함한 네트워크 전체를 target 도메인에서 end-to-end로 최적화해 도메인 편향을 줄이는 것.
2. Pattern collapse·학습 불안정성·decoder가 정상/이상을 구분 없이 다 잘 복원해버리는 "identical shortcut" 없이 encoder를 안전하게 학습시키는 것.

**갭 종합**: <mark style="background: #FFF3A3A6;">ReContrast의 통찰은 contrastive SSL이 이미 갖고 있던 collapse 방지 메커니즘(global distance·stop-gradient·dual-view)을 reconstruction UAD 구조에 그대로 재해석해 이식하면, encoder를 열어도 무너지지 않는다는 것이다.</mark>

> [!info] 내 메모
> 

# 해결 방법 요약

| | 문제 ① — Frozen encoder의 도메인 편향 | 문제 ② — Encoder를 열면 발생하는 pattern collapse |
|---|---|---|
| **해결 방법** | Contrastive learning의 stop-gradient·global distance·dual-view 구성을 feature reconstruction에 이식해 encoder를 decoder와 함께 target 도메인에서 직접 최적화 | Decoder를 SimSiam의 predictor로 재해석해 stop-gradient를 적용하고, augmentation 대신 frozen encoder + domain-specific encoder라는 "서로 다른 두 view"로 contrastive pair를 구성 |
| **예상되는 문제점** | 이 최적화는 카테고리별 데이터가 적은 UAD 환경에서 batch normalization 통계가 불안정해질 위험을 안고 있다(아래 "제안 방법" ③, Discussion 참고) | Dual-encoder 구조가 추론 시 encoder 2개(파라미터·연산량 약 2배)를 요구하고, 학습 자체도 여전히 일부 불안정성(loss spike)이 완전히 해소되지 않는다 |

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">Reverse Distillation(RD4AD)류 feature reconstruction 구조(encoder-bottleneck-decoder)에 <span style="color:#c0392b; font-weight:bold;">global cosine distance</span>, <span style="color:#c0392b; font-weight:bold;">stop-gradient</span>, <span style="color:#c0392b; font-weight:bold;">frozen encoder + domain-specific encoder의 dual-encoder contrastive pair</span>를 순차적으로 도입해 encoder를 decoder와 함께 안전하게 end-to-end 학습시키고, 마지막으로 <span style="color:#c0392b; font-weight:bold;">hard-normal mining</span>으로 정상 중에서도 원래 재구성이 어려운 영역에 학습을 집중시킨다.</mark>

## 전체 파이프라인 (Fig. 2 Config.E + Eq.5 기준, 최종 ReContrast 구조)

```
입력 이미지 x
       │
       ├──────────────────────────────┐
       ▼                                ▼
① Frozen Encoder (WideResNet50,     ① Domain-Specific Encoder (같은 구조,
   ImageNet 사전학습 고정, BN eval)     ImageNet 사전학습에서 시작해 target
       │                                도메인에서 함께 학습됨, lr=1e-5)
       │ f_E,frozen^k  (k=1,2,3층)       │ f_E,domain^k  (k=1,2,3층)
       │  각 (C^k, H^k, W^k)              │  각 (C^k, H^k, W^k)
       ▼                                ▼
       └──────────────┬─────────────────┘
                       ▼
② Bottleneck (RD4AD와 동일 구조, 1×1 conv 2개로 두 encoder feature를 각각 project)
                       │  → (C⁴, H⁴, W⁴)  [4번째 layer 해당 압축 표현]
                       ▼
③ Decoder (encoder를 역순으로 뒤집은 구조, up-sampling으로 3개 층 복원, lr=2e-3)
       │
       ├─→ decoder(bottleneck(frozen 특징))   → f_D→frozen^k   (k=1,2,3)
       └─→ decoder(bottleneck(domain 특징))   → f_D→domain^k   (k=1,2,3)
                       │  (양방향 cross-reconstruction, 가중치 공유)
                       ▼
④ Global Cosine Distance × 6쌍 (stop-gradient 적용)     → M^k(h,w) 6세트, 각 (H^k, W^k)
       │  frozen enc ↔ domain enc→decoder 재구성 3쌍
       │  domain enc ↔ frozen enc→decoder 재구성 3쌍
       ▼
⑤ Hard-Normal Mining (학습 시에만, α로 조절되는 discard rule)   → 학습에 남는 gradient만 선별
       │
       ▼ (추론 시)
⑥ Anomaly Map 집계  — 6세트 M^k를 모두 이미지 크기로 up-sampling 후 합산  → S^map (H₀, W₀)
       │
       ▼
S^img = max(S^map)  — 이미지 단위 이상 점수
```

> [!info] 내 메모
> 

### ① Global Cosine Distance
- **역할**:
  RD4AD가 쓰던 지점별(point-by-point) regional cosine distance는 개별 지점에 과적합되기 쉬워 학습이 불안정하다. Global cosine distance는 <span style="color:#c0392b; font-weight:bold;">feature map 전체를 1-D로 flatten한 뒤 하나의 벡터로 거리를 계산</span>해, 개별 지점이 아니라 전체 매니폴드 간 일관성을 측정하도록 목적함수를 바꾼다. 이 단계는 아직 encoder를 얼린 채(Config.B) decoder 손실 함수만 바꾼 상태다.
- **구현**:
  encoder/decoder의 각 층 feature map `f_E^k, f_D^k ∈ ℝ^(C^k×H^k×W^k)`를 flatten 연산 `F`로 벡터화한 뒤 cosine distance를 계산. 흥미롭게도 저자들은 RD4AD의 공식 코드가 논문에 적힌 regional 공식이 아니라 이 global 방식(사실상 코딩 버그)을 쓰고 있었고, 논문 그대로의 regional 공식으로 재현하면 I-AUROC가 95.3%까지 떨어짐을 발견했다.
- **입출력 shape**: `f_E^k, f_D^k` 각 `(C^k, H^k, W^k)` → flatten `(C^k·H^k·W^k,)` → 스칼라 거리 1개(층별 k=1,2,3 합산).

```python
# 논문 Eq.(1)-(3) 기반. M^k: 2-D distance map(regional), L_global: flatten 후 스칼라 거리
# Eq.1 (regional, point-by-point)
M_k(h, w) = 1 - dot(f_E_k[:, h, w], f_D_k[:, h, w]) / (norm(f_E_k[:, h, w]) * norm(f_D_k[:, h, w]))

# Eq.3 (global, flatten)
L_global = sum(
    1 - dot(flatten(f_E_k), flatten(f_D_k)) / (norm(flatten(f_E_k)) * norm(flatten(f_D_k)))
    for k in [1, 2, 3]
)
```

<mark style="background: #FFF9D6A6;">Loss landscape 시각화(Fig. 4)에서 global 방식이 regional 방식보다 평탄(flatter)함이 확인된다 — encoder를 end-to-end로 열기 전에 먼저 최적화 지형 자체를 안정시켜 두는 사전 준비 단계로, 이것이 없으면 문제 ②의 불안정성이 encoder를 열기도 전에 이미 존재한다.</mark> 실제로 Config.B(I-AUROC 98.86)는 Config.A(RD4AD 원 공식, 95.31)보다 크게 안정적이다(Table 6).

> [!info] 내 메모
> 

### ② Stop-Gradient
- **역할**:
  Encoder와 decoder를 그냥 함께 학습시키면(Config.C) training loss가 거의 0으로 급락하는데, 이는 encoder feature 다양성이 붕괴하는 pattern collapse 때문이다(Table 6에서 Config.C의 I-AUROC가 frozen baseline Config.B보다도 낮은 91.54로 하락). Stop-gradient는 decoder→encoder로 직접 흐르는 gradient를 차단해, decoder를 SimSiam의 predictor로, reconstruction 자체를 contrastive learning으로 재해석하는 핵심 장치다.
- **구현**:
  Global cosine distance 식(Eq.3)의 encoder 쪽 항에 stop-gradient(`sg`) 연산을 적용한다. Encoder는 decoder로부터의 직접 gradient 없이, decoder가 계속 최적화되면서 간접적으로 "더 domain-specific한 feature를 요구받는" 상호 강화(mutual reinforcement) 방식으로 학습된다.
- **입출력 shape**: ①과 동일 — `(C^k, H^k, W^k)` → 스칼라 거리(encoder 쪽 gradient만 차단).

```python
# 논문 Eq.(4) 기반. sg: stop-gradient(순전파 값은 유지, 역전파만 차단)
L_global = sum(
    1 - dot(sg(flatten(f_E_k)), flatten(f_D_k)) / (norm(sg(flatten(f_E_k))) * norm(flatten(f_D_k)))
    for k in [1, 2, 3]
)
```

<mark style="background: #FFF9D6A6;">Encoder에 decoder로부터의 직접 gradient가 들어가지 못하면 "decoder만 보고 손쉽게 답을 맞추는" 지름길이 사라져, encoder가 무의미한 특징으로 붕괴하지 않는다 — Fig. 5(b)에서 Config.D의 feature std가 Config.C처럼 무너지지 않고 유지되는 것이 이를 직접 뒷받침하며, 문제 ②의 pattern collapse를 정면으로 해소한다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> Stop-gradient만으로는 부작용이 남는다 — Config.D는 학습 초반 정점을 찍은 뒤 성능이 재하락한다(Fig. 5a). Decoder가 정상/이상 구분 없이 둘 다 잘 복원해버리는 "identical shortcut"에 빠지기 때문이며, 이는 ③에서 해결된다.

> [!info] 내 메모
> 

### ③ Contrastive Pairs (Dual Encoder)
- **역할**:
  일반적인 contrastive learning은 이미지 증강(flip, color jitter 등)으로 두 view를 만들지만, UAD에서는 어떤 증강도 잠재적 이상을 만들 수 있고 공간적 증강(shift/rotate/scale)은 지점별 대응관계를 깨뜨려 애초에 쓸 수 없다. `x=x'`로 두면 predictor(decoder)가 자기 입력을 그대로 예측하는 self-reconstruction으로 퇴화해 Config.D와 같은 identical shortcut 위험이 남는다. 이를 피하기 위해 <span style="color:#c0392b; font-weight:bold;">frozen encoder(사전학습 상태로 고정)와 domain-specific encoder(target 도메인에서 학습되는, ①②를 적용한 encoder)</span> 두 개를 두어, 증강 없이도 한 이미지에서 "서로 다른 두 view"를 자연스럽게 구성한다.
- **구현**:
  Decoder+bottleneck이 두 encoder의 특징을 상호(cross) 재구성한다 — domain-specific encoder의 특징을 입력받아 frozen encoder의 특징을 재구성하고, 반대로 frozen encoder의 특징을 입력받아 domain-specific encoder의 특징을 재구성한다(양방향, 가중치 공유). 그 결과 총 6쌍의 feature map 거리(층 3개 × 방향 2개)가 계산되고, `S^map`은 이 6개의 up-sampling된 거리 맵을 모두 합산해서 얻는다.
- **입출력 shape**: frozen encoder 출력 `(C^k, H^k, W^k)` × 3층 + domain-specific encoder 출력 `(C^k, H^k, W^k)` × 3층 → cross-reconstruction된 decoder 출력 6세트 `(C^k, H^k, W^k)` → 거리 맵 6세트.

```python
# 논문 Config.E(Figure 2), Section 2.5 기반 의사코드
f_frozen = frozen_encoder(x)          # 3개 층, 사전학습 후 고정
f_domain = domain_encoder(x)          # 3개 층, target 도메인에서 학습(stop-grad 적용)

recon_from_domain = decoder(bottleneck(f_domain))   # domain -> frozen 재구성
recon_from_frozen = decoder(bottleneck(f_frozen))   # frozen -> domain 재구성

loss = global_cosine_distance(sg(f_frozen), recon_from_domain) \
     + global_cosine_distance(sg(f_domain), recon_from_frozen)   # 6쌍(층3 x 방향2) 합
```

<mark style="background: #FFF9D6A6;">두 encoder의 출력이 서로 다른 도메인 view이므로 decoder는 더 이상 입력을 그대로 베끼는 지름길을 쓸 수 없고, frozen encoder가 계속 자연 이미지 도메인의 기준점 역할을 하며 domain-specific encoder를 붙잡아준다 — identical shortcut과 도메인 표류를 동시에 억제해 문제 ①(도메인 편향)과 문제 ②(collapse) 둘 다에 대응한다.</mark> Fig. 5(b)에서 Config.E의 encoder feature 다양성은 무너지지 않고 오히려 소폭 증가하며, Table 6에서 Config.E(99.13)가 frozen encoder baseline Config.B(98.86)를 확실히 넘어선다.

> [!warning] 이 구조 때문에 예상되는 문제점
> Encoder를 두 개 두는 구조라 추론 시 파라미터·연산량이 늘어난다(Table A5 — WideResNet50 기준 RD4AD 117M/36.0G MACs → Ours 145M/75.2G MACs, 약 2배). 또한 카테고리별 데이터가 적은 UAD 환경에서 일부 카테고리는 encoder의 batch normalization을 train 모드로 두면 학습이 불안정해져, 저자들이 실제로 일부 카테고리(Toothbrush, Leather, Grid, Tile, Wood, Screw / VisA cashew, pcb1 / OCT2017)에서 BN을 eval 모드로 강제 전환해 우회했다(자세한 원인은 Discussion 참고).

> [!info] 내 메모
> 

### ④ Hard-Normal Mining
- **역할**:
  ①~③이 "encoder를 열어도 무너지지 않게" 만들었다면, hard-normal mining은 그 위에서 탐지 정밀도 자체를 높이는 마지막 장치다. 정상 영역이라도 edge·디테일이 많은 부분(hard-normal)은 평범한 영역(easy-normal)보다 재구성 오차가 원래 크다. 이 intrinsic error가 실제 이상에 의한 epistemic error와 혼동되면 탐지 정밀도가 떨어지므로, "쉬운(easy)" 정상 지점의 gradient는 끊고 hard-normal 영역에 학습 예산을 집중시킨다.
- **구현**:
  단순히 easy-normal 지점의 feature 자체를 버리면 `L_global`이 요구하는 전역 매니폴드 구조가 깨지므로, decoder 출력 `f_D^k(h,w)`가 미니배치 평균보다 충분히 가까우면(=쉬우면) 그 지점만 stop-gradient 처리하는 `L_global-hm`으로 대체한다.
- **입출력 shape**: ③의 거리 맵 `M^k(h,w)` 6세트 각 `(H^k, W^k)` → 동일 shape의 선별된 gradient(easy 지점은 gradient 0, hard 지점만 학습 반영).

```python
# 논문 Eq.(5) 기반. mu, sigma: 미니배치 근사 평균/표준편차. alpha: 학습 초반 -3 -> 1로 선형 증가
for h, w in spatial_positions:
    if M_k[h, w] < mu(M_k) + alpha * sigma(M_k):    # "쉬운" 지점
        f_D_k[h, w] = sg(f_D_k[h, w])                # gradient 차단, 값은 유지
    # else: 그대로 두어 gradient가 흐르게 함 (hard-normal)
```

α는 학습 초반 1/10 구간 동안 -3에서 1로 선형 증가한 뒤 나머지 구간은 1로 고정 — 학습 초반엔 거의 모든 지점을 학습에 반영(discard rate 0%)하다가, 점차 hard-normal 영역(discard rate 약 84%)에만 집중하도록 스케줄링된다.

<mark style="background: #FFF9D6A6;">Intrinsic error(원래 어려운 정상 영역의 재구성 오차)와 epistemic error(진짜 이상 신호)를 혼동하지 않도록 학습 압력을 재배분해, 최종 탐지 정밀도를 한 단계 더 끌어올린다 — Table 6에서 hard-mining 추가만으로 Config.E(99.13) → Ours(99.45)로 오르고, baseline reconstruction(Config.B)에 단독 적용해도 Config.B+hm(99.00)으로 개선되어 독립적인 기여임이 확인된다.</mark> Fig. 5(c): hard-mining 적용 시 easy-normal 영역이 덜 활성화되어 과적합도 함께 완화된다.

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| Frozen/Domain Encoder | 이미지 `(3,H₀,W₀)` | `f^k (C^k,H^k,W^k)` × 3층 × 2개 encoder | 두 도메인 view 추출 | WideResNet50 첫 3개 layer(기본), ResNet18/50도 지원(Table A5) |
| Bottleneck | `f^k` × 3층 | `(C⁴,H⁴,W⁴)` | 4번째 layer급 압축 표현 생성 | 1×1 conv 2개(RD4AD와 동일 구조) |
| Decoder | `(C⁴,H⁴,W⁴)` | `f_D^k` × 3층 × 2방향(cross) | 반대편 encoder 특징으로 재구성 | encoder를 역순으로 뒤집은 구조, up-sampling |
| ① Global Cosine Distance | `f_E^k, f_D^k` 각 `(C^k,H^k,W^k)` | 스칼라 거리 (층별 합산) | 지점 과적합 대신 전역 일관성 측정 | flatten 후 cosine distance |
| ② Stop-Gradient | 위와 동일 | 동일 (encoder gradient 차단) | pattern collapse 방지 | encoder 쪽 항에 `sg` 적용 |
| ③ Contrastive Pairs | frozen/domain encoder 특징 각 3층 | cross-reconstruction 6쌍 | identical shortcut·도메인 표류 억제 | dual-encoder + 양방향 재구성 |
| ④ Hard-Normal Mining | 거리 맵 `M^k` 6세트 | 선별된 gradient | intrinsic/epistemic error 혼동 완화 | 미니배치 평균+α·표준편차 기준 discard |
| Anomaly Map 집계 | `M^k` 6세트 각 `(H^k,W^k)` | `S^map (H₀,W₀)`, `S^img` | 최종 이상 맵/점수 | up-sampling 후 합산, max로 이미지 점수 |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table 1 (MVTec AD, 단일 클래스 I-AUROC)
**보는 법**: All Avg 행이 15개 카테고리 평균 — RD4AD(frozen encoder baseline) 대비 ReContrast(encoder까지 최적화)의 개선폭을 보면 된다.

| 벤치마크 | 지표 | RD4AD (baseline) | Ours (ReContrast) |
|---|---|---|---|
| MVTec AD (Table 1) | I-AUROC All Avg. | 98.5% | **99.5%** |

> [!note]- 세부 결과 및 Ablation
> #### 설정
> - **산업 결함 탐지**: MVTec AD(15개 카테고리, 정상 3,629장/테스트 1,725장, 256×256), VisA(12개 카테고리, 정상 9,621장/이상 1,200장, 256×256)
> - **의료 영상**: OCT2017(광간섭단층촬영, 정상 26,315장/테스트 1,000장), APTOS(안저·당뇨망막병증, 정상 1,000장 학습/2,662장 테스트), ISIC2018(피부 병변, 정상 6,705장/테스트 193장, ISIC은 이상=다른 병변 클래스라 사실상 OCC에 가까워 S^map 집계에 max 대신 mean 사용)
> - **지표**: I-AUROC(image-level), P-AUROC/AUPRO(pixel-level), 의료 데이터셋은 F1/ACC 병행(threshold는 최적 F1 기준)
> - **구현**: WideResNet50(ImageNet 사전학습) 기본 backbone, AdamW(β=(0.9,0.999), wd=1e-5), decoder/bottleneck lr=2e-3·encoder lr=1e-5(encoder를 훨씬 느리게 학습시켜 표류 억제). MVTec AD/ISIC2018 2,000 iter, VisA 3,000 iter, APTOS/OCT2017 1,000 iter, batch 16(산업)/32(의료). RTX 3090(24GB), PyTorch 1.12.0.
>
> #### MVTec AD — anomaly segmentation (Table 2, P-AUROC/AUPRO)
> **보는 법**: All Avg 행의 두 숫자(P-AUROC/AUPRO)를 RD4AD·PatchCore와 비교.
> | 방법 | Text. Avg. | Obj. Avg. | All Avg. |
> |---|---|---|---|
> | RD4AD [2] | 97.7 / 95.0 | 97.9 / 93.4 | 97.8 / 93.9 |
> | PatchCore [11] | 97.5 / 93.6 | 98.4 / 93.3 | 98.1 / 93.4 |
> | **Ours** | **98.0 / 96.2** | **98.6 / 94.8** | **98.4 / 95.2** |
>
> #### VisA (Table 3)
> | 지표 | RD4AD [2] | PatchCore [11] | **Ours** |
> |---|---|---|---|
> | I-AUROC | 96.0 | 95.1 | **97.5** |
> | P-AUROC | 90.1 | 98.8 | 98.2 |
> | AUPRO | 70.9 | 91.2 | **92.6** |
>
> #### Multi-class unified 모델 (Table 4, 단일 모델로 전 카테고리 처리, 5,000 iter)
> | 데이터셋 | RD4AD [2] | UniAD [9] | **Ours** |
> |---|---|---|---|
> | MVTec AD I-AUROC | 95.8 | 96.5 | **98.2** |
> | VisA I-AUROC | 92.7 | 91.5 | **95.1** |
>
> #### 의료 영상 (Table 5, I-AUROC/F1/ACC)
> | 데이터셋 | 2위 방법 | **Ours** |
> |---|---|---|
> | APTOS | CFA[14] 94.21 | **97.51 / 95.27 / 93.35** (전 지표 최고) |
> | OCT2017 | PatchCore[11] I-AUROC 99.61 | 99.60 / 98.53 / 97.80 (comparable best) |
> | ISIC2018 | AE-flow[32] I-AUROC 87.79 | **90.15 / 81.12 / 86.01** (전 지표 최고) |
>
> PatchCore 대비 MVTec AD 오차(100%−I-AUROC)는 0.9%→0.5%로 상대적 44% 감소, VisA AUPRO는 RD4AD 대비 +1.3%p, +1.4%p(PatchCore 대비).
>
> #### Table 6 — Ablation (MVTec AD I-AUROC/AUPRO, APTOS I-AUROC, last/best iteration)
> **보는 법**: 각 구성요소를 하나씩 추가하며 last iteration/best iteration 값을 함께 본다(일부 구성이 불안정해 last와 best가 다름).
> | 구성 | `L_global` | encoder 최적화 | stop-grad | contrastive pairs | hard mining | MVTec I-AUROC | MVTec AUPRO | APTOS I-AUROC |
> |---|---|---|---|---|---|---|---|---|
> | Config.A (RD4AD, `L_region`) | | | | | | 95.31/97.55 | 93.34/94.05 | 90.12/90.50 |
> | Config.B (+`L_global`) | ✓ | | | | | 98.86/99.07 | 94.51/94.59 | 92.49/93.62 |
> | Config.B+hm | ✓ | | | | ✓ | 99.00/99.12 | 94.86/94.71 | 93.87/94.36 |
> | Config.C (+encoder 최적화) | ✓ | ✓ | | | | 91.54/95.96 (pattern collapse로 하락) | 88.24/92.14 | 90.71/91.06 |
> | Config.D (+stop-grad) | ✓ | ✓ | ✓ | | | 94.64/97.07 | 84.11/87.38 | 93.06/95.66 |
> | Config.C+cp (stop-grad 없이 contrastive pairs만) | ✓ | ✓ | | ✓ | | 97.59/97.88 | 93.76/93.92 | 92.68/92.68 |
> | Config.E−gl (`L_global` 없이 stop-grad+cp) | | ✓ | ✓ | ✓ | | 98.93/99.26 | 94.65/94.71 | 96.39/96.39 |
> | Config.E (stop-grad+contrastive pairs) | ✓ | ✓ | ✓ | ✓ | | 99.13/99.34 | 94.59/94.60 | 97.32/97.43 |
> | **Ours (+hard-mining)** | ✓ | ✓ | ✓ | ✓ | ✓ | **99.45/99.52** | **95.20/95.29** | **97.51/97.51** |
>
> - Config.C가 frozen baseline(Config.B)보다도 크게 떨어지는 것이 pattern collapse의 직접적 증거.
> - Stop-gradient와 contrastive pairs는 각각 단독으로도 Config.C를 일부 회복시키지만(Config.D, Config.C+cp), 두 요소를 모두 넣은 Config.E에서야 frozen encoder baseline(Config.B)을 확실히 능가.
> - Config.E−gl과 Config.E를 비교하면 `L_global`(개별 요소로는 이미 Config.B에서 검증됨)이 stop-grad+contrastive pairs 조합 위에서도 추가 이득을 줌(MVTec AUPRO는 거의 동률이나 APTOS는 96.39→97.32로 개선).
> - Hard-mining은 baseline reconstruction(Config.B+hm)에도 별도로 도움이 되는 독립적인 개선.
>
> #### Table A4 — α 민감도 (`L_global-hm`, MVTec AD)
> **보는 법**: α가 커질수록 discard rate(쉬운 지점을 gradient에서 제외하는 비율)가 커진다 — 극단값(0%, 97.7%)보다 중간 구간이 낫다는 걸 확인.
> α=-2/-1/0/0.5/1/1.5/2 → discard rate 2.3%/15.9%/50%/69.1%/84.1%/93.3%/97.7%, I-AUROC 99.13/99.13/99.14/99.32/**99.45**/99.38/98.85. Discard rate 50~93.3% 구간에서 I-AUROC 99.32~99.45로 안정적이며 α=1(discard rate 84.1%)이 최적.
>
> #### Table A5 — Backbone 일반화 (MVTec AD)
> **보는 법**: Params/MACs로 비용을, I-AUROC/P-AUROC/AUPRO로 성능을 backbone별 RD4AD와 짝지어 비교.
> | Backbone | Params(M) RD4AD/Ours | MACs(G) RD4AD/Ours | I-AUROC RD4AD/Ours | P-AUROC | AUPRO |
> |---|---|---|---|---|---|
> | ResNet18 | 18.7/21.7 | 4.95/10.1 | 97.9/**98.7** | 97.1/**97.9** | 91.2/**94.2** |
> | ResNet50 | 63.6/74.9 | 19.4/42.0 | 98.4/**99.1** | 97.7/**98.3** | 93.1/**95.0** |
> | WideResNet50 | 117/145 | 36.0/75.2 | 98.5/**99.5** | 97.8/**98.4** | 93.9/**95.2** |
>
> 모든 backbone에서 일관되게 RD4AD를 능가. ResNet18 기반 Ours(98.7)가 WideResNet50 기반 RD4AD(98.5)와 comparable — encoder 최적화 자체의 기여가 backbone 크기보다 클 수 있음을 시사.
>
> #### Table A6 — Feature map 조합 Ablation (dual-encoder가 단순 앙상블인지 검증)
> **보는 법**: 추론 시 어떤 encoder의 feature로 S^map을 계산하는지 바꿔가며 비교.
> | 구성 | encoder feature 조합 | MVTec AD | APTOS |
> |---|---|---|---|
> | Config.B | frozen 3개 | 98.86 | 92.49 |
> | Config.B (R50+WR50 앙상블) | frozen 6개(다른 backbone 2개) | 98.94 | 92.14 |
> | Ours | frozen 3개만 | 99.16 | 95.32 |
> | Ours | domain-specific 3개만 | 99.38 | **97.73** |
> | Ours (default) | trained 3 + frozen 3 | **99.45** | 97.51 |
>
> Domain-specific encoder 단독(99.38)이 frozen 단독(99.16)보다 낫고 default(99.45)와 거의 동등 — dual-encoder가 단순 앙상블 효과가 아니라 domain-specific encoder 자체의 학습이 핵심 기여임을 보여줌. APTOS는 domain-specific 단독(97.73)이 default(97.51)보다 근소하게 앞섬.
>
> #### Table A7 — MVTec LOCO (구조적 vs 논리적 이상)
> **보는 법**: struct. I-AUROC(정상 vs 구조적 이상), logic. I-AUROC(정상 vs 논리적 이상), mean이 둘의 평균. GCAD는 logical anomaly 전용 특화 설계.
> | | RD4AD | DRAEM | PaDiM | PatchCore | GCAD[49] | **Ours** |
> |---|---|---|---|---|---|---|
> | struct. I-AUROC | 88.0 | 74.4 | 70.5 | 82.0 | 80.6 | **90.7** |
> | logic. I-AUROC | 69.4 | 72.8 | 63.7 | 69.0 | **86.0** | 73.4 |
> | mean I-AUROC | 78.7 | 73.6 | 67.1 | 75.5 | **83.3** | 82.1 |
>
> ReContrast는 structural anomaly에서 압도적 1위지만, logical anomaly는 특화 설계된 GCAD에 못 미침 — 아래 Discussion에서 재론.
>
> #### Fig. A1-A3 — 정성적 시각화
> **보는 법**: 이미지/ground truth/S^map을 나란히 배치. Fig. A1은 hard-mining 유무에 따른 MVTec AD S^map 비교(hard-normal 영역이 hm 적용 시 덜 활성화됨), Fig. A2는 VisA, Fig. A3은 세 의료 데이터셋(정상/이상 샘플 비교) 결과.
>
> #### Table A2 — 3-seed 재현성 (MVTec AD)
> Random seed 1/11/111 세 번 반복 결과 All Avg I-AUROC 99.42±0.05, P-AUROC 98.39±0.01, AUPRO 95.21±0.06 — 본문 Table 1/2의 단일 시드 결과(99.5/98.4/95.2)와 거의 일치, 재현성 확인.

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- **Encoder를 열면 학습이 불안정해질 위험**:
  <mark style="background: #FF5582A6;">논문 스스로 이 우려가 현실임을 인정한다 — 일부 카테고리에서 encoder BN을 train 모드로 두면 불안정해져, 저자들은 해당 카테고리(Toothbrush, Leather, Grid, Tile, Wood, Screw / VisA cashew, pcb1 / OCT2017)에서 BN을 eval 모드로 바꿔 우회했다.</mark> "encoder를 열어도 안전하다"는 주장이 무조건적이지 않고 데이터셋별 완화책이 필요한 조건부 안전이다.
- **Dual-encoder pairing이 완전히 collapse를 막지는 못할 가능성**:
  두 encoder의 초기 표현이 우연히 매우 가까운 도메인이라면(자연 이미지와 target 도메인 격차가 작은 경우) contrastive 신호 자체가 약해져 identical shortcut에 가까워질 위험을 이론적으로 배제하기 어렵다. 논문은 이 경우를 별도로 실험하지 않았다.

### 한계
- <mark style="background: #FF5582A6;">적용 범위: 정상/이상이 "같은 객체의 국소적 결함 유무"로 구분되는 UAD에 초점을 맞추며, 정상/이상이 의미론적으로 아예 다른 OCC(예: 고양이 vs 다른 동물)는 다루지 않는다고 저자가 명시.</mark>
- <mark style="background: #FF5582A6;">학습 불안정성(저자 명시): validation set이 없는 UAD 특성상 마지막 iteration이 loss spike 구간에 걸리는지가 random seed에 좌우된다.</mark> 원인으로 (1) 한 카테고리만 있는 배치에서 특정 채널이 거의 활성화되지 않아 BN 분산이 0에 가까워지는 문제, (2) Adam의 gradient 제곱 과거 추정치 관련 불안정성([50;51])을 지목한다. 완화책(BN 분산을 running_var로 대체, Adam의 gradient momentum·2차 모멘트 상태를 500 iter마다 리셋)으로 MVTec 99.41%/VisA 97.28%까지 회복했지만 <mark style="background: #FF5582A6;">"BN 모드 수동 선택이 불필요한" 완전한 해결에는 이르지 못했다고 자인한다.</mark>
- <mark style="background: #FF5582A6;">Logical anomaly: MVTec LOCO에서 structural anomaly는 GCAD 대비 우수(90.7 vs 80.6)하지만, 부품 누락·배치 오류 같은 logical anomaly는 GCAD(86.0)에 못 미친다(73.4, mean도 82.1 vs 83.3)</mark> — 국소적 재구성 오차 기반이라 "전체적 배치/논리" 이상 포착에는 원천적으로 약함.

### 생각할 점
- <mark style="background: #A6E3A1A6;">"Frozen encoder가 당연하다"는 전제를 깬 이 접근은, UAD를 넘어 다른 self-supervised/frozen-backbone 파이프라인 전반(예: 세그멘테이션의 frozen ViT backbone, few-shot classification의 frozen feature extractor)에도 "왜 얼려야 한다고 생각했는지" 재검토를 촉발하는 사례로 볼 수 있다.</mark>
- Dual encoder를 "frozen + domain-specific" 대신 서로 다른 두 사전학습 도메인(예: ImageNet vs 의료영상 특화)으로 구성하면 domain gap을 더 세밀하게 조정할 수 있을지도 궁금한 지점 — 논문은 frozen encoder를 항상 ImageNet 고정 기준점으로만 사용.

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">[[ReContrast_Dual_Encoder_Contrastive_Reconstruction]]의 hard-normal mining 아이디어(정상 중에서도 재구성이 원래 어려운 영역에 학습을 집중시키는 방식)는, [[Gaussian_Box_Uncertainty_Modeling]]에서 다루는 "라벨 자체가 원래 애매한 영역"을 다루는 방식과 구조적으로 유사하다 — 두 방법 모두 "원래 어려운 것"과 "진짜 이상 신호"를 구분하려는 시도라는 공통점이 있어, uncertainty 관점에서 두 방법을 통합적으로 이해할 여지가 있다.</mark>
- 학습 불안정성 문제(BN·Adam 관련)는 이 논문이 완전히 풀지 못했다고 밝힌 채 남겨둔 지점이므로, Adam의 대안 optimizer나 BN 대신 LayerNorm/GroupNorm으로 이 구조를 다시 학습시켰을 때 안정성이 개선되는지 검증해볼 가치가 있다.

> [!info] 내 메모
> 

# 관련 개념
- [[ReContrast_Dual_Encoder_Contrastive_Reconstruction]] — 이 논문의 핵심 기여인, contrastive learning 요소(global distance, stop-gradient, dual encoder, hard-normal mining)를 feature reconstruction UAD에 결합해 encoder까지 end-to-end로 최적화하는 프레임워크. 기존 문서와 대조·검증 완료, 내용 일치.
- [[1x1_Convolution]] — bottleneck에서 두 encoder의 특징을 각각 project하는 데 사용(Appendix C, "two 1x1 convolutions are used to project the feature map").

# 관련 문서
(아직 이 논문과 직접 비교할 만한 다른 anomaly-detection 노트가 위키에 없음 — 향후 RD4AD, PatchCore, UniAD, [[2024_CVPRW_LogicAL|LogicAL]] 등이 추가되면 비교 문서 작성 검토)

# 읽어볼 만한 논문
- 참고문헌 기반: H. Deng and X. Li, "Anomaly Detection via Reverse Distillation from One-Class Embedding" (CVPR 2022) [2] — 이 논문의 직접적인 baseline이자 출발점(Config.A). ReContrast의 모든 단계적 개선이 이 구조를 기준으로 설명되므로 먼저 읽어야 논문의 논증 흐름이 이해된다.
- 참고문헌 기반: X. Chen and K. He, "Exploring simple Siamese representation learning" (SimSiam, CVPR 2021) [16] — ReContrast가 stop-gradient 아이디어를 직접 차용한 원조 논문. Collapse 방지 메커니즘 자체를 더 깊이 이해하려면 필수.
- 참고문헌 기반: Z. You et al., "A unified model for multi-class anomaly detection" (UniAD, NeurIPS 2022) [9] — multi-class unified 세팅에서 ReContrast가 비교하는 SOTA. 단일 모델로 여러 카테고리를 처리하는 방향은 산업 배포 관점에서 실용적 함의가 크다.
- 참고문헌 기반: P. Bergmann et al., "Beyond dents and scratches: Logical constraints in unsupervised anomaly detection and localization" (GCAD, IJCV 2022) [49] — ReContrast가 유일하게 뒤처지는 logical anomaly 영역에서 특화 설계된 방법. ReContrast의 한계(구조적 vs 논리적 이상)를 보완하는 방향을 구체적으로 이해하는 데 도움.
- 자유 추천(검증 필요): Adam optimizer의 불안정성을 다루는 최신 개선 연구 — 검색 키워드: `Adam optimizer instability large-scale training second moment estimate fix`. ReContrast가 Appendix E에서 스스로 미해결로 남긴 학습 불안정성 문제(원인으로 지목한 [50;51]의 후속 연구)를 확인할 때 참고할 만하다.
