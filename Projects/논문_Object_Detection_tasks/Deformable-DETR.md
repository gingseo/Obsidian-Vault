---
pm-task: true
projectId: "paperwiki-object-detection"
parentId:
id: "t-deformable-detr-8orhgsop6w"
title: "Deformable DETR: Deformable Transformers for End-to-End Object Detection"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-08-20"
due:
progress: 0
assignees: []
tags: []
customFields:
  "nh3oelhxmtcnb377": 2021
  "gx1mmrf0mtcnb37a": "ICLR"
subtaskIds: []
dependencies: []
year: 2021
venue: "ICLR"
jcr_quartile: Q1
task: [object-detection]
direction: [improvement, foundational]
paper_tags: [paper, object-detection, transformer, deformable-attention, multi-scale-feature, sparse-attention, end-to-end]
source: "Projects/논문_pdf/Object_Detection/2021_ICLR_Deformable-DETR.pdf"
source_type: personal
createdAt: "2026-08-24T03:03:00.000Z"
updatedAt: "2026-08-24T03:03:00.000Z"
---

Project: [[논문_Object_Detection|Object Detection]]
#paper #object-detection #transformer #deformable-attention #multi-scale-feature #sparse-attention #end-to-end

> [!quote] 원제
> **Deformable DETR: Deformable Transformers for End-to-End Object Detection**
> Xizhou Zhu, Weijie Su, Lewei Lu, Bin Li, Xiaogang Wang, Jifeng Dai — SenseTime Research / University of Science and Technology of China / The Chinese University of Hong Kong, ICLR 2021
> https://arxiv.org/abs/2010.04159

# 한 줄 요약
<mark style="background: #FFF3A3A6;">DETR의 attention이 모든 픽셀을 균일하게 바라봐 느린 수렴과 소형 객체 성능 열세를 낳는다는 문제를, 각 query가 reference point 주변의 소수 sampling point만 attend하는 (multi-scale) deformable attention module로 대체해, FPN 없이도 멀티스케일 feature를 직접 통합하면서 10배 적은 학습 epoch로 DETR을 능가하는 Deformable DETR.</mark>

> [!info] 내 메모
> 

# 정리

## 기존 방법의 한계

- **극도로 느린 수렴**:
  DETR은 COCO 기준 500 epoch가 필요해 Faster R-CNN 대비 10~20배 느리게 수렴한다. 초기화 시 cross-attention은 feature map 전체에 거의 균일한 attention을 주는데, 학습이 끝날 무렵에는 특정 위치에 집중된 sparse attention map으로 바뀌어야 하므로 이 극단적 변화를 학습하는 데 오래 걸린다.
- **소형 객체에서 낮은 성능**:
  최신 detector는 고해상도 feature map으로 소형 객체를 더 잘 탐지하는데, DETR encoder의 self-attention은 픽셀 수($N_q=N_k=HW$)에 대해 $O(H^2W^2C)$ 이차 복잡도를 가져 고해상도 feature map을 입력으로 쓰는 것 자체가 연산·메모리 측면에서 감당 불가능하다.

## 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — 사전 정의된 sparse attention pattern**
- Local window 기반(Liu et al. 2018a; Parmar et al. 2018; Child et al. 2019 외 다수), 특수 토큰으로 전역 접근 허용(Beltagy et al. 2020; Ainslie et al. 2020; Zaheer et al. 2020): attention 범위를 고정된 로컬 이웃으로 제한하거나 일부 sparse 패턴을 추가로 결합 — 연산량은 줄지만 전역 정보를 잃고, 패턴 자체는 고정(입력에 무관).
- 이미지 도메인에서 이 갈래의 방법들(Parmar et al. 2018; Child et al. 2019 등)은 이론적 복잡도는 줄어도, 표준 convolution과 동일 FLOPs 대비 실제로는 3배 이상 느리다고 저자들 스스로 인정 — 메모리 접근 패턴의 근본적 한계.
- **타겟/해결**: 느린 수렴(문제 ①) — attention을 sparse하게 만들어 연산·수렴 부담을 줄이려는 시도이지만, 이미지 특화 설계는 아니다.

**갈래 2 — 데이터 기반 학습된 sparse attention**
- LSH 기반 해싱(Kitaev et al. 2020), k-means 기반 클러스터링(Roy et al. 2020), block permutation(Tay et al. 2020a): query·key를 유사도로 그룹화해 sparse 연산 — 여전히 이미지 특화 설계는 아니며, 도입 사례가 드물다.
- **타겟/해결**: 느린 수렴(문제 ①) — 갈래 1과 마찬가지로 attention 연산 자체를 sparse화하려는 시도.

