---
pm-task: true
projectId: "paperwiki-object-detection"
parentId:
id: "t-detr-j94vuetv7h"
title: "DETR"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-08-12"
due:
progress: 0
assignees: []
tags: []
customFields:
  "nh3oelhxmtcnb377": 2020
  "gx1mmrf0mtcnb37a": "ECCV"
subtaskIds: []
dependencies: []
year: 2020
venue: "ECCV"
jcr_quartile: Q1
task: [object-detection]
direction: [foundational, novel-approach]
paper_tags: [paper, object-detection, transformer, set-prediction, bipartite-matching, end-to-end, panoptic-segmentation]
source: "/Users/GyeongSeo/Workspace/논문_pdf/Small_Object_Detection/2020_ECCV_DETR.pdf"
createdAt: "2026-08-18T11:00:00.000Z"
updatedAt: "2026-08-28T16:40:00.000Z"
---

Project: [[논문_Object_Detection|Object Detection]]
#paper #object-detection #transformer #set-prediction #bipartite-matching #end-to-end #panoptic-segmentation

> [!quote] 원제
> **End-to-End Object Detection with Transformers**
> Carion, Massa, Synnaeve, Usunier, Kirillov, Zagoruyko — Facebook AI, ECCV 2020

# 한 줄 요약
<mark style="background: #FFF3A3A6;">Object detection을 "집합(set) 예측 문제"로 재정의해, CNN backbone + Transformer encoder-decoder + bipartite matching loss만으로 anchor·NMS 같은 수작업 컴포넌트 없이 박스와 클래스를 한 번에(end-to-end) 예측하는 DETR.</mark>

> [!info] 내 메모
> 

# 정리

| | 문제 ① — 수작업 컴포넌트 의존 | 문제 ② — 중복 제거를 위한 후처리 의존 |
|---|---|---|
| **문제 정의** | 기존 detector는 anchor box, grid cell, region proposal처럼 "사람이 미리 설계한 위치 후보"를 기준으로 예측하고, 중복 예측은 NMS(비최대 억제)로 사후에 걸러낸다. 이 설계 자체가 사람의 사전 지식(prior knowledge)을 인코딩한 것이라 파이프라인이 복잡해진다. | 같은 객체를 여러 proposal/anchor가 동시에 맞히면 중복 예측이 생기는데, 이를 NMS라는 미분 불가능한 휴리스틱 후처리로 걸러낸다. NMS는 학습 과정에 포함되지 않는 별도 단계이며, 임계값 등 하이퍼파라미터에 성능이 민감하다. |
| **풀고자 하는 문제** | Anchor/proposal 같은 사람이 설계한 초기 추측 없이, 이미지에서 곧바로 최종 박스 집합을 예측하는 것 | NMS 같은 미분 불가능한 후처리 없이, 학습 과정 자체에서 중복 예측을 억제하는 것 |
| **선행 연구 접근** | - Two-stage(Faster R-CNN 등): region proposal 생성 후 분류·회귀<br>- Anchor 기반(RetinaNet 등): 격자마다 여러 크기·비율의 anchor 배치<br>- Anchor-free(CenterNet, FCOS 등): grid cell/center point 기준 예측<br>- **갭**: 방식은 달라도 전부 "초기 추측(proposal/anchor/center)"을 사람이 정한 규칙으로 만들고, 그 규칙에 성능이 크게 좌우된다. | - 표준 NMS: IoU 임계값 기준으로 낮은 confidence 박스 제거<br>- Learnable NMS[16], Relation Networks[17]: attention으로 예측 간 관계를 명시적으로 모델링해 NMS 의존도 완화 시도<br>- **갭**: 관계를 모델링하더라도 proposal 좌표 같은 추가적인 손수 설계한(hand-crafted) 문맥 정보에 의존하거나, 여전히 후처리가 필요. |
| **해결 방법** | Bipartite matching(헝가리안 알고리즘)으로 예측-정답을 1:1 매칭 — 이 매칭 자체가 "무엇이 무엇을 담당할지"를 매번 데이터로부터 계산하므로, anchor처럼 미리 정해둔 위치 규칙이 필요 없다. | 같은 매칭이 정답당 예측을 정확히 하나만 배정하므로, 나머지 중복 예측은 자동으로 "no object"에 매칭되어 억제된다 — 별도 후처리 없이 학습 과정 자체에서 중복이 억제된다. |
| **예상되는 문제점** | 매칭을 매 스텝 다시 계산하므로 학습 초반 신호가 불안정해, 매우 긴 학습 스케줄(300~500 epoch)이 필요하다(아래 "제안 방법" ⑦ 참고). | 이 억제는 디코더의 self-attention(query끼리 상호 참조)에 의존하는데, object query 개수(N=100)가 고정이라 실제 객체 수가 이를 초과하면 애초에 억제할 예측 자체가 부족해진다(아래 "제안 방법" ⑤, "실험 결과" Fig.12 참고). |

