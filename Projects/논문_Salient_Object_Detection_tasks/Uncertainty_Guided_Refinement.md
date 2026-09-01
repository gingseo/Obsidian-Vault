---
pm-task: true
projectId: "paperwiki-salient-object-detection"
parentId:
id: "t-uncertainty_guided_refinement-wehh8t8q35"
title: "Uncertainty-Guided Refinement for Fine-Grained Salient Object Detection"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-04-16"
due:
progress: 0
assignees: []
tags: []
customFields:
  "5t6guexamtck1e8y": 2025
  "njv4e7krmtck1e8z": "IEEE TIP"
subtaskIds: []
dependencies: []
year: 2025
venue: "IEEE TIP"
jcr_quartile: Q1
task: [salient-object-detection]
direction: [novel-approach, improvement]
paper_tags: [paper, salient-object-detection, uncertainty, attention, fine-grained-prediction, dynamic-inference]
source: "Projects/논문_pdf/Salient_Object_Detection/2025_TIP_Uncertainty-Guided-Refinement.pdf"
source_type: personal
createdAt: "2026-08-18T11:00:00.000Z"
updatedAt: "2026-08-31T00:00:00.000Z"
---

#paper #salient-object-detection #uncertainty #attention #fine-grained-prediction #dynamic-inference

> [!quote] 원제
> **Uncertainty-Guided Refinement for Fine-Grained Salient Object Detection**
> Yao Yuan, Pan Gao, Qun Dai, Jie Qin, Wei Xiang — College of Computer Science and Technology, Nanjing University of Aeronautics and Astronautics / School of Computing, Engineering and Mathematical Sciences, La Trobe University, Melbourne, IEEE TIP 2025
> https://doi.org/10.1109/TIP.2025.3557562

# 한 줄 요약
<mark style="background: #FFF3A3A6;">경계 가이던스(boundary guidance) 대신 예측 saliency map 자체에서 결정론적으로 유도한 불확실성 맵으로 attention을 마스킹해, 저채도(unsaturated)·아티팩트 영역을 반복적으로 정제하는 UGRAN(Uncertainty Guided Refinement Attention Network)을 제안한 논문.</mark>

> [!info] 내 메모
> 

# 정리

## 기존 방법의 한계
- **다중 레벨 feature 통합의 비일관성**:
  저수준 feature 자체에 노이즈와 non-salient 객체 정보가 많이 섞여 있어, 단순 집계만으로는 fine-grained 예측에 유의미하게 기여하는 정보가 적다. 또한 서로 다른 스케일에서 집계된 feature 안에서도 salient 정보의 공간적 일관성이 충분히 고려되지 않는다.
- **경계 기반 가이던스의 고정성 한계**:
  경계 정보로 fine-grained 예측을 가이드하는 방법(EGNet, PoolNet, F3Net, VST 등)은 학습·추론 내내 고정된 prior에 의존해, 모델이 실제로 어디를 저채도로 예측하는지에 적응적으로 반응하지 못한다.

## 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Feature enhancement/aggregation**
- PoolNet[33](FPN으로 top-down 신호 전달), MiNet[37](인접 레벨 상호작용, 업/다운샘플링 필요), ICON[38](다양한 kernel로 feature 다양성), BBRF[30](switch-path 디코더).
- **타겟/해결**: 다중 레벨 feature 통합의 비일관성(문제①) — 대부분 feature 집계 경로 설계에 집중할 뿐, 집계된 feature 안에서 salient 정보의 통합 자체는 소홀히 함.

**갈래 2 — Boundary guidance**
- VST[28](멀티태스크 디코더), EGNet[32](salient+boundary feature 별도 추출), PoolNet[33](경계 검출 joint training), F3Net[34](경계 픽셀 가중치), Amulet[10](경계→예측 short connection).
- **타겟/해결**: 경계 기반 가이던스의 고정성 한계(문제②) — non-salient 객체의 경계 정보로 오히려 혼란을 줄 수 있고 고정 prior라 적응 못함.

