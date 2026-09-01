---
pm-task: true
projectId: "paperwiki-visual-grounding"
parentId:
id: "t-vgrss-q83oxmdh87"
title: "VGRSS: Datasets and Models for Visual Grounding in Remote Sensing Ship Images"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-08-24"
due:
progress: 0
assignees: []
tags: []
customFields:
  "bm1wp6i4mtck1e92": 2025
  "xy2qmm1smtck1e93": "IEEE Transactions on Geoscience and Remote Sensing (TGRS)"
subtaskIds: []
dependencies: []
year: 2025
venue: "IEEE Transactions on Geoscience and Remote Sensing (TGRS)"
jcr_quartile: Q1
task: [visual-grounding]
direction: [novel-approach, foundational]
paper_tags: [paper, visual-grounding, remote-sensing, ship-detection, multimodal, transformer, dataset, sar]
source: "Projects/논문_pdf/Visual_Grounding/2025_TGRS_VGRSS.pdf"
source_type: personal
createdAt: "2026-08-24T03:48:00.000Z"
updatedAt: "2026-08-31T00:00:00.000Z"
---

#paper #visual-grounding #remote-sensing #ship-detection #multimodal #transformer #dataset #sar

> [!quote] 원제
> **VGRSS: Datasets and Models for Visual Grounding in Remote Sensing Ship Images**
> Yaxiong Chen, Liwen Zhan, Yichen Zhao, Shengwu Xiong, Xiaoqiang Lu — Wuhan University of Technology / Fuzhou University, IEEE TGRS 2025
> https://doi.org/10.1109/TGRS.2025.3562717

# 한 줄 요약
<mark style="background: #FFF3A3A6;">원격탐사 선박 영상에서 자연어 표현으로 특정 선박을 위치시키는 새로운 과제 VGRSS(Visual Grounding for Remote Sensing Ship images)를 정의하고, 자동화된 속성 추출+템플릿 기반 표현 생성으로 두 대규모 벤치마크(광학 RSSVG, SAR SARVG)를 구축한 뒤, 언어 정보로 시각 feature를 융합 이전에 미리 강화하는 LVFE 모듈과 차원 압축 없이 시각-언어 feature를 쌓아 융합하는 VLF 모듈, EIoU 손실을 결합한 Transformer 기반 모델로 자연 이미지 SOTA VG 방법들을 세 데이터셋 모두에서 능가한 논문.</mark>

> [!info] 내 메모
> 

# 정리

## 기존 방법의 한계
- **선박 특화 대규모 벤치마크 부재**:
  원격탐사 VG 연구(RSVG 등)조차 경기장·대형 건물 같은 더 큰 타겟에 집중해, 소형이고 클래스 간 유사도가 높은 선박 영상 특화 대규모 벤치마크가 없다.
- **언어 정보의 융합 단계 국한**:
  기존 VG 방법 대부분은 시각-언어 상호작용을 fusion 모듈에서만 수행해, 언어 정보가 시각 feature 추출 과정 자체에는 관여하지 않는다.
- **융합 시 공간 정보 손실**:
  전통적 방법들은 융합 시 시각 feature의 차원을 압축하는데, 이 과정에서 객체 위치 판별에 중요한 공간 정보가 손실될 수 있다.

## 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — One-stage/Two-stage 자연 이미지 VG**
- One-stage[26-29]: FAOA, LBYL-Net 등 — 자연 이미지 설계라 원격탐사에 그대로 적용 시 성능 열세
- Two-stage[32]: MAttNet 등 사전학습 detector 기반 — 원격탐사와 호환 안 되는 시각 feature가 성능 병목
- **타겟/해결**: 선박 특화 벤치마크 부재(문제①) — 자연 이미지용 설계라 원격탐사 도메인 자체에 부적합하다는 근본적 갭이 있다.

**갈래 2 — Transformer 기반 VG**
- TransVG, VLTVG, QRNet[41,44,45] — 대부분 자연 이미지용, 시각-언어 상호작용을 fusion 모듈에서만 수행
- **타겟/해결**: <mark style="background: #FFF3A3A6;">언어 정보의 융합 단계 국한(문제②)·융합 시 공간 정보 손실(문제③) — Transformer 기반 방법들도 언어 정보를 fusion에서만 활용하고 융합 시 차원을 압축해, 두 문제 모두 미해결로 남는다.</mark>