**갭 종합**: <mark style="background: #FFF3A3A6;">"초기 추측을 사람이 설계"하는 문제와 "중복을 후처리로 제거"하는 문제는 서로 다른 단계처럼 보이지만, 둘 다 "예측과 정답을 어떻게 대응시킬지"를 사람이 정한 규칙에 의존한다는 공통 원인에서 나온다. 이 대응 규칙 자체를 학습 가능한 알고리즘(bipartite matching)으로 대체하면 두 문제를 동시에 없앨 수 있다는 것이 DETR의 통찰이다.</mark>

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">이미지를 CNN으로 압축한 뒤 Transformer encoder-decoder에 통과시켜, <span style="color:#c0392b; font-weight:bold;">"object query"라 부르는 고정된 N개의 학습된 슬롯이 각자 하나의 객체(또는 "no object")를 담당하도록</span> 병렬로 디코딩하고, <span style="color:#c0392b; font-weight:bold;">bipartite matching(헝가리안 알고리즘)으로 예측-정답을 1:1 매칭한 뒤 그 매칭에 대해서만 손실을 계산</span>해 anchor·NMS 없이 학습한다.</mark>

## 전체 파이프라인 (Fig. 1, Fig. 2 기준)

```
입력 이미지 (3, H₀, W₀)
       │
       ▼
① CNN Backbone (ResNet-50/101)              → (2048, H, W)      [H, W = H₀/32, W₀/32]
       │
       ▼
② 1×1 Conv (채널 축소)                        → (256, H, W)
       │
       ▼
③ Positional Encoding 추가 + Flatten          → (HW, 256)
       │
       ▼
④ Transformer Encoder (self-attention × 6층)  → (HW, 256)        [encoder memory]
       │
       ▼
⑤ Transformer Decoder (self-attn + cross-attn × 6층, object query 100개 입력) → (100, 256)
       │
       ▼
⑥ Prediction FFN (클래스 + 박스, 공유 가중치)   → 클래스(100, K+1) + 박스(100, 4)
       │
       ▼
출력: 100개의 (클래스, 박스) 예측
       │
       ▼ (학습 시에만)
⑦ Bipartite Matching + Hungarian Loss         → 1:1 매칭 σ, 스칼라 loss
```

> [!info] 내 메모
> 

### ① CNN Backbone
- **역할**: 원본 이미지의 저수준 픽셀 정보를 압축해, 객체를 구별하는 데 필요한 고수준 semantic feature map으로 변환한다.
- **구현**: ImageNet 사전학습된 ResNet-50 또는 ResNet-101(마지막 분류 레이어 제거, conv 레이어만 사용).
- **입출력 shape**: 입력 `(3, H₀, W₀)` → 출력 `(C=2048, H, W)`, 여기서 `H=H₀/32, W=W₀/32` (32배 다운샘플링).

```python
# self.backbone: torchvision resnet50(pretrained=True)에서 마지막 두 레이어(avgpool, fc) 제거
x = self.backbone(inputs)          # (3, H0, W0) -> (2048, H, W)
```

> [!info] 내 메모
> 

### ② 1×1 Conv — 채널 압축
- **역할**: ResNet 출력 채널(2048)이 트랜스포머가 다루기엔 너무 크므로, 트랜스포머의 작업 차원 `d`(보통 256)로 줄인다. 공간 정보는 건드리지 않고 채널만 재조합한다 — 자세한 동작은 [[1x1_Convolution]] 참고.
- **입출력 shape**: `(2048, H, W)` → `(d=256, H, W)`.

```python
self.conv = nn.Conv2d(2048, hidden_dim, 1)   # kernel_size=1
h = self.conv(x)                              # (2048, H, W) -> (256, H, W)
```