**갈래 3 — Uncertainty guidance (충분히 탐구 안 됨)**
- <mark style="background: #FFF3A3A6;">UCF[41](불확실 conv feature 학습), ISPRN[42](SOD에 불확실성 맵 최초 도입, 문맥 정보로 혼합), RCSBN[43](불확실 픽셀에 손실 가중치).</mark>
- **타겟/해결**: 경계 기반 가이던스의 고정성 한계(문제②) — 갈래는 존재하지만 불확실성을 문맥 정보로 "섞거나" 손실 가중치로 "반영"하는 암묵적 방식에 머묾, attention을 명시적으로 마스킹하는 방식은 없었음.

**갭**: <mark style="background: #FFF3A3A6;">Boundary guidance(갈래 2)는 non-salient 객체의 경계 정보로 오히려 혼란을 줄 수 있고 고정 prior라 적응 못하며, uncertainty guidance(갈래 3)는 존재하지만 불확실성을 문맥 정보로 섞거나 손실 가중치로 반영하는 암묵적 방식에 머물러, "모델이 실제로 어디에서 헷갈리는지"를 매 순간 명시적으로 반영하는 방법은 없었다.</mark>

## 이 논문이 풀고자 하는 문제
1. 멀티레벨 feature를 집계할 때 서로 다른 공간 스케일에 걸친 salient 정보를 일관성 있게 통합하는 것.
2. 예측된 saliency map에 남는 저채도 영역·그림자·아티팩트를 고정 prior가 아니라 현재 예측에 적응적인 방식으로 감소시키는 것.

**갭 종합**: <mark style="background: #FFF3A3A6;">기존 방법들은 feature 집계 경로 개선(문제 ①)과 경계 기반 가이던스(문제 ②)를 각각 다뤘지만, 두 문제 모두 "모델이 실제로 어디에서 헷갈리는지"를 반영하지 못한다는 공통점이 있다. 이 논문의 통찰은 경계라는 고정된 prior 대신, 예측 saliency map 자체에서 매 순간 다시 계산되는 불확실성을 attention 마스크로 명시적으로 사용해, 모델의 현재 예측 상태에 적응적인 정제를 하는 것이다.</mark>

> [!info] 내 메모
> 

# 해결 방법 요약

| | 문제 ① — 다중 레벨 feature 통합의 비일관성 | 문제 ② — 경계 기반 가이던스의 고정성 한계 |
|---|---|---|
| **해결 방법** | MIA가 상위 레벨의 이미 정제된 전역 salient 표현을 K/V로 저수준 feature를 attention으로 정제하고, SSCA가 저해상도로 다운샘플링한 feature에서 self-attention으로 스케일 간 salient 정보를 통합 | URA가 현재 saliency 예측 S로부터 "0.5(결정 경계)에 가까울수록 불확실"이라는 결정론적 공식으로 매 정제 단계마다 불확실성 맵을 다시 계산해, attention 마스크로 직접 사용 |
| **예상되는 문제점** | SSCA는 local detail 복원을 URA에 위임하는 대신 localization에만 집중하도록 설계되어, MIA·SSCA 단계 자체에는 fine-grained 보정 능력이 없다(구조적으로 URA에 전적으로 의존). | 불확실성이 예측값의 0.5로부터의 거리로만 정의되어, 모델이 극단값으로 강하게 예측했지만 틀린 경우("확신에 찬 오답")는 URA의 정제 대상에서 원천적으로 제외된다(아래 "제안 방법" ③, Discussion 참고). |

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">멀티레벨 feature를 <span style="color:#c0392b; font-weight:bold;">MIA(Multilevel Interaction Attention)</span>→<span style="color:#c0392b; font-weight:bold;">SSCA(Scale Spatial-Consistent Attention)</span> 두 단계로 상호작용·통합해 salient 정보를 정교화한 뒤, 매 정제 단계마다 현재 saliency 예측 자체로부터 결정론적 공식으로 <span style="color:#c0392b; font-weight:bold;">불확실성 맵</span>을 만들고 이를 <span style="color:#c0392b; font-weight:bold;">URA(Uncertainty Refinement Attention)</span>의 마스크로 사용해 저수준 feature와의 상호작용을 불확실 영역에만 강제로 집중시킨다. 계산량은 불확실 영역 비율에 따라 윈도우를 재귀적으로 나누거나 멈추는 ADP(Adaptive Dynamic Partition)로 절감한다.</mark>

