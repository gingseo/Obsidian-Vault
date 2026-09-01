---
pm-task: true
projectId: "paperwiki-small-object-detection"
parentId:
id: "t-ig-detr-mmgdtbr6lk"
title: "IG-DETR: Instance-Guided Dynamic Queries for Small Object Detection"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-08-24"
due:
progress: 0
assignees: []
tags: []
customFields:
  "7l8l795xmtcjvlf6": 2026
  "1frf59rymtcjvske": "ICASSP"
subtaskIds: []
dependencies: []
year: 2026
venue: "ICASSP"
jcr_quartile: Q2
task: [small-object-detection]
direction: [improvement]
paper_tags: [paper, small-object-detection, tiny-object-detection, detr, dynamic-query, feature-enhancement, remote-sensing]
source: "Projects/논문_pdf/Small_Object_Detection/2026_ICASSP_IG-DETR.pdf"
source_type: personal
createdAt: "2026-08-24T03:12:00.000Z"
updatedAt: "2026-08-31T00:00:00.000Z"
---

#paper #small-object-detection #tiny-object-detection #detr #dynamic-query #feature-enhancement #remote-sensing

> [!quote] 원제
> **IG-DETR: Instance-Guided Dynamic Queries for Small Object Detection**
> Yuejie Li, Bowen Li, Chengjun Mao — AI Force, Ant Group, ICASSP 2026
> https://doi.org/10.1109/ICASSP59912.2026.11465098

# 한 줄 요약
<mark style="background: #FFF3A3A6;">DQ-DETR의 4단계보다 세분화된 6단계 장면 난이도 분류(HIP)로 query 개수를 정하고, 곱셈 마스킹 대신 덧셈 residual 주입(additive residual injection)으로 encoder feature의 semantic backbone을 보존하면서 고주파 텍스처를 강화하는 IFE 모듈, 그리고 이 강화된 feature에서 "salient seed"를 top-K로 뽑아 query의 content·position을 동시에 생성하는 IGQ 모듈을 결합해, DINO 대비 AP +5.0%p·DQ-DETR 대비도 우위를 보인 짧은 형식(ICASSP)의 dynamic query DETR.</mark>

> [!info] 내 메모
> 

# 정리

## 기존 방법의 한계
- **Quantity dilemma(수량 딜레마)**:
  Transformer 기반 detector는 고정된 query 수 K로 초기화되는데, 이는 감지 가능한 최대 객체 수의 상한이 된다. DINO 같은 강력한 baseline도 900 query로는 수천 개 밀집 인스턴스에 압도되어 대량의 false negative가 발생한다(Fig. 1c).
- **Quality deficit(품질 결핍)**:
  초기 query는 대개 입력 이미지와 무관하게 학습된 임베딩이라, 명시적 공간 사전 정보(spatial prior)가 없어 넓은 feature map 안의 작고 밀집된 객체를 "건초더미에서 바늘 찾기"처럼 찾아야 한다. SAM 같은 foundation model이 개별 인스턴스를 낮은 수준에서 인지할 수 있음에도(Fig. 1b), 이 저수준 인지가 transformer detector의 성공적인 탐지로 이어지지 않는다.

## 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — DETR 기반 구조 개선**
- Deformable DETR: 초기 연산 한계 해결, 멀티스케일 feature 사용 가능하게 함.
- DINO: denoising anchor 개선으로 강력한 baseline 확립 — 여전히 고정 query 수(900).
- Anchor DETR, Conditional DETR, DAB-DETR, DN-DETR: query 형식·수렴 속도 개선 — query "개수"는 다루지 않음.
- **타겟/해결**: Quantity dilemma(문제 ①) — query 형식·수렴은 개선했지만 고정 query 수 문제 자체는 다루지 않는다.