**갈래 3 — Low-rank 근사**
- Linear projection(Wang et al. 2020b), kernelization(Katharopoulos et al. 2020; Choromanski et al. 2020): attention의 저랭크 성질을 활용해 연산량 감소 — 근사 오차가 발생하고 이미지 feature map 처리에 특화되지 않는다.
- **타겟/해결**: 느린 수렴(문제 ①) — 근사로 연산량을 줄이지만 sparse 패턴 자체를 다루진 않는다.

**갈래 4 — Deformable convolution ([[Deformable_Convolutional_Networks]])**
- <mark style="background: #FFF3A3A6;">이미지 인식에서 sparse spatial location에 강력하고 효율적으로 attend하는 메커니즘이지만, DETR 성공의 핵심인 element 간 관계 모델링(relation modeling) 메커니즘이 없다.</mark>
- **타겟/해결**: 소형 객체 성능 열세(문제 ②) — sparse sampling으로 연산량은 효율적이지만, 이것만으로는 DETR의 relation modeling을 대체할 수 없다.

**갭**: <mark style="background: #FFF3A3A6;">이미지 특화 효율적 attention(갈래 1) 대다수는 이론적 복잡도만 줄일 뿐 실제 구현에서는 표준 convolution보다 느리고, deformable convolution(갈래 4)은 빠르고 sparse하지만 relation modeling이 없다. Deformable convolution의 sparse spatial sampling과 Transformer의 relation modeling을 동시에 갖춘 attention 메커니즘은 없었다.</mark>

## 이 논문이 풀고자 하는 문제
1. DETR의 극도로 느린 수렴(500 epoch) 문제를 해결하는 것
2. 고해상도 feature map을 이차 복잡도 없이 처리해 소형 객체 탐지 성능을 개선하는 것
3. 별도 FPN 없이 멀티스케일 feature를 attention 메커니즘 자체로 통합하는 것

**갭 종합**: <mark style="background: #FFF3A3A6;">Deformable convolution(갈래 4)의 sparse spatial sampling과 Transformer의 relation modeling을 동시에 갖춘 attention 메커니즘을 만들면, "느린 수렴"(갈래 1~3이 노리던 문제)과 "고해상도 처리 불가"(갈래 4가 노리던 문제)라는 서로 달라 보이는 두 문제를 하나의 메커니즘으로 동시에 해소할 수 있다는 것이 이 논문의 통찰이다.</mark>

> [!info] 내 메모
> 

# 해결 방법 요약

| | 문제 ① — DETR의 극도로 느린 수렴 | 문제 ② — 소형 객체 성능 열세 / 고해상도 feature 사용 불가 |
|---|---|---|
| **해결 방법** | 각 query가 reference point 주변 소수(K=4) sampling point만 보는 deformable attention module을 encoder/decoder 전체에 적용해, attention이 처음부터 sparse하게 시작하므로 "균일 → sparse"로 바뀌는 학습 부담 자체가 사라짐 | 같은 deformable attention을 feature level별로 확장(multi-scale deformable attention)해, query당 연산량이 feature map 크기와 무관한 상수에 가깝게 유지되므로 고해상도·멀티스케일 feature를 그대로 입력에 사용 가능 |
| **예상되는 문제점** | Sampling location이 무작위 접근(unordered memory access)을 유발해 표준 convolution보다는 여전히 느림 | Reference point 근방만 sampling하므로 reference point 자체가 부정확하면 관련 정보를 원천적으로 놓칠 위험 |

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">Deformable convolution처럼 각 query가 reference point 주변의 고정된 소수(K개) sampling point만 attend하도록 제한하는 <span style="color:#c0392b; font-weight:bold;">deformable attention module</span>을 제안한다 — attention을 "전체 feature map에 대한 pre-filtering 없는 전역 연산"에서 "reference point 주변 후보만 훑는 sparse 연산"으로 바꾸되, sampling offset과 함께 attention weight도 학습해 요소 간 관계 모델링 능력은 유지한다. 이를 <span style="color:#c0392b; font-weight:bold;">multi-scale deformable attention</span>으로 확장해 FPN 없이 encoder/decoder 자체에서 멀티스케일 feature를 통합한다.</mark>

## 전체 파이프라인 (Fig. 1, Fig. 2, Fig. 4 기준)