## 전체 파이프라인 (Fig. 3 기준)

```
입력 이미지
       │
       ▼
Backbone (ResNet/Res2Net/SwinTransformer)   → F0(최저수준) ~ F4(최고수준), 공간 크기 순차 감소
       │
       ▼
① MIA (F1~F4 각각, F(i+1)을 상위 문맥으로 사용)   → F1^I ~ F4^I  [채널 C=64로 통일]
       │
       ▼
② SSCA (F1^I~F3^I는 concat 후 채널축소 2C→C, F4^I는 그대로) → F1^S~F3^S  [각 SSCA 출력에서 saliency 예측 S1~S3]
       │
       ▼ (S1: 가장 얕은 레벨 예측을 초기 saliency로 사용)
③ URA × 3 (반복, Query=이전 정제 결과, Key/Value=F0)
       F_1^R = F1^S → URA → F_2^R → URA → F_3^R → URA → F_4^R
       │
       ▼
최종 예측 (학습 시 고정 해상도, 추론 시 점진적 업샘플링하며 재적용) → Final Prediction
```

> [!info] 내 메모
> 

### ① Multilevel Interaction Attention (MIA)
- **역할**:
  백본의 멀티레벨 feature(F0~F4)는 레벨마다 특성이 다른데, 단순 집계는 저수준 feature의 노이즈·non-salient 정보를 그대로 끌고 온다. MIA는 상위 레벨의 이미 정제된 전역 salient 표현을 참조해 하위 레벨 feature의 노이즈를 줄인다.
- **구현**:
  각 레벨 feature `Fi`를 채널 attention(GAP→FC→GeLU→FC)으로 먼저 정제(`F̂i`). `F̂i`를 Query, 한 단계 상위 레벨 `F(i+1)`을 Key/Value로 하는 cross-[[Multi_Head_Self_Attention|attention]]을 적용해 residual로 결합. F4(최상위)는 상위 레벨이 없어 채널 attention만 적용. 업/다운샘플링 없이 그대로 attention에 사용해 정보 손실을 피한다.
- **입출력 shape**:
  `Fi ∈ (Ci, Hi, Wi)` → 채널 attention 후 → cross-attention(`F(i+1)`과) → $F_i^I$ `∈ (C=64, Hi, Wi)` (공간 크기 불변, 채널이 C=64로 통일).

```python
# 논문 Eq.(2) 기반. χ,ψ: 1x1 conv/linear, BN: batchnorm
F_hat_i = chi(F_i * psi(GeLU(psi(GAP(F_i)))))          # channel attention으로 정제
Q = psi(F_hat_i)
K = psi(F_{i+1}); V = psi(F_{i+1})                      # 상위 레벨을 K/V로
F_i_I = BN(chi(F_hat_i + Attention(Q, K, V)))           # residual 결합
```

<mark style="background: #FFF9D6A6;">저수준 feature를 그 자체로만 쓰거나 단순히 상위 레벨과 더하던 기존 방식과 달리, MIA는 이미 정제된 상위 레벨의 전역 salient 표현을 K/V로 삼아 저수준 feature가 "어디를 봐야 하는지"를 attention으로 직접 알려주므로 노이즈가 걸러진 채로 활용된다 — "정리" 표의 문제 ①을 직접 겨냥한다(Table III에서 MIA 단독 추가만으로 DUTS MAE .046→.041, Fβw .776→.812).</mark>

> [!info] 내 메모
> 

### ② Scale Spatial-Consistent Attention (SSCA)
- **역할**:
  MIA로 정제된 feature들을 채널 축소(2C→C)로 합친 뒤, 서로 다른 공간 스케일에 걸친 salient 정보를 저비용으로 통합한다. VST류의 단순 self-attention은 스케일 간 정보 불일치를 무시하고 비용도 크다는 문제를 겨냥.
- **구현**:
  `r×r` strided convolution($r=2^{(3-i)}$, F3와 같은 해상도가 되도록)으로 K/V를 저해상도로 만들어 self-attention 수행 — 저해상도 feature가 더 나은 전역 salient 표현을 담고 있다는 가정. 각 SSCA 출력에서 1×1 conv로 saliency 예측을 뽑아 multilevel supervision에 사용.