**갈래 2 — Density 기반 dynamic query**
- <mark style="background: #FFF3A3A6;">DQ-DETR(직접 baseline): density map 기반 4단계 이산 분류로 query 개수 조정. static·content-agnostic query 메커니즘 자체를 sparse-to-dense 딜레마의 근본 병목으로 명확히 지목한 것은 DQ-DETR이 처음이나, 분류 세분화·feature 강화 결합 방식에 개선 여지가 남는다. DQ-DETR이 CGFE로 attention 기반 feature 강화를 시도했으나 곱셈 마스킹 방식이라 semantic backbone 정보 손실 위험을 내재한다.</mark>
- **타겟/해결**: Quantity dilemma(문제 ①)와 Quality deficit(문제 ②) 둘 다 — query 개수 조정은 시도했지만 분류 단계가 성기고(4단계), feature 강화도 semantic 정보를 훼손할 위험이 있다.

**갭**: <mark style="background: #FFF3A3A6;">기존 DETR 계열 연구들은 static·content-agnostic query 메커니즘 자체를 항공 영상의 sparse-to-dense 딜레마의 근본 병목으로 명확히 지목하지 않았다. DQ-DETR이 이 문제에 처음 접근했지만, 더 세분화된 난이도 분류나 semantic backbone을 보존하는 feature enhancement까지는 다루지 않는다.</mark>

## 이 논문이 풀고자 하는 문제
1. 장면 복잡도(밀도)를 세밀하게 분류해 query 개수를 이미지별로 적절히 배정하는 것.
2. Query 초기화에 쓰일 encoder feature를 semantic 정보 손실 없이 고주파(tiny object) 정보로 보강하는 것.

**갭 종합**: <mark style="background: #FFF3A3A6;">이 논문은 (1) 더 세분화된 난이도 분류(4→6단계), (2) semantic backbone을 보존하는 additive feature enhancement, (3) salient seed 기반 query 생성이라는 세 가지 지점에서 DQ-DETR 대비 추가 개선 여지를 발견했다는 것이 통찰이다.</mark>

> [!info] 내 메모
> 

# 해결 방법 요약

| | 문제 ① — Quantity dilemma(수량 딜레마) | 문제 ② — Quality deficit(품질 결핍) |
|---|---|---|
| **해결 방법** | HIP이 고해상도 encoder feature로 장면을 6단계(DQ-DETR의 4단계보다 세분화)로 분류해 query 개수를 결정 | IFE가 덧셈 residual 주입(`(1+W)`)으로 spatial·channel saliency prior를 encoder feature에 결합, semantic 정보를 항상 최소 원본 비율로 보존 |
| **예상되는 문제점** | 6단계 분류의 구간 경계값이 논문에 명시되지 않고, 분류 정확도(confusion matrix 등) 자체도 보고되지 않아 "6단계가 4단계보다 낫다"는 주장의 직접 근거가 부족 | `(1+W)` 방식이 배경 영역의 원본 신호도 항상 일정 비율 유지하므로, 곱셈 마스킹 대비 배경 억제력이 약해질 가능성이 있으나 정량 분석 없음 |

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">고해상도 encoder feature로 장면 난이도를 6단계로 분류해 query 개수를 정하고(<span style="color:#c0392b; font-weight:bold;">HIP</span>), 이 난이도 정보를 spatial saliency prior로 변환해 곱셈 마스킹이 아닌 덧셈 residual 주입으로 encoder feature를 보강한 뒤(<span style="color:#c0392b; font-weight:bold;">IFE</span>), 보강된 feature에서 카테고리에 무관한 salient seed를 top-K로 뽑아 query의 content(semantic projection)와 position(coordinate 정제)을 함께 생성한다(<span style="color:#c0392b; font-weight:bold;">IGQ</span>).</mark>

## 전체 파이프라인 (Fig. 2 기준)