> [!info] 내 메모
> 

### ③ Positional Encoding 추가 + Flatten
- **역할**: Transformer는 입력 순서를 구별하지 못하는 permutation-invariant 구조이기 때문에, "이 feature가 이미지의 어느 위치였는지"를 명시적으로 알려주는 벡터(sine/cosine 기반 2D positional encoding)를 각 위치의 feature에 더해준다. 그 다음 2차원 공간(H, W)을 1차원 시퀀스로 펼친다(flatten).
- **입출력 shape**: `(d, H, W)` + positional encoding → flatten → `(HW, d)` — 즉 "HW개의 토큰, 각 토큰은 d차원 벡터"인 시퀀스로 변환. 이 시퀀스가 인코더의 입력이 된다.

```python
# row_embed, col_embed: 학습되는 위치 임베딩 (H, d/2), (W, d/2)
pos = torch.cat([
    col_embed[:W].unsqueeze(0).repeat(H, 1, 1),   # (H, W, d/2)
    row_embed[:H].unsqueeze(1).repeat(1, W, 1),   # (H, W, d/2)
], dim=-1).flatten(0, 1).unsqueeze(1)             # (HW, 1, d)
src = h.flatten(2).permute(2, 0, 1)               # (256, H, W) -> (HW, 1, 256)
```

> [!info] 내 메모
> 

### ④ Transformer Encoder
- **역할**: HW개의 이미지 패치(토큰)들이 서로의 정보를 참고해, "이 근처에 객체가 있다/여기는 배경이다" 같은 전역적 문맥을 반영한 feature로 업데이트된다. Multi-head self-attention의 정확한 동작은 [[Multi_Head_Self_Attention]] 참고.
- **구조**: self-attention + FFN 블록을 6층 쌓음(각 층마다 Add&Norm 포함).
- **입출력 shape**: `(HW, d)` → `(HW, d)` (개수·차원 불변, 값만 문맥을 반영해 갱신됨). 이 최종 출력을 "encoder memory"라 부르며 디코더의 cross-attention에 재사용된다.
- <mark style="background: #FFF9D6A6;">인코더의 self-attention이 이미지 전체를 한 번에 보기 때문에, 서로 멀리 떨어진 두 객체도 한 층 만에 "겹치는 객체인지 아닌지"를 구별할 수 있게 된다 — 이것이 "정리" 표의 초기 추측(anchor) 없이도 객체 후보를 전역적으로 분리해내는 근거가 된다(ablation에서 인코더 층 제거 시 AP가 3.9 하락, Fig.3의 attention map이 인스턴스를 실제로 분리함을 시각적으로 보여줌).</mark>

```python
memory = encoder_layers(src + pos)   # (HW, 1, 256) -> (HW, 1, 256), 6층 반복
```

> [!warning] 이 구조 때문에 예상되는 문제점
> self-attention의 연산량은 토큰 개수의 제곱, 즉 `O((HW)²)`에 비례한다([[Multi_Head_Self_Attention]] 참고). 이미지 해상도가 커지면 `HW`가 빠르게 늘어나므로, 고해상도 입력(특히 소형 객체를 잘 보려고 해상도를 올리는 경우)일수록 인코더 비용이 급격히 커진다 — 실제로 이 논문의 DC5 변형(dilation으로 해상도를 4배 키움)은 성능은 오르지만 인코더 self-attention 비용만 16배, 전체 연산량은 약 2배로 증가한다고 논문이 직접 밝히고 있다.

> [!info] 내 메모
> 