**갈래 3 — 원격탐사 특화 VG**
- GeoVG/RSVG[4,5](대형 타겟 위주), MGVLF/DIOR-RSVG(대형 원격탐사 타겟에 최적화)
- **타겟/해결**: <mark style="background: #FFF3A3A6;">선박 특화 벤치마크 부재(문제①) — 원격탐사 도메인 자체는 다루지만 경기장·대형 건물 같은 대형 타겟 위주로 설계되어, 소형이고 클래스 간 유사도가 높은 선박에는 그대로 적용하기 어렵다.</mark>

**갭**: <mark style="background: #FFF3A3A6;">기존 원격탐사 VG(갈래 3)는 대형 타겟 위주로 설계되어 선박처럼 작고 클래스 간 유사도가 높으며 클래스 내 다양성이 큰 객체를 다루는 대규모 벤치마크·모델이 없었다. Transformer 기반 방법(갈래 2)을 포함해 기존 방법들은 언어 정보를 융합 단계에서만 활용하고 융합 시 공간 정보를 압축해, 소형 타겟의 정밀한 위치 판별에 불리했다.</mark>

## 이 논문이 풀고자 하는 문제
1. 원격탐사 선박 영상에 특화된 대규모 VG 벤치마크(광학+SAR)를 구축하는 것.
2. 언어 정보를 융합 이전 단계부터 시각 feature 강화에 활용하는 것.
3. 융합 시 시각 feature의 공간 정보를 압축 없이 보존하는 것.

**갭 종합**: <mark style="background: #FFF3A3A6;">기존 원격탐사 VG 연구는 대형 타겟 위주로 설계되어 선박처럼 작고 클래스 간 유사도가 높으며 클래스 내 다양성이 큰 객체를 다루는 대규모 벤치마크·모델이 없었다. 또한 기존 방법들은 언어 정보를 융합 단계에서만 활용하고 융합 시 공간 정보를 압축해, 소형 타겟의 정밀한 위치 판별에 불리했다는 것이 이 논문의 통찰이다.</mark>

> [!info] 내 메모
> 

# 해결 방법 요약

| | 문제 ① — 선박 특화 대규모 벤치마크 부재 | 문제 ② — 언어 정보의 융합 단계 국한 | 문제 ③ — 융합 시 공간 정보 손실 |
|---|---|---|---|
| **해결 방법** | 기존 4개 detection 데이터셋(FAIR1M, CGWX, SAR-Ship-Dataset, DIOR-RSVG)에서 데이터 필터링→속성 자동 추출→템플릿 기반 표현 생성 3단계 파이프라인으로 RSSVG(광학)·SARVG(SAR) 자동 구축 | LVFE 모듈이 multihead self-attention으로 언어 정보를 시각 feature에 융합 이전 단계에서 미리 주입 | VLF 모듈이 차원 압축 없이 시각-언어 feature를 채널 방향으로 stacking한 뒤 학습 가능한 토큰으로 반복 정제 |
| **예상되는 문제점** | 사람 검수 없는 완전 자동 생성이라, 생성된 표현의 자연어다움(naturalness)이 검증되지 않음 | LVFE를 EIoU와만 결합(VLF 없이)하면 오히려 baseline보다 성능이 낮아짐(Table II, −1.35%p) | (VLF 자체의 구조적 문제점은 논문에서 별도로 제기되지 않음) |

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">CNN backbone과 BERT로 시각·언어 feature를 각각 추출한 뒤, <span style="color:#c0392b; font-weight:bold;">LVFE(Language-guided Visual Feature Enhancement)</span> 모듈이 multihead self-attention으로 언어 정보를 시각 feature에 미리 주입해 강화하고, 이 강화된 시각 feature를 원본 언어 feature와 채널 방향으로 concat(차원 압축 없이 공간 정보 보존)한 뒤, <span style="color:#c0392b; font-weight:bold;">VLF(Visual-Linguistic Fusion)</span> 모듈이 학습 가능한 토큰을 쌓아 이 결합 feature 위에서 multihead self-attention으로 반복 정제해 최종 bounding box를 회귀한다.</mark>