```
입력 이미지
       │
       ▼
CNN Backbone (ResNet50) + Deformable Transformer Encoder    → 멀티스케일 encoder feature F_enc^(i), i=1..l
       │
       ▼ (최고해상도 F_enc^(1) 선택)
① HIP (Hierarchical Instance Perception)
   F_enc^(1) → 1×1 conv + dilated conv 계열                  → F_inst (instance perception feature)
   F_inst → 2-layer 분류 head                                 → 6단계 난이도 → query 수 K (300~1500 중 선택)
       │
       ▼
② IFE (Instance-Aware Feature Enhancement)
   F_inst → 1×1 conv로 각 레벨 해상도에 맞춤                    → F_inst^(i), i=1..l
   Spatial: AvgP+MaxP(F_inst^(i)) → 7×7 conv+sigmoid           → W_sp^(i)
   F_sp^(i) = F_enc^(i) ⊗ (1+W_sp^(i))                         → (dual-residual, spatial 보강)
   Channel: MLP(AvgP(F_sp)) + MLP(MaxP(F_sp)) → sigmoid        → W_ch^(i)
   F_tot^(i) = F_sp^(i) ⊗ (1+W_ch^(i))                         → (channel 보강까지 완료)
       │
       ▼
③ IGQ (Instance-Guided adaptive Query Prediction)
   F_tot → flatten                                            → F_flat (b, 256, hw)
   F_flat → Φ_score → TopK(K)                                 → salient seed 인덱스 Ω_K
   F_Ω(seed feature) → P_sem(semantic projection)              → Q_cont (content query)
   B_init(coarse grid) ⊕ Ψ_reg(F_Ω)(offset 회귀)                → Q_pos (position query)
       │
       ▼
Transformer Decoder (Q_cont, Q_pos, K개) + encoder memory      → 예측 헤드
       │
       ▼
출력: K개의 (클래스, 박스) 예측
```

> [!info] 내 메모
> 

### ① Hierarchical Instance Perception (HIP)
- **역할**:
  <span style="color:#c0392b; font-weight:bold;">HIP(Hierarchical Instance Perception)</span>은 이미지의 장면 복잡도(객체 밀도)를 미리 파악해, 디코더에 배정할 query 개수 K를 이미지마다 동적으로 결정하는 모듈이다.
- **구현**:
  가장 고해상도인 encoder feature `F_enc^(1)`을 1×1 conv + 일련의 dilated convolution으로 처리해 넓은 문맥 정보를 담은 instance perception feature map `F_inst`를 생성한다. `F_inst`를 2-layer 분류 head에 통과시켜 이미지를 6단계 이산 난이도로 분류하고, 각 단계는 사전 정의된 query 수(300, 500, 700, 900, ...)에 직접 대응한다. 회귀 대신 분류를 택한 근거는 DQ-DETR과 동일한 논리(AI-TOD-V2의 극단적 객체 수 편차, 1~2267)이지만 구간을 4→6개로 세분화했다.
- **입출력 shape**:
  `F_enc^(1) (b, 256, h₁, w₁)` → `F_inst (b, d_inst, h₁, w₁)` → 분류 head → query 수 `K`(스칼라, 6개 후보 중 하나).

```python
# 논문 §2.2 서술 기반 의사코드
F_inst = dilated_conv_stack(conv1x1(F_enc_1))       # 넓은 문맥 정보 확보
level = classifier_2layer(F_inst)                    # 6단계 이산 분류
K = QUERY_COUNTS[level]                               # {300,500,700,900,...} 중 선택
```

<mark style="background: #FFF9D6A6;">"문제 정의"의 첫 번째 문제(quantity dilemma)를, DQ-DETR보다 더 세밀한 6단계 구간으로 query 수를 배정해 완화한다 — Table 4에서 고정 query 수를 300~1500까지 바꿔가며 비교한 결과 1200에서 정점(30.5)을 찍고 1500에서 오히려 하락하는데, 동적 방식(30.9)이 모든 고정 구성을 능가함을 실험으로 확인해, "세분화된 동적 배정"이 "가장 좋은 단일 고정값"보다도 우월함을 보인다.</mark>

> [!info] 내 메모
> 

### ② Instance-Aware Feature Enhancement (IFE)
- **역할**:
  <span style="color:#c0392b; font-weight:bold;">IFE(Instance-Aware Feature Enhancement)</span>는 HIP이 만든 `F_inst`의 정보를 각 encoder 레벨의 멀티스케일 feature에 주입해, query 초기화에 쓰일 feature의 품질(특히 tiny object의 고주파 텍스처)을 끌어올리는 모듈이다.