### ⑤ Transformer Decoder — Object Query
- **역할**: <span style="color:#c0392b; font-weight:bold;">object query</span>라 부르는 **학습되는(learned) 위치 임베딩 N개(=100개)**를 디코더 입력으로 사용한다. 각 query 슬롯은 학습이 끝나면 "이미지 중앙의 큰 객체", "우측 하단의 작은 객체" 식으로 암묵적인 역할을 갖게 되고(Fig.7에서 실제로 확인됨), 최종적으로 100개의 (클래스, 박스) 예측 후보를 만든다.
- **구조**: 각 디코더 층은 (1) query들끼리의 self-attention → (2) query가 encoder memory를 참고하는 cross-attention → (3) FFN, 순서로 구성되며 이 블록을 6층 쌓는다. 6층 전부의 출력에 보조 손실(auxiliary loss)을 걸어 학습을 돕는다.
- **입출력 shape**: object query `(N=100, d)` (처음엔 학습된 초기값, self-attention 통과 시마다 갱신) + encoder memory `(HW, d)` → 디코더 출력 `(N=100, d)`.
- <mark style="background: #FFF9D6A6;">디코더의 self-attention(query끼리 서로 참고)이 "다른 슬롯이 이미 이 객체를 담당하고 있다"는 정보를 슬롯 간에 공유하게 해, 같은 객체에 대해 여러 슬롯이 중복 예측하는 것을 억제한다 — 이것이 "정리" 표의 NMS 없이 중복을 억제하는 메커니즘이다(ablation Fig.4에서 NMS를 추가로 걸어도 층이 깊어질수록 이득이 사라짐 → 모델 스스로 중복을 이미 억제하고 있다는 증거).</mark>

```python
self.query_pos = nn.Parameter(torch.rand(100, hidden_dim))   # (100, 256), 학습되는 파라미터
h = decoder_layers(query=self.query_pos, memory=memory)      # (100, 256) -> (100, 256), 6층 반복
```

> [!warning] 이 구조 때문에 예상되는 문제점
> object query 개수 N(=100)이 학습 전에 고정된 상수라, 한 이미지에 실제 객체가 N개보다 많으면 구조적으로 전부 예측할 수 없다. 논문 자체의 실험(Fig.12)에서도 같은 클래스 객체가 50개를 넘어가면서부터 탐지 누락이 급격히 늘어나는 게 확인된다 — "실험 결과" 절에서 다시 다룸.

> [!info] 내 메모
> 

### ⑥ Prediction FFN
- **역할**: 디코더가 만든 N개의 출력 임베딩 각각을, 최종 (클래스, 박스) 예측으로 변환한다. 모든 슬롯이 같은 FFN 가중치를 공유한다.
- **구현**: 3-layer perceptron(ReLU, hidden dim=d) + 박스용 linear projection, 클래스용 별도 linear + softmax. 박스는 (중심 x, 중심 y, 너비, 높이) 4개 값을 sigmoid로 0~1 정규화해 출력, 클래스는 실제 클래스 + "no object(∅)" 포함 `num_classes+1`개 중 하나.
- **입출력 shape**: `(N=100, d)` → 클래스 `(N=100, num_classes+1)` + 박스 `(N=100, 4)`.

```python
self.linear_class = nn.Linear(hidden_dim, num_classes + 1)
self.linear_bbox = nn.Linear(hidden_dim, 4)

logits = self.linear_class(h)          # (100, 256) -> (100, num_classes+1)
boxes = self.linear_bbox(h).sigmoid()  # (100, 256) -> (100, 4), 0~1 정규화
```

> [!info] 내 메모
> 

### ⑦ Bipartite Matching + Hungarian Loss (학습 시에만)
- **역할**: 100개 예측과(∅로 패딩된) 100개 정답 슬롯을 1:1로 최적 매칭한 뒤, 매칭된 쌍에 대해서만 분류 손실 + 박스 손실(L1 + GIoU)을 계산한다. 상세 원리는 [[Bipartite_Matching_Hungarian_Algorithm]] 참고.
- <span style="color:#c0392b; font-weight:bold;">이 매칭 알고리즘 하나가 anchor 설계와 NMS 후처리를 동시에 대체하는, DETR 전체에서 가장 핵심적인 아이디어다.</span>

```python
# 의사코드 — 실제 구현은 scipy.optimize.linear_sum_assignment(헝가리안 알고리즘) 사용
cost_matrix = compute_matching_cost(preds, targets)   # (100, 100), 클래스+박스 비용
sigma = hungarian_algorithm(cost_matrix)               # 최적 1:1 매칭
loss = hungarian_loss(preds, targets, sigma)           # 매칭된 쌍만 loss 계산
```