```
입력 이미지 (3, H₀, W₀)
       │
       ▼
Backbone (ResNet-50, ImageNet 사전학습)      → C3 (512, H₀/8, W₀/8)
                                                C4 (1024, H₀/16, W₀/16)
                                                C5 (2048, H₀/32, W₀/32)
       │
       ▼
1×1 Conv 투영 (C3~C5) + 3×3 stride-2 conv (C5→C6)   → x¹ (256, H₀/8, W₀/8)
                                                       x² (256, H₀/16, W₀/16)
                                                       x³ (256, H₀/32, W₀/32)
                                                       x⁴ (256, H₀/64, W₀/64)   [L=4 레벨, FPN 미사용]
       │
       ▼
Positional Encoding + Scale-level Embedding 추가   → 레벨별 (H_l·W_l, 256) + 레벨 구분 정보
       │
       ▼
① Multi-Scale Deformable Attention Encoder × 6층    → Σ(H_l·W_l), 256)   [encoder memory, self-attn 전량 교체]
       │
       ▼
② Multi-Scale Deformable Attention Decoder × 6층
   (object query 300개, self-attn은 표준 attention 유지, cross-attn만 교체)
       │  reference point p̂_q = sigmoid(Linear(query))   ← query별 2D 정규화 좌표
       ▼
Decoder 출력                                         → (N=300, 256)
       │
       ▼
Prediction FFN (3-layer MLP + Linear)                → 클래스(300, K+1) + 박스 offset(300, 4)
       │
       ▼ (선택적 확장)
③ Iterative Bounding Box Refinement (층마다 박스 갱신)  → 층별 reference point 재조정, 최종 박스 정제
④ Two-Stage (encoder-only proposal → decoder 초기 reference point로 사용)  → region proposal 기반 초기화
       │
       ▼
출력: 300개의 (클래스, 박스) 예측
```

> [!info] 내 메모
> 

### ① Deformable Attention Module
- **역할**:
  Transformer attention이 이미지 feature map을 처리할 때 겪는 근본 문제는 모든 spatial location을 다 봐야 한다는 점이다. Deformable attention module은 query feature로부터 예측한 reference point 주변의 고정된 소수 sampling point만 보게 해, feature map 크기와 무관하게 query당 연산량을 일정하게 유지한다. [[Deformable_Sampling_Offset]]에서 CNN에 도입된 오프셋 샘플링 개념을 Transformer attention의 sampling location으로 확장한 것이며, 표준 multi-head attention 자체는 [[Multi_Head_Self_Attention]] 참고.
- **구현**:
  Query feature `z_q`와 2D reference point `p_q`가 주어지면, `z_q`를 선형 투영해 `M`개 attention head 각각에 대해 `K`개의 sampling offset `Δp_mqk`와 이에 대응하는 attention weight `A_mqk`(softmax로 정규화, `Σ_k A_mqk=1`)를 동시에 예측한다. 각 head에서 `reference point + offset` 위치의 feature를 bilinear interpolation으로 읽어와 attention weight로 가중합한다. `M=8`, `K=4`가 기본값.
- **입출력 shape**:
  Query `z_q (256,)` + reference point `p_q (2,)` + feature map `x (256, H, W)` → 출력 `(256,)` (query 1개 기준, 실제로는 `N_q`개 query에 대해 배치 연산).

```python
# 논문 Eq.(2) 기반 의사코드
def deform_attn(zq, pq, x, M=8, K=4):
    offsets = linear_offset(zq)          # zq(256,) -> (M, K, 2), 2MK개 채널로 투영
    weights = softmax(linear_weight(zq)) # zq(256,) -> (M, K), MK개 채널, head별로 K개 합=1
    out = 0
    for m in range(M):
        head_out = 0
        for k in range(K):
            sample_loc = pq + offsets[m, k]                 # 2-d 실수 좌표 (분수 가능)
            v = bilinear_interpolate(Wv[m] @ x, sample_loc)  # value projection 후 샘플링
            head_out += weights[m, k] * v
        out += Wm[m] @ head_out
    return out
```

> [!example]- 구현 디테일
> 복잡도(Appendix A.1): `O(N_q C² + min(HWC², N_q KC²) + 5N_q KC + 3N_q CMK)`. `M=8, K≤4, C=256` 기본값에서 `5K+3MK < C`이므로 사실상 `O(N_q C² + min(HWC², N_q KC²))`로 근사된다. Encoder에서는 `N_q=HW`이므로 `O(HWC²)`로 spatial size에 선형(DETR encoder self-attention의 `O(H²W²C)`보다 낮은 차수), decoder에서는 `N_q=N=300`으로 spatial size와 무관해 `O(NKC²)`.
>
> Offset·weight 예측 선형 투영의 weight는 0, bias는 `M=8`개 head가 원형으로 서로 다른 방향을 향하도록 초기화(예: head별 `(±k,0),(0,±k),(±k,±k)` 방향, `K`개 지점을 균등 분산). Attention weight 초기값은 `A_mqk = 1/(LK)`로 균등.