- **구현**:
  `F_inst`를 1×1 conv로 각 encoder 레벨 해상도에 맞게 다운샘플링해 `F_inst^(i)`(i=1..l)를 만든다. **Spatial Prior Injection**: `F_inst^(i)`에 average pooling과 max pooling을 적용해 concat 후 7×7 conv + sigmoid로 spatial saliency prior `W_sp^(i)`를 산출하고, 곱셈 마스킹(`F⊗W`) 대신 `F_sp^(i) = F_enc^(i) ⊗ (1+W_sp^(i))` 형태의 **덧셈 residual 주입**을 적용한다. **Channel Context Calibration**: `F_sp^(i)`를 global spatial pooling으로 채널 기술자로 압축한 뒤 공유 MLP(AvgP·MaxP 두 경로 합)로 channel calibration 가중치 `W_ch^(i)`를 만들고, 동일하게 `F_tot^(i) = F_sp^(i) ⊗ (1+W_ch^(i))`로 2차(channel) 보강한다.
- **입출력 shape**:
  `F_inst (b, d_inst, h₁, w₁)` + `F_enc^(i) (b, 256, hᵢ, wᵢ)` → `W_sp^(i) (b, 1, hᵢ, wᵢ)` → `F_sp^(i) (b, 256, hᵢ, wᵢ)` → `W_ch^(i) (b, 256, 1, 1)` → `F_tot^(i) (b, 256, hᵢ, wᵢ)`.

```python
# 논문 Eq.(1)-(4) 기반 의사코드
W_sp = sigmoid(conv7x7(concat(avgpool(F_inst_i), maxpool(F_inst_i))))
F_sp = F_enc_i * (1 + W_sp)                            # ≡ F_enc_i + F_enc_i * W_sp, dual-residual
W_ch = sigmoid(mlp(avgpool(F_sp)) + mlp(maxpool(F_sp)))
F_tot = F_sp * (1 + W_ch)
```

<mark style="background: #FFF9D6A6;">"문제 정의"의 두 번째 문제(quality deficit)를, DQ-DETR/Density-Aware DETR과 유사한 attention 계산식을 쓰되 결합 방식을 곱셈에서 덧셈 residual로 바꿔 해결한다 — 순수 곱셈 마스킹이 attention 가중치가 낮은 영역의 원본 semantic 정보까지 억제해버리는 위험을 원천 차단하며, 저자는 이것이 "tiny object 정보 복원을 위한 gradient 흐름"에 중요하다고 설명한다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> `(1+W)` 형태의 덧셈 residual이 항상 "최소 원본 정보 보존"을 보장하지만, 이는 동시에 배경 영역의 노이즈도 항상 일정 비율 이상 유지된다는 의미다. 논문은 이 트레이드오프(semantic 보존 vs 배경 억제력 약화)를 정량적으로 분석하지 않는다.

> [!info] 내 메모
> 

### ③ Instance-Guided Adaptive Query Prediction (IGQ)
- **역할**:
  <span style="color:#c0392b; font-weight:bold;">IGQ(Instance-Guided adaptive Query prediction)</span>는 HIP이 정한 query 수 K와 IFE로 보강된 feature `F_tot`을 이용해, decoder에 넣을 K개의 content·position query를 이미지 내용에 맞춰 직접 생성하는 모듈이다.
- **구현**:
  DAB-DETR 방식을 따라 query를 semantic content `Q_cont`와 geometric position `Q_pos`로 분리한다. `F_tot`을 flatten한 `F_flat`을 category-agnostic한 scoring 함수 `Φ_score`에 통과시켜 confidence map `S`를 얻고, 단순 threshold가 아니라 top-K개의 "salient seed" 인덱스 `Ω_K`를 추출한다. 선택된 seed feature `F_Ω`로부터, **Semantic Projection**(`P_sem`)으로 `Q_cont`를, coarse grid 좌표를 기준점(`B_init`)으로 삼아 regressor `Ψ_reg`가 예측한 offset을 더하는 **Geometric Rectification**으로 `Q_pos`를 생성한다.