## 전체 파이프라인 (Fig. 3, Fig. 4, Fig. 5 기준, 640×640 입력 예시)

```
입력 이미지 (3, 640, 640) + 언어 쿼리 (예: "The large white Dry Cargo Ship")
       │                              │
       ▼                              ▼
CNN Backbone(ResNet) + Visual Transformer   BERT + Language Transformer
       → 고레벨 시각 임베딩 F_v (256, 400)         → 고레벨 언어 임베딩 F_t (768, 20)
       │                              │
       └──────────────┬───────────────┘
                       ▼
① LVFE (언어→시각, 3회 반복 self-attention + residual)   → F_v(256,400) [언어로 강화]
                       │
                       ▼ (원본 F_t 확장 후 채널 방향 concat)
       결합 feature F_vt (512, 400)
                       │
                       ▼
② VLF (학습가능 토큰 T_q를 query로, F_vt를 key/value로 self-attention, 3층 stack)   → T_q (512, 1)
                       │
                       ▼
③ Prediction Head (MLP, 2-layer)              → bounding box (4,)
```

> [!info] 내 메모
> 

### ① Language-guided Visual Feature Enhancement (LVFE)
- **역할**:
  기존 VG 방법이 시각-언어 상호작용을 융합 모듈에서만 수행해 텍스트 정보를 충분히 활용하지 못한다는 문제를 겨냥해, 융합 이전 단계부터 언어 정보로 시각 feature 자체를 강화한다.
- **구현**:
  고레벨 시각 임베딩 `F_v`(256×400)와 고레벨 언어 임베딩 `F_t`(768×20, BERT 출력)를 선형 투영으로 동일 차원 공간(256×20)에 정렬. `F_v`를 query로, `F_t`를 key·value로 하는 [[Multi_Head_Self_Attention|multihead attention]]을 4회 반복 적용해(논문 최적 설정, ablation에서 확인), 매번 residual 연결로 원본 `F_v`에 업데이트를 누적. 언어로 강화된 시각 feature `F_v`(256×400)를 얻은 뒤, 원본 언어 feature `F_t`를 확장(차원 변환·복제·재구성)해 채널 방향으로 concat, 최종 결합 feature `F_vt`(512×400) 생성.
- **입출력 shape**:
  `F_v(256,400)` + `F_t(768,20)` → 선형 투영 후 `F_t(256,20)` → attention 반복(4회) → `F_v(256,400)` → `F_t` 확장 후 concat → `F_vt(512,400)`.

```python
# 논문 Fig.4, 본문 서술 기반 의사코드
q = Linear(F_v)          # (256,400)
k = Linear(F_t); v = Linear(F_t)   # F_t를 (256,20) 공간으로 정렬 후 key/value
for _ in range(4):                  # 4층이 최적(ablation, Table III)
    F_v_prime = MultiHeadAttention(q, k, v)
    F_v = F_v + Linear(F_v_prime)   # residual
F_t_expanded = Expand(F_t)          # 차원 변환+복제+재구성
F_vt = Concat(F_v, F_t_expanded, dim=channel)   # (512, 400), 압축 없이 stacking
```

<mark style="background: #FFF9D6A6;">"정리" 표의 문제 ②(언어 정보가 융합 단계에서만 활용됨)를, 융합 이전에 이미 언어 가이드로 시각 feature 자체를 사전 강화함으로써 해결한다 — Table II ablation에서 LVFE 단독 추가만으로 정확도 57.12%→59.27%(+2.15%p)가 개선되어, "융합 전 언어 가이드 강화"라는 설계가 실제로 유효함을 확인.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> LVFE 레이어 수는 4층이 최적이며(Table III: 3층 83.15%→4층 84.20%), 5층에서는 83.47%로 소폭 하락한다 — 과도한 깊이는 오히려 역효과를 내지만, 그 메커니즘적 원인(과적합, 정보 희석 등)은 논문에서 설명되지 않는다.