<mark style="background: #FFF9D6A6;">Query별 key 후보를 소수로 제한하는 것은 deformable convolution의 sparse sampling 원리를 그대로 가져온 것이지만, offset과 함께 attention weight도 학습해 sampling point 간 상대적 중요도를 결정하므로("relation modeling"), deformable convolution에는 없던 요소 간 관계 모델링이 유지된다 — "정리" 표의 문제 ①(느린 수렴)을, attention이 처음부터 소수 위치에 sparse하게 집중하도록 만들어 "균일→sparse로 바뀌는 학습 부담" 자체를 없애는 방식으로 해결한다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> Sampling location이 query·이미지마다 달라 무작위 접근(unordered memory access)을 유발한다. 논문 스스로 "deformable attention이 전통적 convolution보다는 여전히 약간 느리다"고 명시하며, 이 속도 손실의 원인을 unordered memory access로 지목한다 — deformable convolution 계보의 근본적 트레이드오프가 그대로 이어짐을 인정.

> [!info] 내 메모
> 

### ② Multi-Scale Deformable Attention
- **역할**:
  Deformable attention module 하나만으로는 여전히 단일 스케일 feature만 다룬다. Multi-scale deformable attention은 이를 `L`개 feature level로 확장해, encoder·decoder가 레벨 간 정보 교환까지 attention 메커니즘 자체로 수행하게 만든다 — 별도의 top-down FPN 경로가 필요 없어진다.
- **구현**:
  ResNet의 C3~C5 stage feature를 1×1 conv로 채널 256 통일한 뒤, C5에 3×3 stride-2 conv를 추가로 적용한 C6까지 총 `L=4` 레벨을 encoder 입출력으로 동시 사용(Fig. 4: C3는 `H/8×W/8×512→H/8×W/8×256`, C4는 `H/16×W/16×1024→H/16×W/16×256`, C5는 `H/32×W/32×2048→H/32×W/32×256`, C6는 `H/64×W/64×256`). 각 query가 `L`개 레벨 각각에서 `K`개씩(총 `LK`개) sampling point를 attend한다. 레벨을 구분하기 위해 위치 임베딩에 더해 레벨별 scale-level embedding(랜덤 초기화, 학습됨)을 추가한다. Encoder는 self-attention 전체를, decoder cross-attention은 이 multi-scale deformable attention으로 교체한다(decoder self-attention은 object query 수가 적어(300개) 연산 부담이 없으므로 표준 [[Multi_Head_Self_Attention]] 유지).
- **입출력 shape**:
  4개 레벨 feature map `{x^l}, x^l∈(256, H_l, W_l)` + query별 정규화 reference point `p̂_q∈[0,1]²` → 출력 `(256,)` (query 1개 기준).

```python
# 논문 Eq.(3) 기반 의사코드. phi_l: 정규화 좌표를 l번째 레벨의 실제 feature map 좌표로 재조정
def ms_deform_attn(zq, p_hat_q, feature_levels, M=8, K=4, L=4):
    offsets = linear_offset(zq)          # -> (M, L, K, 2)
    weights = softmax(linear_weight(zq)) # -> (M, L, K), sum over (L,K) per head = 1
    out = 0
    for m in range(M):
        head_out = 0
        for l in range(L):
            for k in range(K):
                loc = phi_l(p_hat_q) + offsets[m, l, k]
                v = bilinear_interpolate(Wv[m] @ feature_levels[l], loc)
                head_out += weights[m, l, k] * v
        out += Wm[m] @ head_out
    return out
# K=1, L=1, Wv=I 로 축소하면 원조 deformable convolution과 수식적으로 동일
```

<mark style="background: #FFF9D6A6;">"정리" 표 문제 ②(고해상도·멀티스케일 처리 불가)를, attention의 sampling location을 레벨마다 별도로 두는 것만으로 해결한다 — query당 연산량이 `min(HWC², N_qKC²)`로 feature map 크기에 무관해지므로 고해상도 feature도 그대로 입력 가능하고, Table 2 ablation에서 FPN을 추가로 결합해도 성능이 개선되지 않아(43.8→43.8 AP) cross-level 정보 교환이 attention 메커니즘 자체로 이미 충분함을 뒷받침한다.</mark>

> [!info] 내 메모
> 

### ③④ Iterative Bounding Box Refinement & Two-Stage
- **역할**:
  Deformable attention은 "reference point 주변만 본다"는 설계이므로, reference point의 품질이 곧 attention 품질을 좌우한다. 두 변형 모두 reference point를 실제 객체 위치에 더 가깝게 정렬시켜 이 설계의 이점을 극대화한다.
  - Iterative bounding box refinement: optical flow의 반복적 정제 방식(Teed & Deng 2020, RAFT)에서 착안해, 각 decoder layer가 이전 layer의 예측 박스를 기준으로 상대 offset만 추가 예측.
  - Two-stage: encoder 단독(decoder 없이)으로 각 픽셀을 object query로 취급해 region proposal을 직접 생성(NMS 없이 top-K 선별)한 뒤, 이를 decoder의 초기 object query·reference point로 사용 — 기존 DETR/Deformable DETR의 "이미지와 무관하게 고정된 학습된 object query" 한계를 완화.