- **입출력 shape**:
  `F_tot (b, 256, h, w)` → flatten `F_flat (b, 256, hw)` → `S (b, 1, hw)` → `Ω_K (b, K)` → `F_Ω (b, K, 256)` → `Q_cont (b, K, 256)` + `Q_pos (b, K, 4)`.

```python
# 논문 Eq.(5)-(6) 기반 의사코드
S = Phi_score(F_flat)
Omega_K = TopK_idx(S, K)
F_Omega = gather(F_flat, Omega_K)

Q_cont = P_sem(F_Omega)                    # Semantic Projection
delta_B = Psi_reg(F_Omega)                  # Geometric Rectification: offset 회귀
Q_pos = B_init + delta_B                    # coarse grid 좌표에 offset을 더해 정제
```

<mark style="background: #FFF9D6A6;">"문제 정의"의 세 번째 문제를, query 생성의 원재료 자체가 이미 고주파 정보로 보강된 `F_tot`이기 때문에 자연스럽게 해결한다 — 저자는 `Q_pos`가 "tiny object 중심에 자연스럽게 정렬"되고 `Q_cont`가 "복원된 텍스처 semantic을 물려받아" decoder의 빠른 수렴을 보장한다고 설명한다. 즉 이 모듈 자체의 새로운 메커니즘이라기보다, HIP+IFE로 이미 만들어진 좋은 재료를 salient seed 선별로 최종 조립하는 역할.</mark>

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| ① HIP | F_enc^(1) (b,256,h₁,w₁) | K (스칼라) | 장면 난이도 6단계 분류 → query 수 결정 | 1×1 conv + dilated conv + 2-layer 분류 head |
| ② IFE | F_inst + F_enc^(i) | F_tot^(i) (b,256,hᵢ,wᵢ) | encoder feature의 고주파 보강(semantic 보존) | Spatial(7×7conv)+Channel(MLP) dual-residual `(1+W)` |
| ③ IGQ | F_tot (flatten) | Q_cont(b,K,256) + Q_pos(b,K,4) | K개 salient seed로 query content·position 생성 | TopK scoring + Semantic Projection + Geometric Rectification |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table 1 (AI-TOD-V2 test, ResNet50, 24 epoch)
**표를 보는 법**: DINO(가장 강한 end-to-end baseline)와 DQ-DETR(직접 능가 대상) 대비 IG-DETR의 개선폭을 확인한다.

| 벤치마크 | 지표 | Before(DINO baseline) | After(IG-DETR) |
|---|---|---|---|
| AI-TOD-V2 | AP | 25.9 | 30.9 (+5.0%p) |
| AI-TOD-V2 | AP_vt / AP_t / AP_s / AP_m | 12.7 / 25.3 / 32.0 / 39.7 | 16.0(+3.3) / 31.2(+5.9) / 37.2(+5.2) / 45.3(+5.6) |
| TinyPerson | AP_tiny50 (전체) | 55.8(DINO) | 60.1 |