> [!info] 내 메모
> 

### ② Visual-Linguistic Fusion (VLF)
- **역할**:
  전통적 방법이 융합 시 시각 feature 차원을 압축해 공간 정보가 손실되는 문제를 겨냥해, 차원 압축 없이 결합 feature를 그대로 self-attention으로 융합한다.
- **구현**:
  무작위 초기화된 학습 가능한 토큰 `T_q`(512×1)를 query로, 결합 feature `F_vt`를 key·value로 하는 multihead self-attention을 반복(3층 stack이 최적, residual 연결) 수행 — `T_q`가 `F_vt`의 모든 위치와 연결을 형성하며 시각·언어 문맥 정보를 점진적으로 집약. 최종 `T_q`는 시각·언어 문맥의 통합 표현이 되어, MLP 기반 prediction head(2-layer, ReLU+linear)에 입력되어 4D bounding box 좌표를 직접 회귀.
- **입출력 shape**:
  `T_q(512,1)` + `F_vt(512,400)` → self-attention 3회 반복 → `T_q(512,1)` (업데이트된 통합 표현) → MLP → `bbox(4,)`.

```python
# 논문 Fig.5, 본문 서술 기반 의사코드
T_q = RandomInit(512, 1)            # 학습 가능한 토큰
for _ in range(3):                   # 3층 stack이 최적(ablation, Table III)
    attn_out = MultiHeadAttention(query=T_q, key=F_vt, value=F_vt)
    T_q = T_q + attn_out             # residual
bbox = MLP_2layer_ReLU(T_q)          # 4D 좌표 직접 회귀
```

<mark style="background: #FFF9D6A6;">"정리" 표의 문제 ③(융합 시 공간 정보 압축 손실)을, 전통적 방법처럼 시각 feature 차원을 줄이는 대신 채널 방향으로 쌓아(stacking) 공간 차원을 그대로 유지한 채 self-attention으로 융합함으로써 해결한다 — Table II에서 VLF 단독 추가는 +3.57%p로 LVFE 단독(+2.15%p)보다 더 큰 개별 기여를 보여, 공간 정보 보존이 특히 소형 선박 위치 판별에 중요함을 시사.</mark>

> [!info] 내 메모
> 

### ③ 손실 함수 — Enhanced IoU (EIoU)
- **역할**:
  기존 smooth L1+GIoU 손실이 IoU 계산 시 교집합/합집합 면적만 고려해 소형·유사 형태 타겟에서 gradient가 충분치 않은 문제를 보완하기 위해, 예측-GT 박스의 대각선·너비·높이 차이까지 명시적으로 반영하는 EIoU 항을 추가.
- **구현**:
  `L = L_smooth-L1 + λ·L_GIoU + λ·L_EIoU` (λ=1). EIoU는 IoU에서 중심점 거리(ρ²/c²), 너비 차이(ρw²/cw²), 높이 차이(ρh²/ch²) 세 페널티 항을 뺀 형태.

```python
# 논문 Eq.(1)-(2) 기반
L_EIoU = IoU(b, b_hat) - (rho**2/c**2 + rho_w**2/c_w**2 + rho_h**2/c_h**2)
L = L_smooth_L1(b, b_hat) + lam * L_GIoU(b, b_hat) + lam * L_EIoU(b, b_hat)   # lam=1
```