- **입출력 shape**:
  $F_i^C$ `∈ (C, Hi, Wi)` → `r×r` conv로 저해상도 K/V → self-attention → $F_i^S$ `∈ (C, Hi, Wi)` (공간 크기는 입력과 동일, 값이 전역 문맥 반영해 갱신).

```python
# 논문 Eq.(3) 기반. r = 2**(3-i), 1<=i<=3
F_hat_i_C = BN(chi_{r,stride=r}(F_i_C))                 # r x r conv로 저해상도 K/V 생성
Q = psi(F_i_C); K = psi(F_hat_i_C); V = psi(F_hat_i_C)
F_hat_i_S = F_i_C + Attention(Q, K, V)
F_i_S = BN(chi(F_hat_i_S + chi(GeLU(chi(F_hat_i_S)))))
```

<mark style="background: #FFF9D6A6;">저해상도 K/V로 전역 salient 표현을 저비용으로 얻어 스케일 간 통합을 겨냥하며("정리" 표 문제 ①), local detail 복원은 다음 단계 URA에 위임하고 자신은 localization에만 집중한다(Table VI에서 SSCA가 기존 feature aggregation 기법 SIM·AFM보다 우수).</mark>

> [!info] 내 메모
> 

### ③ Uncertainty Refinement Attention (URA)
- **역할**:
  이전 단계 saliency 예측에서 남는 아티팩트·저채도 영역을 저수준 feature `F0`와의 상호작용으로 채운다. 예측값이 0.5에 가까울수록(결정 경계에 가까울수록) 불확실하다는 결정론적 공식으로 불확실성 맵을 만들고, 이를 attention의 마스크로 사용해 불확실 영역에만 attention이 유효하도록 강제한다.
- **구현**:
  이전 saliency 예측 `S`로부터 `Û = t - |S - t|`(t=0.5), Gaussian smoothing(k=7, σ=1) 후 이진 마스크 `M`(불확실 영역은 0, 확실 영역은 -∞)을 만든다. 정제 대상 feature(Query)와 `F0`(Key/Value)로 masked attention을 수행해 불확실 영역에서만 attention이 유효하게 한다. 이 refinement를 3회 연속 적용. 학습 시 feature map 크기 고정, 추론 시에는 attention의 공간 스케일 불변성을 이용해 점진적으로 업샘플링하며 재적용해 원본 해상도까지 정제.
- **입출력 shape**:
  `S(1,H,W)` → 불확실성 맵 `U(1,H,W)` → 마스크 `M(H,W)` → $F_i^R$`(C,H,W)` + `F0(C,H,W)` → masked attention → $F_{i+1}^R$`(C,H,W)` (3회 반복, 추론 시 매 반복마다 해상도 2배 업샘플).

```python
# 논문 Eq.(4)-(6) 기반. t=0.5(threshold), Gaussian smoothing k=7,sigma=1
U_hat = t - abs(S - t)
U = GaussianConv_k7_s1(U_hat)
M = where(U > 0.01, 0, -inf)                            # 불확실 영역만 attention 허용

Q = psi(F_i_R); K = psi(F0); V = psi(F0)
MaskAttention = softmax(M + Q @ K.T) @ V
F_next_R = BN(chi(F_i_R + MaskAttention))
```

<mark style="background: #FFF9D6A6;">Boundary guidance는 훈련·추론 내내 고정된 prior만 참조해 모델이 실제로 어디를 저채도로 예측하는지 반영 못한다는 것이 "정리" 표 문제 ②의 핵심이었다. URA의 불확실성 맵은 매 단계마다 "현재" 예측 S로부터 다시 계산되어 모델의 실시간 예측 상태에 맞춰 적응적으로 바뀐다. Attention을 아예 마스킹하는 것은 F3Net/RCSBN류의 손실 가중치보다 훨씬 명시적(explicit)이며, Table VII에서 boundary guidance 대비 우위로 검증된다(None .862/.921/.912, Boundary .863/.920/.914, Uncertainty .865/.922/.917, 각 DUTS/ECSSD/HKU-IS Fβw).</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> 불확실성이 "예측값의 0.5로부터의 거리"로만 정의되므로, 모델이 극단값(0 또는 1)으로 강하게 예측했지만 틀린 "확신에 찬 오답"은 불확실 영역으로 잡히지 않아 URA의 정제 대상에서 원천적으로 제외된다 — 저자가 Limitations 절에서 직접 인정하는 한계다.