> [!note]- 세부 결과 및 Ablation
> #### AI-TOD-V2 전체 비교 (Table 1)
> | 방법 | Epochs | AP | AP50 | AP75 | AP_vt | AP_t | AP_s | AP_m |
> |---|---|---|---|---|---|---|---|---|
> | RetinaNet(CNN) | 12 | 8.9 | 24.2 | 4.6 | 2.7 | 8.4 | 13.1 | 20.2 |
> | Faster R-CNN(CNN) | 12 | 12.8 | 29.9 | 9.4 | 0.0 | 9.2 | 24.6 | 37.0 |
> | RFLA(CNN 최고) | 12 | 25.7 | 58.9 | 18.8 | 9.2 | 25.5 | 30.2 | 40.2 |
> | Deformable-DETR | 50 | 18.9 | 50.0 | 10.5 | 6.5 | 17.6 | 25.3 | 34.4 |
> | DAB-DETR | 50 | 22.4 | 55.6 | 14.3 | 9.0 | 21.7 | 28.3 | 38.7 |
> | DINO | 24 | 25.9 | 61.3 | 17.5 | 12.7 | 25.3 | 32.0 | 39.7 |
> | DQ-DETR | 24 | 30.2 | 68.6 | 22.3 | 15.3 | 30.5 | 36.5 | 44.6 |
> | **IG-DETR(Ours)** | 24 | **30.9(+5.0)** | **69.3** | **23.0** | **16.0** | **31.2** | **37.2** | **45.3** |
> - DQ-DETR 대비도 전 지표에서 근소하지만 일관된 우위(AP +0.7, AP_vt +0.7 등).
>
> #### TinyPerson 비교 (Table 2, 극소형 인물 탐지)
> | 방법 | AP_tiny1_50 | AP_tiny2_50 | AP_tiny3_50 | AP_tiny_50 | AP_small_50 | AP_tiny25 | AP_tiny75 |
> |---|---|---|---|---|---|---|---|
> | FCOS | 0.99 | 2.82 | 6.20 | 3.26 | 20.19 | 13.28 | 0.14 |
> | Adaptive RetinaNet | 27.08 | 52.63 | 57.88 | 46.56 | 59.97 | 69.60 | 4.49 |
> | Faster RCNN-FPN | 30.25 | 51.58 | 58.95 | 47.35 | 63.18 | 68.43 | 5.83 |
> | DINO | 37.5 | 60.2 | 66.5 | 55.8 | 70.1 | 74.8 | 9.2 |
> | **IG-DETR(Ours)** | **42.8** | **64.5** | **70.1** | **60.1** | **73.5** | **78.2** | **13.5** |
> - 가장 작은 크기 구간(tiny1)에서 개선폭이 가장 큼(37.5→42.8, +5.3) — 극소형 객체에 특히 강함을 시사.
>
> #### 모듈별 ablation (Table 3, DINO baseline, AI-TOD-V2 test)
> | HIP | IFE | IGQ | AP | AP_vt | AP_t | AP_s | AP_m |
> |---|---|---|---|---|---|---|---|
> | ✓ | | | 25.9 | 12.7 | 25.3 | 32.0 | 39.7 |
> | ✓ | ✓ | | 28.5 | 13.0 | 28.1 | 35.0 | 44.0 |
> | ✓ | ✓ | ✓ | 30.9 | 16.0 | 31.2 | 37.2 | 45.3 |
> - HIP+IFE(feature 보강)만으로 이미 +2.6 AP, IGQ(동적 query 배정) 추가로 +2.4 AP — 저자는 "동적 query 메커니즘이 가장 실질적인 기여를 제공한다"고 서술.
>
> #### Query 수 ablation (Table 4)
> | Query 수 | AP | AP_vt | AP_t | AP_s | AP_m |
> |---|---|---|---|---|---|
> | 300 | 27.5 | 13.5 | 27.8 | 31.0 | 42.1 |
> | 500 | 28.5 | 14.1 | 28.9 | 33.5 | 43.0 |
> | 700 | 29.2 | 14.8 | 29.6 | 35.0 | 43.8 |
> | 900 | 29.8 | 15.1 | 30.1 | 36.0 | 44.5 |
> | 1200 | 30.5 | 15.8 | 30.9 | 36.9 | 45.1 |
> | 1500 | 30.3 | 15.6 | 30.6 | 36.7 | 44.9 |
> | dynamic(채택) | **30.9** | **16.0** | **31.2** | **37.2** | **45.3** |
> - 고정 300→1200까지 증가하며 AP 상승 후 1500에서 소폭 하락 — 동적 방식이 모든 고정 구성 대비 최고.
>
> #### 분류 vs 회귀 (Table 5, HIP 모듈)
> | 방법 | AP | AP_vt | AP_t | AP_s | AP_m |
> |---|---|---|---|---|---|
> | Baseline(DINO) | 25.9 | 12.7 | 25.3 | 32.0 | 39.7 |
> | Regression | 15.5 | 6.0 | 16.8 | 20.5 | 15.0 |
> | Classification(채택) | 30.9 | 16.0 | 31.2 | 37.2 | 45.3 |
> - 연속적인 query 개수를 직접 회귀하는 방식은 AI-TOD-V2의 극단적인 객체 수 분산(1~2267) 때문에 급락(15.5) — DQ-DETR과 동일한 패턴 재확인.
>
> #### 연산 비용 비교 (Table 6)
> | 방법 | GFLOPs | FPS | AP | AP_vt |
> |---|---|---|---|---|
> | DINO | 205 | 22.0 | 25.9 | 12.7 |
> | DQ-DETR(Paper) | - | - | 30.2 | 15.3 |
> | DQ-DETR(Re-impl.) | 215 | 20.8 | 29.6 | 14.9 |
> | IG-DETR | 222 | 20.1 | **30.9** | **16.0** |
> - IG-DETR이 DQ-DETR 재구현보다 약간 더 무겁지만(GFLOPs +7, FPS -0.7) AP는 더 높음 — 저자가 직접 자신들의 DQ-DETR 재구현 결과(29.6, 논문 기재값 30.2와 차이)를 명시해 공정 비교를 시도한 점이 눈에 띔.

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- IFE의 `(1+W)` 덧셈 residual이 항상 "최소 원본 정보 보존"을 보장하지만, 이는 동시에 배경 영역의 노이즈도 항상 일정 비율 이상 유지된다는 의미 → <mark style="background: #FF5582A6;">논문은 이 트레이드오프(semantic 보존 vs 배경 억제력 약화)를 정량적으로 분석하지 않는다 — 곱셈 마스킹 대비 배경 억제가 더 약해질 가능성을 검증하지 않았다.</mark>
- 6단계 분류가 4단계보다 세밀하지만 경계 구간의 오분류 위험은 여전히 남음 → <mark style="background: #FF5582A6;">DQ-DETR·Density-Aware DETR과 달리 이 논문은 분류 정확도 자체(confusion matrix 등)를 전혀 보고하지 않는다 — "6단계가 4단계보다 낫다"는 주장의 직접적 정량 근거가 없다.</mark>