> [!warning] 이 구조 때문에 예상되는 문제점
> 매칭이 "이번 배치에서 무엇이 최적인지"를 매 스텝 다시 계산하는 방식이라, 학습 초반에는 같은 객체가 스텝마다 다른 query에 매칭되어 학습 신호가 불안정하다. 이는 논문이 인정한 "매우 긴 학습 스케줄(300~500 epoch)이 필요하다"는 한계의 근본 원인 중 하나로 볼 수 있다.

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| ① CNN Backbone | (3, H₀, W₀) | (2048, H, W) | 이미지 → semantic feature map | ResNet-50/101 (32× 다운샘플) |
| ② 1×1 Conv | (2048, H, W) | (d=256, H, W) | 채널 압축 | 1×1 convolution |
| ③ Pos.Enc + Flatten | (256, H, W) | (HW, 256) | 위치정보 주입 + 시퀀스화 | sine/cosine 2D positional encoding |
| ④ Encoder × 6 | (HW, 256) | (HW, 256) | 전역 문맥 반영(인스턴스 분리) | Multi-head self-attention + FFN(1×1conv 2겹) |
| ⑤ Decoder × 6 | query(100,256) + memory(HW,256) | (100, 256) | 슬롯별 객체 후보 생성 + 중복 억제 | Self-attn + Cross-attn + FFN |
| ⑥ Prediction FFN | (100, 256) | 클래스(100,K+1) + 박스(100,4) | 최종 예측 변환 | 3-layer MLP(class), Linear(box) |
| ⑦ Matching+Loss | 예측 100 + 정답(패딩) 100 | 1:1 매칭 σ | 학습 신호 결정 | Hungarian algorithm, L1+GIoU+CE |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table 1 (COCO val, ResNet-50/101 backbone 비교)
**표를 보는 법**: 각 행이 하나의 모델이고, `AP`가 전체 성능, `AP_S/M/L`이 객체 크기별(작음/중간/큼) 성능이라 — DETR과 Faster R-CNN을 같은 파라미터 수(행)끼리 비교하면 된다.

| 벤치마크 | 지표 | Faster R-CNN-R101-FPN+ (baseline) | DETR-DC5-R101 |
|---|---|---|---|
| COCO val | AP | 44.0 | 44.9 |
| COCO val | AP_S (작은 객체) | 27.2 | 23.7 |
| COCO val | AP_L (큰 객체) | 56.0 | 62.3 |

> [!note]- 세부 결과 및 Ablation
> #### Table 2 — 인코더 층 수 효과 (0/3/6/12층)
> **보는 법**: 층이 늘수록 AP가 계속 오르는지 확인 → 0층(36.7)에서 12층(41.6)까지 단조 증가, 특히 AP_L 개선이 큼(54.2→61.9) — 인코더가 전역 문맥(주로 큰 객체 구분)에 기여함을 보여줌.
>
> #### Fig.3 — 인코더 self-attention 시각화
> **보는 법**: 특정 기준점(빨간 점)에서 어디를 attention하는지 히트맵으로 표시 — 같은 이미지 안 개별 소·양 등 인스턴스별로 attention이 분리되어 있는지 눈으로 확인하면 됨. 인코더가 이미 개체를 분리하고 있다는 근거.
>
> #### Fig.4 — 디코더 층별 AP + NMS 유무 비교
> **보는 법**: x축이 디코더 층 번호, y축이 AP/AP50 — NMS를 걸어도(점선) 층이 깊어질수록 NMS 없는 경우(실선)와 차이가 사라지는지 확인 → 층 1개일 땐 NMS가 도움(+8.2/9.5AP) 되지만 6층이 되면 거의 무의미해짐, 즉 모델이 스스로 중복 예측을 억제하도록 학습됨.
>
> #### Table 3 — Positional encoding 종류별 AP
> sine encoding을 encoder+decoder 양쪽에 다 넣은 baseline(마지막 행, AP 40.6)이 최고 — 공간 위치 인코딩을 제거하면 AP가 7.8까지 급락(32.8), positional encoding이 필수임을 보여줌.
>
> #### Table 4 — Loss 요소별 AP (class / L1 / GIoU)
> class+L1만(GIoU 없음) 35.8 → class+GIoU만(L1 없음) 39.9 → 셋 다 사용 40.6(baseline) — GIoU 단독으로도 대부분의 성능을 낸다는 것이 핵심 관찰.
>
> #### Fig.7 — Query slot별 예측 위치 분포
> **보는 법**: 각 서브플롯이 하나의 query slot, 점 하나가 "이 slot이 예측한 박스의 중심 위치"(색은 박스 크기·방향) — slot마다 다른 위치·크기에 특화되어 있는지 패턴을 눈으로 확인. 모든 slot이 "이미지 전체를 덮는 큰 박스" 모드도 공유함(가운데 정렬된 초록 점들).
>
> #### Fig.12 — Query 개수 한계 분석
> **보는 법**: x축이 한 이미지 안 실제 객체 수, y축이 놓친 비율(%) — 50개 근처부터 곡선이 급격히 올라가는지 확인. object query가 100개인데도 50개 근처부터 이미 누락이 급증하는 것이 "제안 방법" 절 ⑤의 구조적 한계와 직결된 실증.

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- 소형 객체 성능(AP_S)이 Faster R-CNN보다 낮음(27.2 vs 23.7) → <mark style="background: #FF5582A6;">논문 스스로 "FPN이 초기 detector에 가져온 것과 같은 개선이 DETR에도 필요하다"고 명시하며 향후 과제로 남김.</mark>
- Query slot이 학습 후 위치·크기에 암묵적으로 특화되어(Fig.7), 학습 데이터 분포(예: 한 이미지에 같은 클래스 객체가 최대 13개)를 벗어나는 극단적 상황(같은 클래스 100개)에서는 <mark style="background: #FF5582A6;">50개 근처부터 탐지 누락이 급증(Fig.12).</mark>