> [!info] 내 메모
> 

### ④ Adaptive Dynamic Partition (ADP)
- **역할**:
  불확실 영역은 평균적으로 전체 픽셀의 2~5%에 불과해, 전역 attention은 이 작은 비율을 위해 매번 전체 이미지 크기 연산을 하는 셈이라 비효율적이다. ADP는 윈도우 단위로 불확실 영역 비율을 계산해 확실한 영역은 잘게 쪼개고 불확실한 영역은 분할을 멈춘다.
- **구현**:
  윈도우별 불확실 영역 비율 `p`를 계산해, `p < p_t`(=0.2, 확실/sharp)이면 재귀적으로 더 세분화, `p ≥ p_t`(불확실/blur)이면 분할 중단하고 그 윈도우에서 attention 계산. 최소 분할 크기는 입력 이미지의 1/32. `p_t=0`이면 전역 attention, `p_t=1`이면 고정 window attention으로 퇴화하는 일반화된 설계.
- **입출력 shape**:
  입력 feature `(B,C,H,W)` + 불확실성 맵 `(B,1,H,W)` → 재귀적 분할 후 각 윈도우에 attention 적용 → 동일 shape 출력.

```python
# 논문 Algorithm 1 기반 의사코드
def ADP(x, l, u, p_threshold=0.2, minsize):
    h, w = x.shape[2:]
    x_w, l_w, u_w = partition(x), partition(l), partition(u)
    for i in windows:
        p = sum(u_w[i]) / (h * w)
        if p < p_threshold and h > minsize:
            x_w[i] = ADP(x_w[i], l_w[i], u_w[i])        # 재귀적으로 더 세분화(확실 영역)
        else:
            x_w[i] = Att(x_w[i], l_w[i], u_w[i])         # attention 계산(불확실 영역, 분할 중단)
    return reverse(x_w)
```

<mark style="background: #FFF9D6A6;">확실한(sharp) 영역은 잘게 쪼개 계산량을 아끼고, 불확실한(blur) 영역은 분할을 멈춰 문맥을 보존한다 — 균일하게 잘게 쪼개면(Table VIII) 불확실 영역이 파편화되어 정제 효과가 떨어지는데(small partition MAE .04/Fβw .857, random partition .038/.854), ADP는 그 "덩어리"를 보존해(ADP p_t=0.2: MAE .033/Fβw .865) 이를 피한다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> 윈도우 분할의 유연성에 한계가 있어, 형태가 복잡한 객체는 불확실 영역이 여러 윈도우에 걸쳐 조각날 수 있다 — SOC 데이터셋의 shape complexity(SC) 서브셋에서 다른 attribute 대비 상대적으로 부진한 성능(Table II)과 저자들이 직접 연결짓는 한계.

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| Backbone | 이미지 | F0~F4 (5개 레벨) | 멀티레벨 feature 추출 | ResNet/Res2Net/SwinTransformer |
| ① MIA | Fi + F(i+1) | Fi^I (C=64, Hi, Wi) | 상위 레벨 문맥으로 하위 레벨 노이즈 감소 | Channel attention + cross-attention |
| ② SSCA | F1^I~F3^I (concat+채널축소) | Fi^S (C, Hi, Wi) | 스케일 간 salient 정보 통합 | r×r conv 저해상도 K/V + self-attention |
| ③ URA ×3 | 이전 예측 S + F0 | F_i^R (C,H,W), 반복마다 해상도 증가(추론 시) | 불확실 영역 반복 정제 | 결정론적 불확실성 맵 + masked attention |
| ④ ADP | feature + 불확실성 맵 | 동일 shape | URA 계산량 절감 | 재귀적 윈도우 분할(불확실 영역은 분할 중단) |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table I (DUTS-TE, 6개 벤치마크 중 대표)
**표를 보는 법**: Table I은 CNN 기반(위)/Transformer 기반(아래) SOTA를 6개 데이터셋 × 4개 지표(M↓/Eξm↑/Sm↑/Fβw↑)로 비교한다. 열 순서에 주의 — Sm과 Fβw를 혼동하지 않도록 원문 표 순서를 그대로 따랐다.