### 한계
- <mark style="background: #FF5582A6;">ICASSP 형식의 짧은 논문(4페이지 본문)이라 방법론 설명이 상대적으로 간결하고, ablation도 AI-TOD-V2 단일 데이터셋에서만 수행 — TinyPerson에는 ablation이 없음.</mark>
- <mark style="background: #FF5582A6;">6단계 분류의 구간 경계값이 명시되지 않음("300, 500, 700, 900, ...") — DQ-DETR·Density-Aware DETR과 달리 구체적 임계값(예: N≤?)이 논문에 기재되지 않아 재현성이 떨어진다.</mark>
- Table 6에서 저자 스스로 DQ-DETR 재구현 결과(29.6)가 원 논문 기재값(30.2)과 다르다고 명시 — 재현 편차가 있음을 인정하면서도 이 편차의 원인은 분석하지 않음.
- DQ-DETR·Density-Aware DETR 대비 이 논문의 실질적 차별점(6단계 분류, additive residual)이 성능 개선에 각각 얼마나 기여하는지 분리된 ablation이 없음 — "HIP+IFE"가 하나로 묶여 보고되어(Table 3) 두 모듈 각각의 개별 기여를 알 수 없다.

### 생각할 점
- <mark style="background: #A6E3A1A6;">이 논문의 IFE는 [[2024_ECCV_DQ-DETR|DQ-DETR]]의 CGFE, [[2025_JSTARS_Density-Aware-DETR|Density-Aware-DETR]]의 spatial/channel attention과 계산식이 거의 동일하지만 "곱셈 vs 덧셈 residual"이라는 한 가지 설계 선택만 다르다 — 6편의 dynamic query DETR 계열 안에서 가장 미세한 차이로 차별화를 시도한 사례이며, 이 작은 차이가 실제로 유의미한지(Table 3에서 HIP+IFE 조합이 DQ-DETR의 CGFE 단독 기여보다 큰지)는 별도 통제 실험 없이는 확정하기 어렵다.</mark>
- <mark style="background: #A6E3A1A6;">Abstract에서 SAM(Segment Anything Model)을 대조군으로 언급(Fig. 1b)한 것은 이 위키의 다른 dynamic query DETR 논문에는 없는 접근 — "저수준 인지는 되지만 탐지로 이어지지 않는다"는 관찰이 향후 SAM류 foundation model을 query 생성에 활용하는 연구로 이어질 수 있는지 궁금증을 남긴다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">이 논문은 "instance-guided"라는 이름과 달리 실제로는 여전히 전역 밀도 분류(HIP) 기반이며, 개별 인스턴스 단위의 직접적 guidance는 salient seed 선택(top-K)에서만 나타난다 — [[2025_arXiv_PaQ-DETR|PaQ-DETR]]이 이미지 조건부로 공유 패턴의 결합 비율을 조정하는 것과 비교하면(둘 다 "이미지 내용에 따라 무언가를 조정한다"는 점은 같지만), PaQ-DETR의 적응 대상은 query *개수*가 아니라 query *표현* 자체라는 점에서 "instance-guided"라는 이 논문의 명명이 정확히 무엇을 가리키는지 재고할 필요가 있다.</mark>
- <mark style="background: #A6E3A1A6;">Additive residual injection(`1+W`) 방식은 이 위키의 다른 attention 기반 feature 강화 논문들([[2024_TGRS_FFCA-YOLO|FFCA-YOLO]]의 SCAM, [[2025_RSASE_RS-TOD|RS-TOD]] 등)이 대체로 곱셈 마스킹을 쓰는 것과 대조된다 — 곱셈 대신 덧셈 residual을 쓰는 것이 다른 attention 모듈에도 일반적으로 이득이 되는지 검토할 가치가 있다.</mark>