- **구현**:
  Iterative refinement는 `d`번째 decoder layer가 `(d-1)`번째 layer의 예측 박스 $\hat{b}_q^{d-1}$를 기준으로 상대 offset $\Delta b_q^d$만 예측하고 $\sigma(\Delta b + \sigma^{-1}(\hat{b}^{d-1}))$로 결합한다(레이어마다 독립된 예측 head, gradient는 offset에만 역전파). Two-stage는 픽셀별로 3-layer FFN(박스 회귀) + linear(foreground/background 이진 분류)를 적용해 proposal을 생성하고, DETR과 동일한 Hungarian loss로 학습한다.
- **입출력 shape**:
  Iterative refinement: 층별 예측 박스 `(N, 4)` → 다음 층 reference point `(N, 2)`로 재사용. Two-stage: 픽셀 단위 proposal `(ΣH_lW_l, 4)` → top-K 선별 후 decoder 초기 query/reference point `(N=300, 256)/(N=300, 2)`.

```python
# 논문 Appendix A.4 기반 의사코드
# Iterative bounding box refinement
b_hat = [p_hat_q]  # d=0: 초기 reference point를 박스 중심 초기값으로
for d in range(1, D + 1):
    delta_b = box_head[d](decoder_layer_output[d])          # (N, 4), 상대 offset만 예측
    b_hat_d = sigmoid(delta_b + inverse_sigmoid(b_hat[d-1])) # 이전 층 예측에 누적
    b_hat.append(b_hat_d)
    reference_point = b_hat_d[:, :2]                          # 다음 층 reference point로 사용

# Two-stage
proposals = pixelwise_box_head(encoder_memory)   # (sum(HlWl), 4), encoder-only
scores = pixelwise_cls_head(encoder_memory)      # (sum(HlWl), 1), foreground/background
topk_proposals = topk(proposals, scores, k=300)  # NMS 없이 top-K
decoder_query, reference_point = init_from(topk_proposals)  # decoder 2nd stage 입력
```

<mark style="background: #FFF9D6A6;">두 변형 모두 reference point를 예측 박스에 점점 더 가깝게 정렬시켜, decoder의 sampling location이 실제 객체 위치를 더 정확히 반영하도록 만든다 — deformable attention이 애초에 "reference point 주변만 본다"는 설계이므로, reference point 품질을 반복적으로 높이는 이 두 변형은 "정리" 표의 예상 문제점(reference point 부정확 시 정보 누락 위험)을 완화하며 deformable attention의 이점을 극대화하는 자연스러운 후속 개선이다.</mark>

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| Backbone + 1×1 Conv | (3, H₀, W₀) | 4레벨 (256, H_l, W_l), l=1..4 | 멀티스케일 feature 추출·채널 통일 | ResNet-50 + [[1x1_Convolution]] + C6용 3×3 stride-2 conv |
| ① Deformable Attention (encoder 기본 단위) | query z_q(256) + p_q(2) + x(256,H,W) | (256,) | Sparse spatial sampling, 연산량을 feature map 크기와 무관하게 | 선형 투영(offset+weight) + bilinear interpolation |
| ② Multi-Scale Deformable Attention Encoder ×6 | 4레벨 (256, H_l, W_l) | 동일 shape, 값만 갱신 (encoder memory) | 레벨 간 정보 교환(FPN 대체) | ①을 L=4 레벨로 확장, self-attention 전체 교체 |
| ② Multi-Scale Deformable Attention Decoder ×6 | query(300,256) + memory | (300, 256) | Object query가 reference point 주변 feature로 갱신 | Cross-attn만 교체, self-attn은 [[Multi_Head_Self_Attention]] 유지 |
| Prediction FFN | (300, 256) | 클래스(300,K+1) + 박스(300,4) | 최종 예측 변환 | 3-layer MLP(box) + Linear(class), [[Bipartite_Matching_Hungarian_Algorithm]]로 학습 |
| ③ Iterative Refinement (선택) | 층별 박스(N,4) | 정제된 박스(N,4) | Reference point를 예측에 맞춰 반복 정렬 | 층별 독립 head, 상대 offset 예측 |
| ④ Two-Stage (선택) | encoder memory | proposal(N,4) → decoder 초기값 | 학습된 고정 query 대신 이미지 기반 초기 proposal 제공 | Encoder-only 픽셀별 박스·분류 head, top-K 선별 |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table 1 (COCO 2017 val, DETR-DC5 vs Deformable DETR)
**표를 보는 법**: 각 행이 하나의 방법이고, Epochs 열의 차이(500 vs 50)에 주목해서 AP·AP_S를 비교하면 수렴 속도 개선을 확인할 수 있다.