| 벤치마크 | 지표 | 2위(ICON-R, CNN 최고) | Ours-S (SwinTransformer) |
|---|---|---|---|
| DUTS-TE | M / Eξm / Sm / Fβw | .043 / .914 / .869 / .817 | .022 / .964 / .931 / .906 |

> [!note]- 세부 결과 및 Ablation
> #### Table I — 6개 벤치마크 전체 (DUT-O/DUTS/ECSSD/HKU-IS/PASCAL-S/SOD)
> **보는 법**: 각 데이터셋 열이 M↓/Eξm↑/Sm↑/Fβw↑ 4개 지표를 담고 있다. Ours-R(ResNet50)은 CNN 계열(위쪽 블록) 최하단, Ours-S(SwinTransformer)는 Transformer 계열(아래쪽 블록) 최하단에 위치.
>
> | 데이터셋 | 지표 | ICON-R | Ours-R | Ours-S |
> |---|---|---|---|---|
> | DUT-O | M/Eξm/Sm/Fβw | .057/.884/.844/.767 | .061/.877/.846/.766 | .045/.911/.878/.818 |
> | DUTS | M/Eξm/Sm/Fβw | .043/.914/.869/.817 | .033/.906/.878/.865 | .022/.964/.931/.906 |
> | ECSSD | M/Eξm/Sm/Fβw | .033/.94/.929/.911 | .029/.961/.934/.922 | .02/.975/.95/.946 |
> | HKU-IS | M/Eξm/Sm/Fβw | .026/.962/.924/.91 | .025/.966/.932/.917 | .019/.978/.945/.937 |
> | PASCAL-S | M/Eξm/Sm/Fβw | .055/.906/.861/.818 | .055/.876/.824/.794 | .045/.924/.878/.845 |
> | SOD | M/Eξm/Sm/Fβw | .084/.882/.876/.822 | .084/.864/.814/.776 | .075/.89/.838/.818 |
>
> Ours-S가 6개 데이터셋 모두에서 Fβw(shadow·저채도에 특히 민감한 지표) 기준 이전 SOTA를 앞섬. Ours-R은 DUT-O에서 MAE가 ICON-R보다 근소하게 뒤지지만(.061 vs .057), Fβw는 대부분 데이터셋에서 개선. SwinTransformer backbone(Ours-S)이 전반적으로 우수하며, 다수의 attention 모듈을 포함하고도 44 fps 실시간 추론 속도 유지.
>
> #### Table II — SOC 데이터셋 속성별 성능 (9개 attribute)
> **보는 법**: AC(외형 변화)/BO(큰 객체)/CL(clutter)/HO(이종 객체)/MB(motion blur)/OC(가림)/OV(out-of-view)/SC(형태 복잡)/SO(소형 객체) 속성별 M/Eξm/Sm/Fβw, 17개 CNN 기반 SOTA와 비교.
> Ours-R이 전반적으로 우수하며 특히 motion blur(MB) 속성에서 개선폭이 큼(M .079, Eξm .849, Sm .812, Fβw .732 — blur 처리 능력 직접 검증). 반대로 shape complexity(SC) 속성에서는 상대적으로 부진(M .066, 다른 attribute 대비 개선폭이 작음) — ADP의 영역 분할이 형태가 복잡한 객체에서 파편화를 일으키기 때문으로 저자들은 추정.
>
> #### Table III — 모듈별 기여도 Ablation (ResNet50, DUTS/ECSSD/HKU-IS)
> **보는 법**: Baseline(UNet류)에 MIA→SSCA→URA를 순차 추가하며 3개 데이터셋에서 M/Sm/Fβw 변화 확인.
>
> | 구성 | DUTS(M/Sm/Fβw) | ECSSD(M/Sm/Fβw) | HKU-IS(M/Sm/Fβw) |
> |---|---|---|---|
> | Baseline | .046/.866/.776 | .047/.904/.871 | .037/.907/.866 |
> | +MIA | .041/.887/.812 | .037/.926/.898 | .032/.923/.891 |
> | +MIA+SSCA | .038/.901/.844 | .034/.929/.909 | .031/.926/.897 |
> | +MIA+SSCA+URA | **.033/.906/.865** | **.029/.934/.922** | **.025/.932/.917** |
>
> 세 모듈 모두 단계적으로 기여, URA의 기여폭(특히 Fβw)이 가장 큼.
>
> #### Table IV — MIA vs 기존 Feature Enhancement 기법 (DFA/ASPP/PSP/RFB)
> **보는 법**: +MIA를 기준선으로 각 FEM 대체 시 성능 비교(DUTS/ECSSD/HKU-IS, M/Sm/Fβw).
> +MIA(.041/.887/.812)가 +DFA(.045/.878/.796), +ASPP(.04/.906/.892), +PSP(.041/.919/.888), +RFB(.043/.882/.807), +AIM(.037/.888/.898) 대비 대체로 우수 — AIM만 일부 지표(Fβw)에서 근접하나 종합적으로 MIA가 앞섬.
>
> #### Table V — MIA 상호작용 방향 Ablation
> **보는 법**: F1/F2/F3가 어느 상위 레벨과 상호작용하는지("2\3\4\-" 표기, F1↔F2, F2↔F3, F3↔F4) 조합별 성능.
> 채택안(2\3\4\-, 인접 상위 레벨과 상호작용)이 가장 우수 — 인접 레벨 상호작용이 먼 레벨보다 낫고, 상위→하위 방향이 하위→상위보다 우수함을 확인.
>
> #### Table VI — SSCA vs 기존 Feature Aggregation 기법 (SIM/AFM)
> **보는 법**: +MIA+SSCA를 +MIA+SIM, +MIA+AFM과 비교(DUTS/ECSSD/HKU-IS, M/Sm/Fβw).
> +MIA+SSCA(.038/.901/.844)가 +MIA+SIM(.037/.895/.834), +MIA+AFM(.04/.894/.837) 대비 ECSSD·HKU-IS에서 우수 — attention 기반 전역 모델링이 convolution 기반 집계보다 유리.
>
> #### Table VII — Uncertainty Guidance vs Boundary Guidance
> **보는 법**: URA 입력을 None(마스킹 없음)/Boundary(EGNet 방식 경계 추출)/Uncertainty(제안)로 교체(DUTS/ECSSD/HKU-IS Fβw).
> None .862/.933/.921, Boundary .863/.933(오히려 ECSSD Sm 하락)/.920/.914, Uncertainty(제안) .865/.934/.922/.917 — Boundary guidance는 개선이 미미하거나 일부 지표는 오히려 하락, Uncertainty guidance만 일관되게 개선.
>
> #### Table VIII — ADP Partition 방식 Ablation
> **보는 법**: Small partition(균일 소분할)/Random partition(50% 확률)/ADP(p_t=0.2/0.1/0.4)를 DUTS/ECSSD/HKU-IS에서 비교.
> Small partition .04/.9/.857, Random partition .038/.901/.854, ADP p_t=0.2(채택) .033/.906/.865(최적), p_t=0.1 .035/.902/.86, p_t=0.4 .037/.9/.857 — ADP(p_t=0.2)가 모든 대안보다 우수.
>
> #### Table IX — 손실 함수 가중치 Ablation
> **보는 법**: L_bce 단독/L_api(TRACER 방식)/L_bce+L_iou/L_wbce+L_wiou(F3Net 방식)/채택(L_bce+L_iou+L_sc) 비교.
> 가중 손실(L_api, L_wbce+L_wiou)을 추가하면 오히려 성능 저하 — URA가 이미 충분한 명시적 가이던스를 제공하므로 손실 가중치까지 더하면 복잡한 영역에 과도하게 치우친다고 저자는 해석.