> [!info] 내 메모
> 

# 관련 개념
- [[Density_Guided_Dynamic_Query]] — DQ-DETR·Density-Aware DETR과 동일 계열(density/난이도 기반 query 개수 결정)의 세 번째 사례로 이미 "등장 논문"에 포함되어 있음. 다만 이 논문 고유의 additive residual injection·salient seed selection은 이 개념 문서보다는 IFE/IGQ라는 이 논문 특유의 구현 디테일 성격이 강해 별도 concept으로는 분리하지 않음.

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]] — dynamic query DETR 계열 3번째 사례. DQ-DETR과 가장 직접적으로 비교(Table 1, 6)하며 근소하지만 일관된 우위를 보고.

# 읽어볼 만한 논문
- 참고문헌 기반: Y.-X. Huang, H.-I. Liu, H.-H. Shuai, W.-H. Cheng, "DQ-DETR: DETR with dynamic query for tiny object detection" (ECCV 2024) [26] — 이미 위키에 추가됨: [[2024_ECCV_DQ-DETR|DQ-DETR]]. 이 논문이 Table 1·6에서 가장 직접적으로 비교하는 baseline.
- 참고문헌 기반: X. Dai, Y. Chen, J. Yang, P. Zhang, L. Yuan, L. Zhang, "Dynamic DETR: End-to-end object detection with dynamic attention" (ICCV 2021) [19] — "dynamic"이라는 이름을 공유하지만 attention 메커니즘 자체를 coarse-to-fine으로 동적화한다는 점에서 이 논문(query 개수·내용의 동적화)과 다른 접근. 두 "dynamic"의 차이를 명확히 구분하는 데 유용.
- 참고문헌 기반: Q. Zhou, C. Yu, Z. Wang, F. Wang, "D2Q-DETR: Decoupling and dynamic queries for oriented object detection with transformers" (ICASSP 2023) [18] — Oriented object detection에 dynamic query를 적용한 선행 사례. 이후 처리할 DQA-DETR(oriented object detection)과 비교하며 읽으면 유용할 것으로 예상.
- 자유 추천(검증 필요): SAM(Segment Anything Model) 기반 pseudo-label이나 prior를 DETR query 생성에 활용하는 연구 — 검색 키워드: `SAM segment anything model object query prior DETR detection`. 이 논문이 도입부에서 SAM을 대조군으로만 언급하고 실제로는 활용하지 않는데, 실제로 SAM feature를 query guidance에 쓰는 후속 연구가 있는지 확인할 가치가 있음.