<mark style="background: #FFF9D6A6;">Ablation(Table II)에서 EIoU 단독 추가는 +4.74%p — 세 손실·모듈 중 개별 기여가 가장 큼. 다만 LVFE와 EIoU를 함께 쓰면 오히려 −1.35%p(성능 저하)로, 모듈 간 상호작용에 따라 단순 가산적이지 않은 비선형적 효과가 관찰됨(아래 "Discussion" 참고).</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> LVFE와 EIoU만 결합하고 VLF가 빠지면 오히려 baseline보다 낮은 성능(55.77%, −1.35%p)을 보인다(Table II row f) — 논문은 이 음의 상호작용의 원인을 분석하지 않는다.

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| 시각/언어 인코딩 | 이미지 + 텍스트 | F_v(256,400) + F_t(768,20) | 각 모달리티 feature 추출 | ResNet+Visual Transformer / BERT+Language Transformer |
| ① LVFE | F_v + F_t | F_vt(512,400) | 언어로 시각 feature 사전 강화 + 압축 없는 결합 | Cross-attention(4층) + concat |
| ② VLF | T_q(512,1) + F_vt(512,400) | T_q(512,1) | 공간 정보 보존한 채 시각-언어 통합 표현 생성 | Self-attention(3층 stack) |
| ③ Prediction Head | T_q(512,1) | bbox(4,) | 최종 박스 좌표 회귀 | MLP(2-layer, ReLU+Linear) |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table I (RSSVG, SARVG, DIOR-RSVG 세 데이터셋, test-accu/mIoU 기준)
**표를 보는 법**: 각 데이터셋에서 val-accu/test-accu/mIoU 세 지표를 비교하며, 각 데이터셋 열마다 이전 최고 baseline이 다르다(RSSVG·SARVG는 TransVG, DIOR-RSVG는 RSVG가 mIoU 기준 최고).

| 벤치마크 | 지표 | 이전 SOTA | Ours(VGRSS) |
|---|---|---|---|
| RSSVG | test-accu / mIoU | 57.12(TransVG) / 51.08(TransVG) | 66.16(+9.04%p) / 56.91(+5.83%p) |
| SARVG | test-accu / mIoU | 95.82(VLTVG 아님, TransVG) / 82.57(TransVG) | 95.94(+0.12%p) / 83.51(+0.94%p) |
| DIOR-RSVG | test-accu / mIoU | 75.19(TransVG=RSVG 동률) / 68.04(RSVG) | 83.01(+6.23%p 대비 val 기준) / 74.85(+6.81%p) |