| 벤치마크 | 지표 | DETR (500 epoch) | Deformable DETR (50 epoch) |
|---|---|---|---|
| COCO val | AP | 42.0 | 43.8 |
| COCO val | AP_S (작은 객체) | 20.5 | 26.4 |
| COCO val | 학습 GPU시간 | 2000 | 325 |

> [!note]- 세부 결과 및 Ablation
> #### Table 1 — DETR과 전면 비교 (COCO 2017 val)
> **보는 법**: params/FLOPs가 비슷한 모델끼리(Deformable DETR ≈ DETR-DC5 ≈ Faster R-CNN+FPN) Epochs·AP·GPU시간을 비교하면 수렴 속도 이득이 뚜렷하다.
>
> | 방법 | Epochs | AP | AP50 | AP75 | AP_S | AP_M | AP_L | params | FLOPs | GPU시간 | FPS |
> |---|---|---|---|---|---|---|---|---|---|---|---|
> | Faster R-CNN+FPN | 109 | 42.0 | 62.1 | 45.5 | 26.6 | 45.4 | 53.4 | 42M | 180G | 380 | 26 |
> | DETR | 500 | 42.0 | 62.4 | 44.2 | 20.5 | 45.8 | 61.1 | 41M | 86G | 2000 | 28 |
> | DETR-DC5 | 500 | 43.3 | 63.1 | 45.9 | 22.5 | 47.3 | 61.1 | 41M | 187G | 7000 | 12 |
> | DETR-DC5 | 50 | 35.3 | 55.7 | 36.8 | 15.2 | 37.5 | 53.6 | 41M | 187G | 700 | 12 |
> | DETR-DC5⁺(focal loss+300 query) | 50 | 36.2 | 57.0 | 37.4 | 16.3 | 39.2 | 53.9 | 41M | 187G | 700 | 12 |
> | Deformable DETR | 50 | 43.8 | 62.6 | 47.7 | 26.4 | 47.1 | 58.0 | 40M | 173G | 325 | 19 |
> | + iterative bbox refinement | 50 | 45.4 | 64.7 | 49.0 | 26.8 | 48.3 | 61.7 | 40M | 173G | 325 | 19 |
> | ++ two-stage Deformable DETR | 50 | 46.2 | 65.2 | 50.0 | 28.8 | 49.2 | 61.7 | 40M | 173G | 340 | 19 |
> - 동일 50 epoch에서 DETR-DC5는 35.3 AP에 그치지만 Deformable DETR은 43.8 AP — 수렴 속도 차이가 극명. AP_S는 DETR-500epoch(20.5) 대비 Deformable DETR-50epoch(26.4)이 이미 앞섬.
> - 추론 속도는 Faster R-CNN+FPN보다 25% 느리지만 DETR-DC5보다 1.6배 빠름 — DETR-DC5의 느린 속도가 Transformer attention의 큰 메모리 접근 때문이라고 분석.
>
> #### Fig. 3 — 수렴 곡선 (COCO val)
> **보는 법**: x축 epoch, y축 AP. Deformable DETR(빨강)이 DETR-DC5(회색)보다 훨씬 이른 epoch에서 더 높은 AP에 도달하는지 확인. 50 epoch 근방에서 Deformable DETR은 이미 43.8까지 도달, DETR-DC5는 500 epoch에서야 43.6.
>
> #### Table 2 — Deformable Attention Ablation (COCO 2017 val)
> **보는 법**: MS inputs(멀티스케일 입력 사용 여부)·MS attention(멀티스케일 attention 사용 여부)·K(sampling point 수)·FPN 열을 하나씩 켜가며 AP 변화를 본다.
>
> | MS inputs | MS attention | K | FPN | AP | AP_S |
> |---|---|---|---|---|---|
> | | | 1 | w/o | 39.7 | 21.2 |
> | ✓ | | 1 | w/o | 41.4 | 24.1 |
> | ✓ | | 4 | w/o | 42.3 | 24.8 |
> | ✓ | ✓ | 4 | w/o | 43.8 | 26.4 |
> | ✓ | ✓ | 4 | FPN(Lin et al. 2017a) 추가 | 43.8 | 26.5 |
> | ✓ | ✓ | 4 | BiFPN(Tan et al. 2020) 추가 | 43.9 | 25.6 |
> - Multi-scale input 자체가 +1.7 AP(특히 AP_S +2.9), sampling point 수 증가(1→4)가 +0.9 AP, multi-scale attention(레벨 간 정보 교환)이 추가로 +1.5 AP.
> - FPN·BiFPN을 추가해도 유의미한 성능 향상 없음 — multi-scale deformable attention 자체가 이미 레벨 간 정보를 충분히 교환한다는 근거.
>
> #### Table 3 — SOTA 비교 (COCO 2017 test-dev)
> **보는 법**: TTA(test-time augmentation) 열이 체크된 행은 horizontal flip + multi-scale testing 적용 결과이므로 TTA 없는 행끼리, 있는 행끼리 비교해야 공정하다.
>
> | 방법 | Backbone | TTA | AP |
> |---|---|---|---|
> | FCOS | ResNeXt-101 | | 44.7 |
> | ATSS | ResNeXt-101+DCN | ✓ | 50.7 |
> | Deformable DETR | ResNet-50 | | 46.9 |
> | Deformable DETR | ResNet-101 | | 48.7 |
> | Deformable DETR | ResNeXt-101 | | 49.0 |
> | Deformable DETR | ResNeXt-101+DCN | | 50.1 |
> | Deformable DETR | ResNeXt-101+DCN | ✓ | **52.3** |
> - ResNeXt-101+DCN(즉 백본에도 deformable convolution 결합) 조합이 없이도(ResNeXt-101 단독) FCOS·ATSS급 SOTA에 근접, DCN을 백본에 추가하면 최고 성능.
>
> #### Fig. 5, 6 — 정성 분석 (Appendix A.5~A.6)
> **보는 법**: Fig. 5는 예측 항목별(x,y,w,h,category) gradient norm을 원본 이미지에 겹쳐 표시 — 밝은 red일수록 그 픽셀이 예측에 크게 기여. Fig. 6은 encoder self-attention·decoder cross-attention의 실제 sampling point(색=attention weight, 초록 십자=reference point)를 이미지 위에 표시.
> - Gradient norm(Fig. 5): DETR과 유사하게 박스 좌표(x,y,w,h)는 객체의 극단점(extreme point, 경계)에 주로 의존. 다만 카테고리 예측(c)은 DETR과 달리 객체 내부 픽셀에도 의존.
> - Sampling point(Fig. 6): encoder self-attention은 이미 개별 인스턴스를 분리(DETR과 유사), decoder cross-attention은 객체의 전경(foreground) 영역 전체에 걸쳐 sampling point가 분포 — extreme point뿐 아니라 내부 point도 카테고리 판별에 필요하다는 gradient 분석과 일치.

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- Sampling location이 무작위 접근(unordered memory access)을 유발 → <mark style="background: #FF5582A6;">논문 스스로 "deformable attention이 전통적 convolution보다는 여전히 약간 느리다"고 명시하며, 이 속도 손실의 원인을 unordered memory access로 지목한다 — deformable convolution 계보의 근본적 트레이드오프가 그대로 이어짐을 인정.</mark>
- Reference point 근방만 sampling하므로 reference point 자체가 잘못되면 관련 정보를 원천적으로 놓칠 위험 → <mark style="background: #FF5582A6;">이 위험을 완화하기 위해 iterative bounding box refinement·two-stage 변형을 별도로 제안했다는 것은, 기본 설계만으로는 이 위험이 충분히 낮지 않았음을 방증한다.</mark>