> [!info] 내 메모
> 

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
- <mark style="background: #A6E3A1A6;">[[Gaussian_Box_Uncertainty_Modeling]]과의 대비가 흥미롭다 — 이 논문의 불확실성은 결정론적·비학습(별도 파라미터·loss 없이 예측값 자체의 함수)이고 픽셀 단위인 반면, gaussian-box-uncertainty-modeling은 학습된 분포 파라미터(σ)와 KL loss를 쓰고 박스 단위다. SOD에도 "학습된" 불확실성(saliency map을 Gaussian/Beta 분포로 모델링해 픽셀별 σ를 직접 예측)을 도입하면 "확신에 찬 오답을 못 잡는" 한계를 완화할 수 있을지 검증해볼 가치가 있다 — 학습된 불확실성은 모델의 실제 오차 패턴에서 신호를 얻으므로 다른 종류의 불확실성을 포착할 가능성이 있다.</mark>
- <mark style="background: #A6E3A1A6;">이 논문의 ADP(불확실 영역 비율 기반 재귀적 윈도우 분할)는 SOD 외에 dense prediction 전반(depth estimation, anomaly detection의 픽셀 단위 이상 스코어 맵 등)에서 "불확실/이상 영역에만 비용을 쓰는" 동적 추론 패턴으로 재사용할 수 있어 보인다.</mark>