### 한계
- <mark style="background: #FF5582A6;">긴 학습 스케줄이 필요(500 epoch급) — Faster R-CNN 대비 수렴이 훨씬 느림.</mark>
- query 슬롯 개수(N=100)가 고정이라, 한 이미지에 100개보다 많은 객체가 있으면 원천적으로 다 예측할 수 없음(Fig.12에서 실증).
- Panoptic segmentation에서 mask AP가 stuff 클래스는 강하지만 things 클래스에서는 최대 8mAP까지 기존 방법에 뒤짐(Table 5).

### 생각할 점
- <mark style="background: #A6E3A1A6;">"매칭 알고리즘으로 후처리를 대체한다"는 아이디어는 이 위키의 [[Deformable-DETR]], [[DQ-DETR]] 등 DETR 계열 후속 논문 전체의 공통 토대다 — 이후 연구들은 대부분 "느린 수렴"과 "소형 객체 약점"이라는 이 논문이 스스로 인정한 두 한계를 각자 다른 방식으로 공략한다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">Query slot이 위치·크기에 특화된다는 관찰(Fig.7)은, "query를 무작위 학습 파라미터로 두지 말고 명시적으로 위치/크기 정보를 주입하면 수렴이 빨라지지 않을까"라는 후속 아이디어로 자연스럽게 이어진다 — 실제로 [[Deformable-DETR]], [[DQ-DETR]] 계열이 이 방향을 다룬다.</mark>

> [!info] 내 메모
> 

# 관련 개념
- [[Multi_Head_Self_Attention]] — 인코더의 전역 문맥 반영, 디코더의 슬롯 간 중복 억제에 사용된 핵심 연산.
- [[1x1_Convolution]] — backbone 출력 채널 압축, 트랜스포머 내부 FFN의 정체.
- [[Bipartite_Matching_Hungarian_Algorithm]] — anchor·NMS를 동시에 대체하는 예측-정답 매칭 알고리즘.

# 관련 문서
- (아직 없음 — 이 위키의 DETR 계열 비교 문서가 쌓이면 추가 예정)

# 읽어볼 만한 논문
- 참고문헌 기반: A. Vaswani et al., "Attention is all you need" (NeurIPS 2017) [47] — Transformer 원조 논문. DETR의 encoder-decoder 구조 자체가 이 논문의 표준 구조를 그대로 채택.
- 참고문헌 기반: S. Zheng et al., "Generalized Intersection over Union" [38] — DETR의 박스 손실에 쓰이는 GIoU의 원 논문. Table 4에서 GIoU 단독으로도 성능 대부분을 낸다는 관찰의 배경.
- 자유 추천(검증 필요): NMS를 학습 가능하게 만드는 다른 접근들(Learnable NMS 계열) — 검색 키워드: `learnable non-maximum suppression relation network object detection`. DETR이 "매칭으로 후처리를 완전히 없앤" 것과 달리 "후처리 자체를 학습시키는" 절충안들이라 비교하면 흥미로울 것으로 예상.