> [!note]- 세부 결과 및 Ablation
> #### Table I — 전체 SOTA 비교 (RSSVG/SARVG/DIOR-RSVG, val-accu/test-accu/mIoU)
> **보는 법**: 각 방법이 어느 데이터셋까지 평가됐는지(원격탐사 VG 특화 방법 RSVG는 DIOR-RSVG만 평가) 확인하며 비교.
>
> | 방법 | Venue | RSSVG (val/test/mIoU) | SARVG (val/test/mIoU) | DIOR-RSVG (val/test/mIoU) |
> |---|---|---|---|---|
> | FAOA | ICCV'19 | 31.85/31.07/27.44 | 91.64/90.91/73.77 | 51.23/49.39/40.65 |
> | ReSC | ECCV'20 | 52.81/55.95/47.63 | 93.47/93.75/75.20 | 70.72/69.73/57.31 |
> | LBYL-Net | CVPR'21 | - | - | 73.29/-/65.86 |
> | TransVG | ICCV'21 | 57.31/57.12/51.08 | 95.67/95.82/82.57 | 76.78/75.19/65.81 |
> | VLTVG | CVPR'22 | - | - | 69.41/-/59.96 |
> | RSVG | TGRS'23 | - | - | 76.78/75.19/68.04 |
> | **Ours(VGRSS)** | - | **66.28/66.16/56.91** | **96.53/95.94/83.51** | **83.68/83.01/74.85** |
>
> RSSVG(선박, 광학)에서 가장 큰 개선폭 — val-accu +8.97%p, test-accu +9.04%p, mIoU +5.83%p(TransVG 대비). 선박이라는 소형·유사 클래스 타겟에서 이 방법의 강점이 가장 두드러짐. SARVG는 이미 baseline 성능이 높아(95%대) 개선폭이 상대적으로 작음(+0.12%p test-accu) — SAR 이미지 자체의 특성(단순한 형태, 명확한 대비)상 여지가 적었을 가능성. DIOR-RSVG는 RSVG 대비 test-accu +6.23%p·mIoU +6.81%p 개선.
>
> #### Table II — 메인 Ablation (RSSVG test)
> **보는 법**: LVFE/VLF/EIoU를 하나씩 또는 조합으로 추가하며 baseline(57.12%) 대비 정확도 변화(%) 확인.
>
> | LVFE | VLF | EIoU | Accuracy | Change |
> |---|---|---|---|---|
> | - | - | - | 57.12 | - |
> | ✓ | - | - | 59.27 | +2.15 |
> | - | ✓ | - | 60.69 | +3.57 |
> | - | - | ✓ | 61.86 | +4.74 |
> | ✓ | ✓ | - | 64.73 | +7.61 |
> | ✓ | - | ✓ | 55.77 | −1.35 |
> | - | ✓ | ✓ | 59.45 | +2.33 |
> | ✓ | ✓ | ✓(전체) | **66.16** | **+9.04** |
>
> LVFE+EIoU 조합만 유일하게 baseline보다 낮음(음의 상호작용) — VLF 없이 LVFE와 EIoU만 결합하면 오히려 방해가 됨을 시사, 세 모듈 모두 함께 있어야 최대 시너지.
>
> #### Table III — LVFE/VLF 레이어 수 Ablation (DIOR-RSVG)
> **보는 법**: LVFE와 VLF 층 수를 각각 바꿔가며 정확도 변화 확인.
> LVFE 3→4층: 83.15→84.20(+1.05%p, 최적) → 5층: 83.47(소폭 하락, 이때 VLF는 4층). VLF 3층이 최적(84.20%, LVFE 4층 기준), 4층으로 늘리면 82.58(−0.57%p) — 두 모듈 모두 과도한 깊이는 오히려 역효과.
>
> #### Fig. 6 — 정성 분석 (LVFE 적용 전후 attention heatmap)
> **보는 법**: 같은 쿼리에 대해 LVFE 적용/미적용 시 attention이 어디에 집중되는지 비교.
> LVFE 적용 시 "오른쪽 위 선박", "왼쪽 아래 중형 정부 선박" 같은 위치·크기 속성이 명시된 쿼리에서 해당 영역에 attention이 뚜렷하게 집중되는 반면, 미적용 시 attention이 분산됨.

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- LVFE와 EIoU를 함께 쓰되 VLF가 빠지면 성능이 baseline보다 낮아짐(Table II, −1.35%p) → <mark style="background: #FF5582A6;">논문은 이 음의 상호작용의 원인을 분석하지 않는다 — LVFE가 만든 강화된 feature 표현이 VLF의 공간 보존 융합 없이 EIoU의 정밀한 기하 손실과 결합될 때 왜 학습이 오히려 방해받는지 설명이 없다.</mark>
- LVFE 레이어·VLF 레이어 모두 특정 깊이(4층/3층) 이후 성능이 정체·하락 → <mark style="background: #FF5582A6;">과도한 깊이가 왜 역효과를 내는지(과적합, 정보 희석 등) 메커니즘적 설명이 제시되지 않는다.</mark>

### 한계
- <mark style="background: #FF5582A6;">저자가 Conclusion에서 직접 명시: "향후 연구는 원격탐사 선박 영상의 특성에 더 잘 맞도록 VGRSS를 더 정교화하는 데 초점을 맞출 것" — 현재 버전이 아직 완성형이 아니라는 것을 인정.</mark>
- <mark style="background: #FF5582A6;">데이터셋 구축이 완전히 자동화되어 있어(사람 검수 없는 템플릿 기반 생성), 생성된 표현의 자연어다움(naturalness)이나 실제 사용자 질의와의 유사성이 검증되지 않는다 — RefCOCO류처럼 사람이 직접 작성한 표현과 비교했을 때의 표현 다양성·품질 차이가 논의되지 않음.</mark>
- Two-stage 방법(사전학습 detector 기반)과의 비교가 Table I에 포함되지 않아, One-stage/Transformer 계열에서만 비교가 이뤄짐 — 저자도 "기존 detector의 원격탐사 도메인 비호환성"을 two-stage의 한계로 지적하지만 실제 정량 비교는 제시하지 않는다.
- SARVG에서의 개선폭이 매우 작아(test-accu +0.12%p), 제안 모듈이 SAR 이미지라는 다른 센서 도메인에서는 상대적으로 기여가 제한적일 가능성이 있으나 이에 대한 원인 분석은 없다.