### 한계
- <mark style="background: #FF5582A6;">Two-stage 방식에서 첫 stage는 decoder 없이 encoder-only로 region proposal을 생성하는데, self-attention의 이차 복잡도 때문에 "각 픽셀을 object query로 직접 사용"할 수 없다고 저자가 명시 — 근본적 해결이 아니라 decoder를 제거해 우회한 것.</mark>
- <mark style="background: #FF5582A6;">M(head)=8, K(sampling point)=4 같은 핵심 하이퍼파라미터의 최적값 탐색 범위가 제한적(Table 2도 K=1/4 두 값만 비교) — 더 넓은 탐색이나 데이터셋별 민감도 분석은 없음.</mark>
- ResNeXt-101+DCN(즉 attention만이 아니라 백본에도 deformable convolution을 결합) 조합이 최고 성능(52.3 AP)을 내는데, 이는 attention만으로는 부족해 여전히 별도 deformable convolution 모듈이 백본 단에 필요함을 시사 — 두 메커니즘이 상호 대체가 아니라 보완 관계임을 논문이 암묵적으로 인정.

### 생각할 점
- <mark style="background: #A6E3A1A6;">이 논문은 [[Deformable_Convolutional_Networks]]의 offset 메커니즘을 "CNN의 고정 grid"에서 "attention의 전역 key 집합"으로 옮긴 사례로, [[Deformable_Sampling_Offset]] 개념이 처음 다른 아키텍처(Transformer)로 이식된 것이다 — Appendix A.1/본문 4.1절에서 `K=1,L=1,W'_m=I`일 때 정확히 deformable convolution으로 퇴화함을 수식으로 증명해, 이 계승 관계가 저자들 스스로도 명시적으로 인식하고 있던 설계임을 보여준다.</mark>
- <mark style="background: #A6E3A1A6;">Table 2에서 "multi-scale input 자체"(+1.7 AP)가 "sampling point 수 4배 증가"(+0.9 AP)보다 기여가 크다는 점은, 이 위키의 [[ORFENet]]에서 관찰된 "다중 소스를 쓴다는 것 자체의 기여가 정교화보다 크다"는 패턴과 다시 한번 일치한다 — 서로 다른 아키텍처(FCOS 기반 vs DETR 기반)에서 반복 관찰되는 현상이라는 점에서 근거가 강화된다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">이 논문이 처음 지적한 AP_S 개선(20.5→26.4)은 여전히 절대값 자체는 낮은 편이다 — 이후 처리할 DQ-DETR·Density-Aware DETR 등 "DETR + dynamic query" 계열이 정확히 이 지점(소형/타이니 객체를 위한 query 자체의 품질)을 더 파고드는 것으로 보이며, deformable attention의 "reference point 주변만 본다"는 sparse sampling과 이후 계열의 "query 자체를 동적으로 생성/우선순위화한다"는 접근이 어떻게 다른 층위에서 상호보완적인지 처리하며 비교할 필요가 있다.</mark>
- Two-stage의 "각 픽셀=object query"라는 초기 proposal 생성 방식은, 이 위키의 [[QueryDet]]이 쓰는 "저해상도 예측으로 고해상도 위치를 좁힌다"는 coarse-to-fine query 방식과 목적(밀집 픽셀에서 후보를 어떻게 좁히는가)이 유사해 보인다 — 다만 QueryDet은 CNN 계열, 이 논문은 attention 계열이라는 아키텍처 층위의 차이가 있다.