> [!info] 내 메모
> 

# 관련 개념
- [[Uncertainty_Masked_Refinement_Attention]] — 이 논문이 제안하는 핵심 기법(URA). 예측 saliency map에서 결정론적으로 생성한 불확실성 맵으로 attention을 마스킹해 반복적으로 저채도 영역을 정제하는 방식.
- [[Gaussian_Box_Uncertainty_Modeling]] — 마찬가지로 "불확실성"을 명시적으로 모델링해 학습을 가이드하지만, 대상이 바운딩 박스 좌표 회귀(학습되는 σ + KL divergence)인 반면 이 논문은 픽셀 단위 saliency 값 자체(0.5로부터의 거리)로부터 결정론적으로 불확실성을 유도하고 attention 마스크로 즉시 사용한다는 점에서 메커니즘이 명확히 다르다. 학습 가능한 분포 파라미터도, 별도의 uncertainty loss도 없다.
- [[Multi_Head_Self_Attention]] — MIA의 cross-attention, SSCA의 self-attention, URA의 masked attention 모두 이 기반 연산을 변형해 사용.

# 관련 문서
(현재 위키에 직접 비교할 다른 salient-object-detection 논문 노트가 없어 comparison 문서는 아직 만들지 않음. [[AIMRINet]]과 함께 향후 2편 이상이 쌓이면 Salient Object Detection 비교 문서 신설을 검토.)

# 읽어볼 만한 논문
- 참고문헌 기반: J. Zhao, J.-J. Liu, D.-P. Fan, Y. Cao, J. Yang, and M.-M. Cheng, "EGNet: Edge guidance network for salient object detection" [32] (ICCV 2019) — 이 논문이 대비축으로 삼는 boundary guidance 계열의 대표 논문이자, URA의 ablation(Table VII "Boundary" 설정)에서 경계 추출 방식으로 직접 채택된 baseline. Boundary guidance의 실제 구현과 한계를 이해하는 데 필수적.
- 참고문헌 기반: T. Kim, K. Kim, J. Lee, D. Cha, J. Lee, and D. Kim, "Revisiting image pyramid structure for high resolution salient object detection" (ISPRN) [42] (ACCV 2022) — SOD에 불확실성 맵 생성을 최초로 도입한 논문. 이 논문(UGRAN)이 "불확실성을 문맥 정보로 섞는" 기존 방식과 "attention 마스크로 명시적으로 쓰는" 자신의 방식을 대비할 때 기준으로 삼는 선행 연구라 배경 이해에 도움.
- 참고문헌 기반: B. Cheng, I. Misra, A. G. Schwing, A. Kirillov, and R. Girdhar, "Masked-attention mask transformer for universal image segmentation" (Mask2Former) [52] (CVPR 2022) — URA의 masked attention 계산식(식 5)이 형태적으로 참조한 원조 기법. Mask2Former는 카테고리 마스크 예측에, UGRAN은 반복적 post-processing 정제에 쓴다는 차이를 이해하려면 원조 메커니즘을 먼저 볼 필요가 있음.
- 자유 추천(검증 필요): saliency/segmentation 모델의 confidence calibration(예: temperature scaling)을 다룬 연구 — 검색 키워드: `confidence calibration semantic segmentation temperature scaling`. 위 Discussion에서 제기한 "확신에 찬 오답은 불확실성 맵이 못 잡는다"는 문제를 완화할 수 있는지 검증할 때 참고할 만한 방향.