### 생각할 점
- <mark style="background: #A6E3A1A6;">이 논문은 이 위키에서 완전히 새로운 task(visual grounding)를 여는 첫 사례로, "자연어로 특정 객체를 지목한다"는 문제는 이 위키의 다른 small object detection 논문들이 다루는 "모든 객체를 빠짐없이 찾는다"는 문제와 근본적으로 다른 목표(특정성 vs 완전성)를 가진다 — 그럼에도 "소형·유사 클래스 객체를 어떻게 구별할 것인가"라는 하위 문제는 공유한다.</mark>
- <mark style="background: #A6E3A1A6;">LVFE의 "언어로 시각 feature를 융합 이전에 미리 강화한다"는 설계는, "정제 대상이 되는 feature를 미리 보강해둔다"는 상위 전략을 다른 신호(언어)로 구현한 사례로 볼 수 있다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">VGRSS의 자동화된 표현 생성 파이프라인(속성 자동 추출+템플릿 기반 문장 생성)은, [[2026_Neural-Networks_YOFOR|YOFOR]]가 다루는 long-tailed 클래스 불균형 문제와 결합할 여지가 있다 — 희소 클래스 선박에 대해 더 다양한 표현 템플릿을 생성해 데이터 증강 효과까지 노릴 수 있을 것으로 보인다.</mark>
- <mark style="background: #A6E3A1A6;">EIoU 손실(대각선+너비+높이 차이를 명시적으로 반영)은 [[2025_JSTARS_Density-Aware-DETR|Density-Aware-DETR]]이 쓰는 anchor L1(log-ratio) 손실과 마찬가지로 "표준 IoU/L1이 소형 객체에서 충분한 gradient를 주지 못한다"는 문제의식을 공유한다 — 두 손실을 직접 비교하면 소형 객체 박스 회귀에 어떤 기하학적 보정이 가장 효과적인지 밝힐 수 있을 것으로 보인다.</mark>

> [!info] 내 메모
> 

# 관련 개념
- [[Language_Guided_Pre_Fusion_Feature_Enhancement]] — 이 논문의 LVFE 핵심 기여. 언어 정보로 시각 feature를 융합 이전 단계에서 미리 강화하는 기법.
- [[Multi_Head_Self_Attention]] — LVFE의 언어→시각 attention, VLF의 학습가능 토큰 기반 self-attention 모두 이 기반 연산을 사용.

# 관련 문서
- 비교: (아직 없음 — 이 위키에서 visual grounding을 다룬 첫 논문이라 비교 대상이 없음)

# 읽어볼 만한 논문
- 참고문헌 기반: Y. Zhan, Z. Xiong, Y. Yuan, "RSVG: Exploring data and models for visual grounding on remote sensing data" (IEEE Trans. Geosci. Remote Sens. 2023) [4] — DIOR-RSVG 데이터셋과 MGVLF 모델의 원조. 이 논문의 세 벤치마크 중 하나(DIOR-RSVG)의 출처이자, 원격탐사 VG 분야의 직접적인 선행 연구로 우선순위가 매우 높음.
- 참고문헌 기반: Z. Deng, Y. Yang, T. Chen, Z. Zhou, H. Li, "TransVG: End-to-end visual grounding with transformers" (ICCV 2021) [41] — 이 논문의 모델이 시각-언어 Transformer 구조 전체를 계승하는 원조 end-to-end Transformer VG 네트워크.
- 참고문헌 기반: L. Yang, Y. Xu, C. Yuan, W. Liu, B. Li, W. Hu, "Improving visual grounding with visual-linguistic verification and iterative reasoning" (VLTVG, CVPR 2022) [44] — 이 논문이 language-guided visual feature aggregation과 multilevel cross-modal decoder를 직접 비교하는 강력한 Transformer 기반 baseline.
- 자유 추천(검증 필요): SAR과 광학 두 센서 도메인을 함께 다루는 멀티모달 원격탐사 visual grounding/retrieval 후속 연구 — 검색 키워드: `SAR optical multimodal visual grounding remote sensing cross-sensor 2025 2026`. SARVG에서의 개선폭이 작았던 이유를 다른 SAR 특화 연구와 비교해 이해하는 데 도움될 것으로 예상.