> [!info] 내 메모
> 

# 관련 개념
- [[Deformable_Sampling_Offset]] — [[Deformable_Convolutional_Networks]]가 CNN에 도입한 개념을 Transformer attention의 sampling location으로 확장한 첫 사례. "등장 논문"에 이번 논문 이미 반영됨.
- [[Multi_Head_Self_Attention]] — Encoder의 원래 self-attention과 decoder의 self-attention(query끼리 상호 참조)에 표준 형태 그대로 사용. Cross-attention만 deformable attention으로 교체됨.
- [[Bipartite_Matching_Hungarian_Algorithm]] — DETR과 동일하게 예측-정답 1:1 매칭과 Hungarian loss로 학습(anchor·NMS 불필요라는 DETR의 이점을 그대로 계승).

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]] — DETR·Deformable Convolutional Networks와 함께 foundational 계열로 별도 취급(비교표 대상 아님). 이후 dynamic query DETR 계열(DQ-DETR 등) 다수의 직접 baseline.

# 읽어볼 만한 논문
- 참고문헌 기반: N. Carion, F. Massa, G. Synnaeve, N. Usunier, A. Kirillov, S. Zagoruyko, "End-to-end object detection with transformers" (DETR, ECCV 2020) — 이미 이 위키의 [[DETR]] 노트로 존재. 이 논문이 직접 개선하는 원조.
- 참고문헌 기반: Z. Teed, J. Deng, "RAFT: Recurrent all-pairs field transforms for optical flow" (ECCV 2020) — iterative bounding box refinement가 아이디어를 차용한 optical flow의 반복적 정제 기법. 반복 정제 설계의 배경 이해에 도움.
- 참고문헌 기반: X. Zhu, H. Hu, S. Lin, J. Dai, "Deformable ConvNets v2: More deformable, better results" (CVPR 2019) — Table 3에서 최고 성능(52.3 AP)에 쓰인 DCN의 개선판. Deformable convolution 계보의 최신 버전.
- 참고문헌 기반: M. Tan, R. Pang, Q. V. Le, "EfficientDet: Scalable and efficient object detection" (CVPR 2020) — Table 2 ablation에서 비교 대상으로 쓰인 BiFPN의 원조 논문. Multi-scale deformable attention이 FPN류 구조 없이도 동등 이상의 성능을 낸다는 주장의 비교 기준.
- 자유 추천(검증 필요): Deformable attention을 vision task 전반(segmentation, tracking 등)으로 확장한 후속 연구 — 검색 키워드: `deformable attention module extension segmentation tracking 2022 2023`. Deformable DETR의 attention 메커니즘이 detection 외 다른 dense prediction task에 어떻게 재사용됐는지 파악하는 데 유용.
