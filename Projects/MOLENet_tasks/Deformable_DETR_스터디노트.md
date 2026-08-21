# Deformable DETR 완전정복 — 초보자용 상세 스터디 노트

> 대상 논문
> 1. **Deformable DETR: Deformable Transformers for End-to-End Object Detection** (Zhu et al., ICLR 2021)
> 2. (배경지식용) **Deformable Convolutional Networks** (Dai et al., ICCV 2017) — 이하 **DCN**

---

## 0. 전체 지도 먼저 보기 (Big Picture)

이 논문을 한 문장으로 요약하면:

> **"DETR은 좋은 아이디어(end-to-end 검출)였지만 느리고 작은 물체를 못 잡는다. 원인은 Transformer의 attention이 이미지의 '모든 픽셀'을 다 보기 때문이다. 그래서 DCN(deformable convolution)처럼 '몇 개의 중요한 점만 골라서' 보는 attention을 만들었더니, 10배 빨리 수렴하면서 작은 물체 성능까지 좋아졌다."**

이걸 이해하기 위한 최단 경로는 아래 3단계입니다.

```
[1단계] Transformer attention이 뭐고 왜 느린가?
        ↓
[2단계] Deformable Convolution은 무엇이고, "모든 곳을 보지 않고
        몇 개의 점만 본다"는 게 무슨 뜻인가?
        ↓
[3단계] 이 둘을 합친 "Deformable Attention"은 정확히 어떻게 계산되는가?
        (Deformable DETR의 핵심)
```

이 노트도 이 순서 그대로 구성했습니다. 즉,

1. 배경지식 (CNN, Attention, Object Detection 용어)
2. DETR 복습 (Deformable DETR이 풀려는 "문제"가 뭔지)
3. Deformable Convolution 완전 이해 (DCN 논문)
4. Deformable DETR의 핵심 — Deformable Attention Module
5. 전체 아키텍처 (인코더/디코더)
6. 추가 기법 (iterative refinement, two-stage)
7. 실험 결과 해석
8. 당신의 연구(Small Object Detection)에 어떻게 연결할지
9. 검수 노트 — 원문 대조

---

## 1. 배경지식 준비

이미 알고 계실 개념도 있겠지만, "Deformable Attention 수식"을 볼 때 헷갈리지 않도록 표기법 위주로 빠르게 정리합니다.

### 1.1 CNN과 "고정된 시야"의 문제

일반적인 3×3 convolution은 이렇게 동작합니다.

- 출력의 한 위치 `p0`를 계산하려면, 입력에서 `p0`를 중심으로 **정해진 9개 위치**(3×3 격자, `R = {(-1,-1), (-1,0), ..., (1,1)}`)의 값을 가져와 가중합을 합니다.
- 이 9개 위치는 **이미지 내용과 무관하게 항상 고정**되어 있습니다. 고양이를 보든 자동차를 보든 무조건 "정사각형 격자"만 봅니다.

> **비유**: 자를 대고 정확히 3cm 간격으로 뚫린 9개 구멍으로만 사진을 훔쳐보는 것과 같습니다. 피사체가 그 구멍 모양과 안 맞으면(예: 길쭉한 사람, 작은 물체) 비효율적으로 배경만 많이 보게 됩니다.

이 "고정된 격자"가 나중에 DCN이 해결하려는 문제이고, Transformer의 attention은 정반대로 "전부 다 보는" 문제를 가지고 있습니다 (아래 1.2).

### 1.2 Attention / Transformer 핵심 개념

Transformer의 attention은 Query–Key–Value(Q,K,V) 구조로 동작합니다.

- **Query(질의)**: "나는 무엇을 찾고 싶은가"를 나타내는 벡터. 예) 디코더에서는 "이 object query가 찾는 물체"
- **Key(열쇠)**: "나는 이런 정보를 가지고 있다"를 나타내는 벡터. 이미지의 각 픽셀 하나하나가 key가 됩니다.
- **Value(값)**: 실제로 가져올 정보(내용물).

동작 방식: Query와 모든 Key를 비교(내적)해서 **유사도 점수(attention weight)**를 계산하고, 그 점수로 Value들을 가중합해서 최종 출력을 만듭니다.

논문의 수식 (Eq. 1)로 보면:

$$\text{MultiHeadAttn}(z_q, x) = \sum_{m=1}^{M} W_m \left( \sum_{k \in \Omega_k} A_{mqk} \cdot W'_m x_k \right)$$

겁먹지 마시고 하나씩 뜯어봅시다.

| 기호 | 의미 |
|---|---|
| $z_q$ | query 요소(찾는 주체)의 특징 벡터 |
| $x_k$ | key 요소(참고할 대상, 예: 이미지 픽셀)의 특징 벡터 |
| $\Omega_k$ | key 요소들의 전체 집합 (예: 이미지의 모든 픽셀) |
| $m$ | attention head 번호 (여러 개의 "관점"으로 동시에 보는 것) |
| $A_{mqk}$ | query $q$가 key $k$를 얼마나 중요하게 보는지의 가중치 (softmax로 합이 1) |
| $W_m, W'_m$ | 학습되는 선형 변환 행렬 |

핵심은 **$\sum_{k \in \Omega_k}$**, 즉 **모든 key(=모든 픽셀)에 대해 합을 구한다**는 점입니다. 이게 바로 "전부 다 본다"는 뜻이고, 이게 문제의 근원입니다.

**왜 문제인가? — 두 가지**

1. **계산량**: query 개수를 $N_q$, key 개수를 $N_k$라 하면 계산 복잡도는 $O(N_q N_k C)$입니다. 이미지에서는 $N_q = N_k = H \times W$(가로×세로 픽셀 수)이므로, 이미지 해상도가 커질수록 계산량이 **제곱으로 폭발**합니다. (feature map을 2배 키우면 계산량은 4배가 아니라, $H \times W$가 4배 되므로 계산량은 16배가 됩니다)
2. **학습 초기의 "멍한 시선"**: 학습 초기에는 모든 파라미터가 랜덤이라, attention weight $A_{mqk}$가 어느 한 곳으로 쏠리지 못하고 거의 $1/N_k$로 **균등하게 흩어져** 있습니다(모든 곳을 조금씩 흐릿하게 보는 상태). 학습이 진행되며 서서히 "정말 중요한 픽셀"에 집중하도록 배워야 하는데, $N_k$(픽셀 수)가 크면 클수록 이 "집중하는 법"을 배우는 데 오래 걸립니다. → **느린 수렴**의 원인.

> **비유**: 넓은 강당에서 눈을 감고 100명이 동시에 말하는 걸 듣다가, 서서히 "저 사람 목소리만 집중해서 들어야겠다"를 배우는 상황. 사람이 100명이면 오래 걸리고, 10명이면 금방 배웁니다. Deformable attention은 "처음부터 몇 명(K명)만 골라서 듣는" 방법입니다.

### 1.3 Object Detection 기본 용어

- **Bounding Box (bbox)**: 물체를 감싸는 사각형. 보통 (중심 x, 중심 y, 너비, 높이)로 표현.
- **AP (Average Precision)**: 검출 정확도 지표. 여러 IoU 임계값에서 정밀도-재현율 곡선의 면적.
- **$AP_S, AP_M, AP_L$**: 각각 **작은/중간/큰 물체**에 대한 AP. COCO 기준 작은 물체는 면적 $32^2$px 이하. → 우리가 관심있는 지표는 주로 $AP_S$.
- **NMS (Non-Maximum Suppression)**: 겹치는 중복 bbox를 후처리로 제거하는 손으로 짠(hand-crafted) 알고리즘. DETR은 이걸 없앤 게 특징 중 하나입니다.
- **Hungarian Matching (이분 매칭)**: DETR이 NMS 없이 "예측 N개 vs 정답 M개"를 1:1로 최적 매칭시켜 loss를 계산하는 방법. 이 논문에서는 자세히 안 다루지만, "예측 개수를 고정(N=300)해두고 정답과 최적으로 짝지어 학습한다" 정도로 이해하면 충분합니다.
- **backbone**: 이미지에서 특징(feature map)을 뽑아내는 CNN (보통 ResNet).
- **multi-scale feature map / FPN**: 작은 물체는 고해상도 feature map에서, 큰 물체는 저해상도 feature map에서 더 잘 잡히므로, 여러 해상도의 feature map을 같이 사용하는 기법. (Deformable DETR에서 매우 중요하게 다뤄짐 — 작은 물체 성능과 직결)

---

## 2. DETR 복습 — Deformable DETR이 풀려는 "문제"

Deformable DETR은 독립적인 새 논문이 아니라, **DETR(Carion et al., 2020)의 리모델링**입니다. 그래서 DETR을 먼저 알아야 합니다.

### 2.1 DETR이 한 일

기존 detector(Faster R-CNN 등)는 anchor 생성, NMS 같은 손으로 설계한 부품이 많았습니다. DETR은:

- CNN backbone → Transformer **encoder** (이미지 특징을 정제) → Transformer **decoder** (N개의 "object query"가 이미지에서 물체를 찾아냄) → FFN으로 bbox와 class 예측
- Hungarian matching으로 NMS 없이 학습

즉, "이미지 픽셀들"이 encoder에서는 서로 attention(self-attention)을 주고받고, decoder에서는 "물체를 찾는 질문들(object query)"이 이미지 픽셀들에 cross-attention으로 질문을 던지는 구조입니다.

### 2.2 DETR의 두 가지 고질병

논문 Table 1에 나온 실측 수치로 보면:

| Method | Epochs | AP | $AP_S$ | 학습 GPU시간 |
|---|---|---|---|---|
| Faster R-CNN+FPN | 109 | 42.0 | 26.6 | 380 |
| DETR | **500** | 42.0 | **20.5** | **2000** |
| DETR-DC5 | 500 | 43.3 | 22.5 | 7000 |

- **① 느린 수렴**: Faster R-CNN은 109 epoch면 충분한데 DETR은 **500 epoch**(약 5~10배)가 필요합니다. GPU 시간도 2000시간으로 압도적으로 오래 걸립니다.
- **② 작은 물체에 약함**: $AP_S$가 20.5로, Faster R-CNN(26.6)보다 명백히 낮습니다. 큰 물체 $AP_L$(61.1)은 오히려 더 좋은데 말이죠.

### 2.3 왜 이런 문제가 생기나 (원인 분석)

원인은 1.2에서 다룬 것과 정확히 같습니다.

- **인코더의 self-attention**: 픽셀 대 픽셀 attention이므로 복잡도가 $O(H^2 W^2 C)$ — **이미지 크기의 제곱**. 그래서 DETR은 어쩔 수 없이 **저해상도 feature map 하나만** 사용합니다(보통 stride 32, 즉 원본의 1/32 크기). 이게 작은 물체 성능이 낮은 **직접적인 이유**입니다 — 애초에 작은 물체가 뭉개져서 안 보이는 해상도만 쓰는 것이니까요.
- **디코더의 cross-attention**: 학습 초기엔 object query가 이미지 전체에 거의 균등하게 attention을 뿌립니다. 학습이 끝날 때쯤에야 "물체의 경계 부분(극점, extremities)"에만 뾰족하게 집중하도록 배웁니다. 이 극적인 변화를 배우는 데 긴 학습이 필요합니다.

> 정리: **"모든 곳을 다 보는" Transformer attention의 특성 → 고해상도를 못 씀(작은 물체 못 봄) + 수렴이 느림(집중하는 법을 배우는 데 오래 걸림)**. 이 두 문제를 **동시에** 풀어야 하는데, 그 힌트를 저자들은 **Deformable Convolution**에서 찾았습니다.

---

## 3. Deformable Convolution 완전 이해 (DCN, 2017)

여기서부터는 두 번째 첨부 논문(DCN) 내용입니다. Deformable DETR을 만든 팀(Jifeng Dai가 두 논문 모두 공저자)이 원래 만들었던 기법이라, 사실상 "형-동생" 관계의 논문입니다.

### 3.1 표준 Convolution 복습 (수식으로)

1.1절에서 말한 "고정된 격자"를 수식으로 쓰면 (DCN 논문 Eq. 1):

$$y(p_0) = \sum_{p_n \in \mathcal{R}} w(p_n) \cdot x(p_0 + p_n)$$

- $y(p_0)$: 출력 feature map의 위치 $p_0$에서의 값
- $\mathcal{R}$: 샘플링할 상대 위치들의 집합. 3×3 kernel이면 $\mathcal{R} = \{(-1,-1), (-1,0), ..., (1,1)\}$ (9개, **항상 고정**)
- $w(p_n)$: 그 위치에 곱해지는 학습된 가중치(커널 값)

→ 입력에서 $p_0 + p_n$ 위치, 즉 항상 **정사각형 격자 모양**으로만 값을 가져옵니다.

### 3.2 Deformable Convolution — "격자에 오프셋을 더하자"

DCN의 아이디어는 아주 단순하고 강력합니다: 저 고정된 $p_n$에 **학습되는 오프셋(offset) $\Delta p_n$을 더해서** 격자 모양이 이미지 내용에 맞게 자유롭게 휘어지도록 만드는 것입니다 (Eq. 2):

$$y(p_0) = \sum_{p_n \in \mathcal{R}} w(p_n) \cdot x(p_0 + p_n + \Delta p_n)$$

달라진 건 딱 하나, $x(p_0+p_n)$이 $x(p_0+p_n+\Delta p_n)$이 된 것뿐입니다.

- $\Delta p_n$은 **정수가 아닌 실수**(fractional)일 수 있습니다. 예를 들어 (1.3, -0.7) 같은 위치. 그런데 이미지(feature map)는 정수 좌표에만 값이 있으므로, 소수점 위치의 값은 주변 4개 정수 픽셀 값을 거리 비례로 섞어서 계산합니다. 이걸 **양선형 보간(bilinear interpolation)**이라 합니다 (Eq. 3, 4).
- 이 $\Delta p_n$은 **어떻게 얻는가?** → 입력 feature map에 작은 conv layer를 하나 더 붙여서(Figure 2), 위치별로 오프셋 값을 예측하게 학습시킵니다. 즉, **오프셋도 데이터로부터 학습되는 파라미터**입니다. 사람이 정하는 게 아닙니다.

> **비유**: 원래는 도장(3×3 격자)을 이미지 위에 무조건 반듯하게 찍었다면, deformable conv는 그 도장의 9개 다리를 **말랑말랑한 관절**로 만들어서, 물체 모양에 맞춰 다리를 구부려 찍는 것과 같습니다. (논문 Figure 1이 이걸 그림으로 보여줍니다: (a) 반듯한 격자, (b) 물체 모양을 따라 휘어진 격자, (c)(d)는 특수 케이스로 확대/회전처럼 보이는 패턴)

**중요한 성질**: deformable conv는 표준 convolution, 확대(dilation), 회전 등 다양한 기하학적 변형을 **일반화한 상위 개념**입니다 (Figure 1(c)(d)가 이걸 시각적으로 증명). 즉 "표준 conv"는 deformable conv에서 오프셋이 항상 0인 특수한 경우일 뿐입니다.

### 3.3 Deformable RoI Pooling (참고용, Deformable DETR과는 간접적 관련)

Faster R-CNN류 detector는 RoI(관심영역)를 고정된 k×k 칸(bin)으로 나눠서 pooling합니다. Deformable RoI Pooling은 이 칸의 위치에도 오프셋을 더해서, RoI 내부에서 물체가 있는 부분으로 칸이 이동하도록 합니다 (Figure 3, 7). — Deformable DETR은 RoI pooling을 쓰지 않으므로 참고만 하고 넘어가도 됩니다.

### 3.4 시각화 결과가 말해주는 것 (Figure 5, 6, Table 2)

DCN 논문에서 가장 인상적인 실험은 "학습된 오프셋이 실제로 무엇을 하는가"를 보여주는 부분입니다.

- **Figure 5, 6**: 배경/작은 물체/큰 물체에 대해 학습된 필터의 샘플링 위치를 시각화했더니, **큰 물체를 볼 때는 샘플링 점들이 넓게 퍼지고, 작은 물체를 볼 때는 좁게 모이며, 배경을 볼 때는 또 다른 패턴**을 보입니다. 즉 필터의 "유효 시야(receptive field)"가 **물체 크기에 맞춰 자동으로 조절**됩니다.
- **Table 2 (effective dilation)**: 이를 수치로 확인 — 작은 물체(small)에서 평균 유효 dilation이 가장 작고, 큰 물체(large)에서 가장 큽니다. → "필터가 물체 크기에 적응적으로 반응한다"는 것을 정량적으로 증명.

> 이 성질이 왜 중요한가? — **"고정된 격자가 아니라, 내용에 따라 스스로 위치를 조절하는 샘플링"**이라는 개념 자체가 이제 곧 볼 Deformable Attention의 핵심 아이디어이기 때문입니다.

### 3.5 DCN → Deformable DETR으로 가는 다리

DCN은 "sparse(희소)하고 적응적인 샘플링"이라는 장점은 있지만, 결정적 한계가 있습니다:

> **DCN은 element(픽셀)들 사이의 "관계"를 모델링하는 능력이 없다.** 필터는 그저 고정된 개수(예: 9개)의 위치를 샘플링해서 고정된 가중치로 더할 뿐, "이 픽셀과 저 픽셀이 서로 얼마나 관련있는지"를 동적으로 계산하지 않습니다.

반대로 Transformer(DETR)는:

> **관계 모델링(relation modeling)은 강력하지만, 모든 픽셀을 다 봐야 해서 느리고 비효율적이다.**

Deformable DETR의 저자들은 정확히 이 지점에서 이렇게 말합니다(Introduction 마지막 문장 취지): *"deformable convolution의 sparse spatial sampling"* + *"Transformer의 relation modeling 능력"* **두 개를 합치자.** 이게 4장의 핵심입니다.

---

## 4. Deformable DETR의 핵심 — Deformable Attention Module

### 4.1 아이디어: "모든 곳을 보지 말고, 몇 개의 중요한 점만 찾아서 보자"

일반 Transformer attention (1.2절 Eq.1)은 query 하나가 **모든** key(모든 픽셀, $N_k = HW$개)를 봅니다.

Deformable attention은 query 하나가 **미리 정한 소수의 K개 점만** 봅니다. (논문 기본 설정: $K=4$). 이 K개 점의 **위치 자체도, 각 점을 얼마나 중요하게 볼지도 모두 학습**됩니다 — DCN에서 오프셋을 학습하던 것과 완전히 같은 방식입니다.

### 4.2 수식 뜯어보기 (Eq. 2)

$$\text{DeformAttn}(z_q, p_q, x) = \sum_{m=1}^{M} W_m \left( \sum_{k=1}^{K} A_{mqk} \cdot W'_m\, x(p_q + \Delta p_{mqk}) \right)$$

일반 attention (Eq.1)과 나란히 놓고 비교하면 차이가 명확합니다.

| | 일반 Transformer Attention (Eq.1) | Deformable Attention (Eq.2) |
|---|---|---|
| 보는 대상 | $\sum_{k \in \Omega_k}$ : **모든** key (최대 $HW$개) | $\sum_{k=1}^{K}$ : 딱 **K개**(예: 4개)의 샘플링 점 |
| key의 위치 | 고정 (이미지의 모든 픽셀 위치) | $p_q + \Delta p_{mqk}$ : **reference point $p_q$ 주변에서 오프셋만큼 이동한, 학습된 위치** |
| 가중치 계산 방법 | $z_q$와 $x_k$를 내적해서 유사도 계산 (query-key 비교) | $z_q$를 선형변환해서 **직접 예측** (query-key 비교 없음!) |
| 위치가 정수가 아니어도 됨? | 애초에 픽셀 위치라 정수 | 실수 가능 → **양선형 보간** 사용 (DCN과 동일) |

하나씩 설명하겠습니다.

- **$p_q$ (reference point, 기준점)**: 이 query가 "대략 어디를 보고 있는지"의 중심 좌표. 인코더에서는 그냥 query 픽셀 자기 자신의 위치이고, 디코더에서는 object query의 임베딩으로부터 학습된 선형변환 + sigmoid로 예측됩니다.
- **$\Delta p_{mqk}$ (sampling offset)**: reference point로부터 얼마나 떨어진 곳을 볼지의 오프셋. $m$번째 head, $k$번째 샘플 점마다 다른 오프셋을 가집니다. **DCN의 $\Delta p_n$과 완전히 같은 개념**이지만, DCN은 고정된 격자 위치에서의 이동이었다면, 여기서는 아예 **"기준점 하나에서 사방으로 흩어지는" K개의 자유로운 점**입니다.
- **$A_{mqk}$ (attention weight)**: 그 샘플링 점을 얼마나 중요하게 볼지의 가중치. $\sum_k A_{mqk}=1$로 정규화(softmax).
- **결정적 차이 — 가중치를 "계산"이 아니라 "예측"한다**: 일반 attention은 $A_{mqk} \propto \exp(z_q^T \cdot x_k)$처럼 **query와 key를 직접 비교(내적)**해서 유사도를 구합니다. 반면 deformable attention은 $\Delta p_{mqk}$와 $A_{mqk}$ **둘 다 오직 query feature $z_q$ 하나만 가지고** 선형변환(Linear layer) 한 방으로 예측해버립니다. (Figure 2에서 "Query Feature $z_q$" 하나가 왼쪽 위에서 시작해 Sampling Offsets과 Attention Weights 두 갈래로 뻗어나가는 걸 보면 이 구조가 명확합니다.)

  > 구현 관점: $z_q$를 $3MK$ 채널짜리 벡터로 projection 한 뒤, 앞의 $2MK$개는 오프셋(2차원 좌표 × M head × K개), 나머지 $MK$개는 softmax를 거쳐 attention weight가 됩니다.

- **$x(p_q + \Delta p_{mqk})$**: 이렇게 정해진 (실수) 위치에서 feature map 값을 양선형 보간으로 읽어옵니다. **DCN의 핵심 연산을 그대로 재사용**하는 부분입니다.

### 4.3 Figure 2 그림으로 정리

논문 Figure 2를 텍스트로 재구성하면 이런 흐름입니다.

```
Query Feature z_q  ──┬─► Linear → Sampling Offsets {Δp_mqk}  (Head별 K개씩)
                      │
                      └─► Linear → Softmax → Attention Weights {A_mqk}

Input Feature Map x + Reference Point p_q
   → (Head 1,2,3 각각) Values = W'_m · x(p_q + Δp_mqk)  (양선형 보간으로 K개 지점 값 추출)

Head별로: Attention Weights × Values → Aggregate (가중합)
   → 각 Head의 결과를 이어붙여 Linear → 최종 출력
```

즉, **"어디를 볼지(offset)"와 "얼마나 중요하게 볼지(weight)"를 전부 query feature 하나로부터 미리 예측**해두고, 그 K개 점의 값만 뽑아서 가중합하는 매우 가벼운 연산입니다.

### 4.4 복잡도가 왜 줄어드는가 (직관 + 수식)

일반 attention 복잡도: $O(N_q N_k C)$ ← $N_k = HW$가 크면 폭발.

Deformable attention 복잡도(논문 Appendix A.1, 대략): $O(N_q C^2 + \min(HWC^2, N_q K C^2))$

핵심은 $N_k$(전체 픽셀 수) 항이 곱셈으로 들어가는 대신, **K(예: 4)라는 작은 상수**로 대체된다는 점입니다. $K \ll HW$이므로 이미지가 아무리 커져도 query 하나당 계산량은 거의 고정됩니다.

- 인코더에 적용($N_q = HW$)하면 복잡도가 $O(HWC^2)$로, **이미지 크기에 대해 선형(linear)** — 기존의 제곱(quadratic)에서 크게 개선.
- 디코더 cross-attention에 적용($N_q = N$, object query 개수)하면 복잡도가 $O(NKC^2)$로, **이미지 해상도 $HW$와 아예 무관**해집니다.

→ 이 덕분에 **고해상도 multi-scale feature map을 여러 장 동시에 써도 감당 가능**해졌고, 이것이 작은 물체 성능 향상의 직접적인 원인이 됩니다.

### 4.5 Multi-scale Deformable Attention (Eq. 3) — 진짜 사용되는 버전

실제 Deformable DETR은 위의 단일 스케일 버전을 그대로 쓰지 않고, **여러 해상도의 feature map을 동시에** 다루는 확장판을 씁니다.

$$\text{MSDeformAttn}(z_q, \hat{p}_q, \{x^l\}_{l=1}^{L}) = \sum_{m=1}^{M} W_m \left( \sum_{l=1}^{L} \sum_{k=1}^{K} A_{mlqk} \cdot W'_m\, x^l(\phi_l(\hat{p}_q) + \Delta p_{mlqk}) \right)$$

Eq.2와 비교하면 딱 하나 늘어난 것: **레벨(level) $l$에 대한 합 $\sum_{l=1}^{L}$이 추가**되었습니다.

- $L$: feature map 레벨 수 (Deformable DETR은 $L=4$, ResNet의 C3~C5 + 추가로 만든 C6 저해상도 한 장)
- $\hat{p}_q \in [0,1]^2$: 정규화된 좌표 (0,0)=왼쪽위, (1,1)=오른쪽아래. 이렇게 정규화해두면 해상도가 다른 각 레벨에 같은 기준점을 적용하기 쉽습니다.
- $\phi_l(\cdot)$: 정규화 좌표를 $l$번째 레벨의 실제 픽셀 좌표로 변환하는 함수.

**직관**: 이제 각 query는 "한 해상도의 K개 점"이 아니라, **"L개 해상도 각각에서 K개씩, 총 $L \times K$개 점"**을 봅니다. 예: $L=4, K=4$면 한 query가 총 16개 점만 보고 그 정보를 종합합니다. 이 16개조차도 전체 픽셀 수(수만~수십만 개)에 비하면 극히 일부입니다.

> 왜 이게 좋은가? — 작은 물체는 고해상도 레벨(예: stride 8)에서, 큰 물체는 저해상도 레벨(예: stride 64)에서 더 잘 보입니다. 하나의 query가 **동시에 여러 해상도를 다 살펴보고 자기한테 필요한 정보를 종합**할 수 있으므로, 별도의 FPN(top-down 연결 구조) 없이도 멀티스케일 정보 교환이 attention 안에서 자연스럽게 일어납니다. (실제로 논문은 FPN을 추가해도 성능이 더 안 오른다는 것을 Table 2 ablation에서 보여줍니다 — 이미 attention이 그 역할을 하고 있기 때문.)

### 4.6 Deformable Conv와의 정확한 관계 (논문이 직접 언급하는 특수 케이스)

논문 원문 취지: **$L=1, K=1$이고 $W'_m$을 항등행렬로 고정하면, multi-scale deformable attention은 정확히 deformable convolution으로 퇴화(degenerate)한다.**

이 문장이 이 논문의 정체성을 가장 잘 설명합니다. 표로 정리하면:

| | Deformable Convolution | Deformable Attention (단일 head, K=1) |
|---|---|---|
| 샘플링 점 개수 | 커널 크기만큼 (예: 9개, 각각 오프셋 학습) | K개 (기본 4개) |
| 스케일 | 단일 스케일만 | **다중 스케일 동시 처리** |
| 관계 모델링 | 없음 (각 위치가 독립적) | **있음** (Query-Key 개념 유지, multi-head로 여러 관점) |
| 위치 특성 | conv처럼 이미지 전체에 sliding window로 반복 적용 | 각 query(위치/object)마다 독립적으로 오프셋·가중치 예측 |

즉, **Deformable Attention = "여러 스케일을 보고, 여러 head로 관계까지 고려할 수 있게 일반화된 Deformable Convolution"** 이라고 이해하면 정확합니다. 반대로 샘플링 점들이 만약 "가능한 모든 위치"를 다 훑는다면, 이 attention은 다시 일반 Transformer attention과 동등해집니다(논문이 명시). 즉 Deformable Attention은 **Deformable Conv와 Transformer Attention의 두 극단을 잇는 스펙트럼 위의 효율적인 중간 지점**입니다.

---

## 5. 전체 아키텍처 뜯어보기 (Figure 1)

Figure 1의 흐름을 순서대로:

```
[Image] → CNN backbone(ResNet) → [Image Feature Maps, 여러 해상도]
   → 1×1 conv로 채널 통일(256) + stride2 conv 하나 추가 → Multi-scale Feature Maps (L=4개 레벨)
   → [Encoder] × 6 layers  (Multi-scale Deformable Self-Attention)
   → [Decoder] × 6 layers  (여기 안에 두 종류의 attention이 섞여 있음)
        - Self-Attention (object query끼리, 기존 Transformer attention 그대로)
        - Multi-scale Deformable Cross-Attention (object query → encoder 출력)
   → Bounding Box Predictions (N=300개 object query 각각에서 class + bbox 예측)
```

### 5.1 인코더 (Deformable Transformer Encoder)

- 입력과 출력 모두 **같은 해상도의 multi-scale feature map**입니다(단순 정제, 해상도 변화 없음).
- ResNet의 C3, C4, C5 출력을 1×1 conv로 채널을 256으로 맞추고, C5에 3×3 stride-2 conv를 하나 더 씌워 C6(가장 저해상도)까지 만들어서 총 **L=4개 레벨**을 씁니다. (Appendix Figure 4 참고 — FPN의 top-down 연결 없이 단순 병렬 구성)
- self-attention이므로 query와 key가 모두 "이미지 픽셀 자기 자신"입니다. 각 픽셀 query의 reference point는 **자기 자신의 위치**입니다.
- 레벨을 구분하기 위해 위치 임베딩(positional embedding)에 더해 **scale-level embedding** $e_l$을 추가로 더해줍니다. 이건 고정된 함수가 아니라 **레벨별로 랜덤 초기화 후 학습되는 벡터**입니다. (같은 좌표라도 "몇 번째 해상도 레벨이냐"를 모델이 구분할 수 있게 하는 역할)

### 5.2 디코더 (Deformable Transformer Decoder)

디코더는 self-attention과 cross-attention 두 종류가 있는데, **딱 cross-attention만 deformable로 교체**했습니다 (self-attention은 object query끼리의 관계라 key가 이미지 픽셀이 아니므로 원래 방식 그대로 유지).

- **Cross-attention (deformable)**: object query가 encoder 출력(이미지 특징)에서 정보를 뽑아옵니다. 각 object query마다 reference point $\hat{p}_q$를 **object query 임베딩으로부터 선형변환+sigmoid로 예측**합니다. → 이 reference point가 "이 object query가 지금 대략 어디를 보고 있는지"의 초기 추정치 역할을 합니다.
- **Bbox를 예측하는 방식이 독특합니다**: 최종 bbox를 절대좌표로 바로 예측하지 않고, **reference point 대비 상대적인 offset**으로 예측합니다 (Appendix A.3). 즉 "기준점에서 얼마나 이동해야 진짜 박스 중심인가"를 배우는 것 — reference point가 이미 박스 중심의 "초기 추측값" 역할을 해주므로 학습이 훨씬 쉬워집니다. 이 설계 덕분에 학습된 decoder attention이 예측된 bbox와 강하게 연관되고, 이게 수렴을 더 가속시킵니다.

---

## 6. 추가 기법 — Deformable DETR이 열어준 두 가지 변형

Deformable DETR이 빠르고 메모리 효율적이라, 저자들은 여기에 다시 살을 붙인 두 가지 변형을 추가로 제안합니다.

### 6.1 Iterative Bounding Box Refinement (반복적 박스 정제)

- Optical flow 추정 기법(RAFT)에서 영감을 받음.
- 디코더가 6개 층(layer)으로 쌓여있는데, **각 층이 이전 층이 예측한 bbox를 그대로 다음 층의 "기준점"으로 넘겨받아 더 정교하게 다듬는** 방식입니다.
- $d$번째 층은 $(d-1)$번째 층이 예측한 박스 $\hat b_q^{d-1}$을 기준으로 상대 offset $\Delta b_q^d$만 예측 → 누적해서 점점 정확해짐.
- 게다가 샘플링 오프셋 $\Delta p_{mlqk}$도 **이전 층이 예측한 박스 크기로 스케일링**됩니다. 즉 박스가 작아지면 샘플링 범위도 좁아지고, 커지면 넓어짐 — 박스 크기에 맞춰 attention의 "탐색 반경"이 같이 조절되는 것입니다.
- 안정성을 위해, 그래디언트는 $\Delta b_q^d$ 방향으로만 흐르고 이전 층 예측값 쪽으로는 역전파를 막습니다(stop-gradient).

### 6.2 Two-Stage Deformable DETR

- 기존 DETR/Deformable DETR은 object query가 이미지와 무관하게 랜덤 초기화된 임베딩이었습니다(이미지를 보기 전부터 "질문 300개"가 고정됨). Two-stage는 이걸 개선합니다.
- **1단계**: 디코더 없이 인코더만으로, feature map의 **모든 픽셀 각각을 하나의 "object query 후보"**로 보고 각 픽셀에서 바로 bbox를 예측(마치 anchor-free detector처럼). 여기서 점수 높은 상위 N개를 뽑아 "region proposal"로 사용합니다. (NMS는 사용하지 않음)
- **2단계**: 이 proposal들을 2단계 디코더의 초기 reference point / object query로 넣어서 iterative refinement로 다듬습니다.
- 효과: 이미지 내용을 미리 반영한 "그럴듯한 시작점"에서 출발하므로 최종 성능이 더 좋아집니다 (Table 1에서 AP 43.8 → 45.4 → 46.2로 순차 개선 확인 가능).

---

## 7. 실험 결과 해석

### 7.1 Table 1 — DETR과의 정면 비교

| Method | Epochs | AP | $AP_S$ | GPU시간 | FPS |
|---|---|---|---|---|---|
| DETR | 500 | 42.0 | 20.5 | 2000 | 28 |
| DETR-DC5 | 500 | 43.3 | 22.5 | 7000 | 12 |
| **Deformable DETR** | **50** | **43.8** | **26.4** | **325** | 19 |
| + iterative box refine | 50 | 45.4 | 26.8 | 325 | 19 |
| ++ two-stage | 50 | 46.2 | 28.8 | 340 | 19 |

읽는 법:
- **Epochs 열**: DETR은 500, Deformable DETR은 50 — 정확히 **10배 적은 epoch**로 비슷하거나 더 좋은 AP를 냄 (논문 abstract의 핵심 주장 실증).
- **$AP_S$ 열**: 20.5 → 26.4로 껑충 뜀. 작은 물체에서 개선폭이 가장 큽니다. (전체 AP 개선폭보다 $AP_S$ 개선폭이 더 큼 — 즉 이 방법의 이득이 "작은 물체"에 유독 크게 쏠려 있다는 뜻)
- **속도(FPS)**: Deformable DETR이 DETR-DC5보다 오히려 빠릅니다(19 vs 12) — DETR-DC5는 고해상도 feature map(dilated C5)을 억지로 써서 정확도는 챙겼지만 메모리 접근이 많아 느려진 반면, deformable attention은 애초에 K개만 접근하므로 고해상도를 써도 덜 느려집니다.

### 7.2 Figure 3 — 수렴 곡선

가로축 epoch, 세로축 AP인 그래프에서, DETR-DC5(회색)는 500 epoch까지 서서히 올라가 43.6에 도달하는 반면, Deformable DETR(빨간색)은 **50 epoch 근방에서 이미 43.8~45.5**에 도달합니다. 즉 그래프 자체가 "왼쪽에서 훨씬 빨리 위로 치고 올라가는" 모양이며, 이게 "10× less training epochs" 주장의 시각적 증거입니다.

### 7.3 Table 2 — Ablation(구성요소 하나씩 떼어보기)

| MS inputs | MS attention | K | AP | $AP_S$ |
|---|---|---|---|---|
| (단일 스케일, K=1) | 없음 | 1 | 39.7 | 21.2 |
| ✓(멀티스케일 입력만) | 없음(레벨간 attention 안함) | 1 | 41.4 | 24.1 |
| ✓ | 없음 | 4 | 42.3 | 24.8 |
| ✓ | ✓ | 4 | **43.8** | **26.4** |

해석:
- 맨 위 줄(단일 스케일, K=1)은 사실상 **deformable convolution과 거의 동일한 설정**입니다(4.6절에서 설명한 특수 케이스). 성능이 가장 낮습니다(39.7 AP).
- "MS inputs(멀티스케일 입력을 쓰는가)"를 켜는 것만으로 AP가 1.7 오르고 $AP_S$는 2.9나 오릅니다 — **작은 물체 성능은 멀티스케일 여부에 가장 민감**하다는 것을 보여주는 실험적 증거.
- K(샘플링 점 개수)를 1→4로 늘리면 추가로 0.9 AP 개선.
- "MS attention(레벨 간 정보 교환을 attention이 직접 하는가)"까지 켜면 추가 1.5 AP 개선.
- 참고로 FPN을 별도로 추가해도(표에는 없지만 본문 언급) 성능이 더 오르지 않는다고 명시 — 이미 attention이 레벨 간 정보 교환 역할을 대신하기 때문.

### 7.4 Table 3 — 최종 SOTA 비교

ResNeXt-101 + DCN(!) 백본을 쓰면 50.1 AP, 여기에 TTA(수평 flip + 멀티스케일 테스트)까지 더하면 52.3 AP. 흥미로운 점은, **Deformable DETR의 backbone으로 (진짜) Deformable Convolution을 사용**하면 성능이 더 오른다는 것 — 즉 "deformable attention"과 "deformable convolution"은 서로 대체재가 아니라 **함께 쓸 수 있는 보완적 기법**입니다.

### 7.5 Appendix A.5, A.6 — "모델이 실제로 어딜 보는가" 시각화

- Figure 5: 예측한 각 항목(x좌표, y좌표, 너비, 높이, 클래스 점수)에 대한 gradient(민감도)를 픽셀별로 시각화 → **박스의 x/y/w/h는 물체의 경계(극점, extremity)에, 클래스 점수는 물체 내부 픽셀에도** 민감하게 반응함을 확인. DETR과 달리 Deformable DETR은 "내부 정보"도 활용해서 분류를 한다는 차이가 있습니다.
- Figure 6: 실제로 학습된 샘플링 점들을 이미지 위에 찍어보면, 인코더 단계에서 이미 같은 물체(instance)에 속한 점들끼리 뭉치는 경향을 보이고, 디코더 단계에서는 물체의 극점뿐 아니라 전체 영역에 퍼져서 샘플링합니다.

---

## 8. 내 연구(Small Object Detection)에 어떻게 연결할지

말씀하신 방향(참조하시는 논문 스타일 — self-reconstruction 모듈로 feature를 원본 이미지로 복원해서 diff map을 만들고, 그 diff map으로 소형 객체의 위치를 추정 → feature enhancement)을 Deformable DETR과 엮으려는 아이디어는 구조적으로 궁합이 아주 좋습니다. 왜 그런지, 그리고 어디를 건드려야 할지를 이 논문의 용어로 짚어보겠습니다.

**핵심 관찰**: Deformable Attention의 "어디를 볼지"는 전부 두 가지 요소로 결정됩니다.
1. **reference point $p_q$** — "대략 어디를 볼지"의 출발점
2. **sampling offset $\Delta p_{mqk}$** — 그 출발점에서 실제로 얼마나/어느 방향으로 벗어나서 볼지 (오직 query feature $z_q$만으로 예측됨, 4.2절)

지금 원 논문은 이 두 값을 **오직 학습 가능한 파라미터**(reference point는 object query 임베딩에서, offset은 query feature에서)로만 결정합니다. 즉 "diff map처럼 명시적으로 계산된 외부 신호"가 이 과정에 전혀 관여하지 않습니다. 여기가 당신이 개입할 지점입니다.

가능한 접근 방향을 몇 가지로 나누면:

1. **Reference point를 diff map 기반으로 초기화/보정 (가장 자연스러운 진입점)**
   - 현재 디코더의 reference point는 "object query 임베딩 → Linear → sigmoid"로만 예측됩니다 (5.2절). 당신의 diff map에서 "소형 객체가 있을 것 같은 위치"를 뽑아, 이를 reference point의 prior(사전 정보) 혹은 초기값으로 주입하면, two-stage Deformable DETR(6.2절)의 "region proposal" 역할을 diff map이 대신하는 셈이 됩니다. Two-stage 구조를 참고해서, 1단계 proposal 생성 로직을 당신의 self-reconstruction/diff map 모듈로 교체하는 그림을 그려볼 수 있습니다.

2. **Sampling offset의 탐색 범위를 diff map으로 변조(modulate)**
   - Iterative box refinement(6.1절)에서 이미 "박스 크기로 offset을 스케일링"하는 선례가 있습니다. 같은 방식으로, diff map에서 나온 "이 지점이 소형 객체일 확률/에너지"를 offset 예측에 곱해주는 modulation term을 추가하면, 소형 객체로 추정된 영역에서 K개의 샘플링 점이 더 촘촘하게/정확하게 모이도록 유도할 수 있습니다. (DCNv2가 실제로 이런 "modulation scalar"를 도입한 전례가 있으니, 관련 문헌으로 참고할 가치가 있습니다.)

3. **Encoder 단(인코더)에서 개입 — feature enhancement를 deformable self-attention 전에 삽입**
   - 인코더의 self-attention에서 reference point는 "자기 자신의 픽셀 위치"로 고정되어 있습니다(5.1절). 여기서는 위치를 바꾸기보다, **diff map으로 강화된 feature를 encoder 입력으로 넣어주는** 방식이 구조적으로 더 깔끔합니다. 즉 원 논문의 attention 메커니즘 자체는 건드리지 않고, "무엇을 볼지 결정하는 재료(query/key feature)"를 강화하는 접근.

4. **Multi-scale 레벨 선택에 diff map을 활용**
   - 4.5절에서 각 query가 $L$개 레벨을 모두 살펴본다고 했는데, 만약 diff map이 "이 위치는 특히 고해상도(L=1, stride 8) 레벨에서 신호가 강하다"는 정보를 준다면, 그 레벨에 대한 attention weight $A_{mlqk}$ 초기화나 loss에 보조 신호(auxiliary supervision)로 줄 수 있습니다.

**실험 설계 관점 팁**: 이 논문의 Table 2 같은 ablation 구성(하나씩만 켜고 끄기)을 그대로 벤치마킹해서, "diff map 개입을 (a) reference point에만, (b) offset에만, (c) 둘 다"로 나눠 ablation을 짜면 논문 리뷰어에게 설득력 있는 실험 설계가 될 것입니다. 또한 $AP_S$ 단독 개선폭을 반드시 강조해서 리포트하는 것이 이 계열 논문들의 공통된 스토리텔링 방식입니다 (7.1, 7.3절 참고).

---

## 9. 한 장 요약 (다른 사람에게 설명할 때 쓸 스크립트)

> "DETR은 Transformer로 물체 검출을 end-to-end로 푼 첫 시도인데, attention이 이미지의 모든 픽셀을 다 보다 보니 계산량이 이미지 크기의 제곱으로 늘어나서 고해상도를 못 쓰고, 그래서 작은 물체를 잘 못 잡았고, 학습도 500 epoch씩 걸렸어요.
>
> Deformable DETR은 여기에 2017년에 나온 deformable convolution의 아이디어 — '고정된 격자 대신, 학습된 오프셋만큼 이동한 곳을 샘플링하자' — 를 attention에 이식했어요. 그래서 이제 각 query는 이미지 전체가 아니라, 자기 위치(reference point) 주변에서 학습으로 정해진 K개(보통 4개) 점만 보고, 그것도 여러 해상도(L개 레벨)에서 동시에 봐요. 이 K개 점의 위치와 중요도는 전부 query feature 하나로부터 예측되고, Deformable Conv처럼 양선형 보간으로 값을 읽어와요.
>
> 그 결과 계산량이 이미지 크기에 선형(또는 무관)으로 줄어서, 여러 해상도의 feature map을 FPN 없이도 동시에 쓸 수 있게 됐고, 이게 작은 물체 성능을 크게 끌어올렸어요($AP_S$ 20.5→26.4). 학습도 10배 적은 epoch(50)에 수렴해요.
>
> 여기에 추가로 iterative bounding box refinement(디코더 레이어마다 이전 박스를 기준으로 점점 정교하게 다듬기)와 two-stage(인코더만으로 먼저 region proposal을 뽑고 그걸 디코더 초기값으로 쓰기) 두 변형을 얹어서 최종 46.2 AP까지 올렸어요."

---

## 10. 검수 노트 — 원문 대조 체크리스트

이 자료를 만들면서 원문과 대조한 결과, 아래 항목들을 특히 신경 써서 정확하게 반영했습니다.

- ✅ Eq.1(일반 attention)과 Eq.2(deformable attention)의 수식 요소(합의 범위, 가중치 계산 방식)를 정확히 대조 — 원문이 강조하는 "query-key 비교 없이 query feature만으로 offset/weight를 예측한다"는 포인트를 놓치지 않고 반영했습니다.
- ✅ 복잡도 분석(Appendix A.1)의 핵심 결론($O(HWC^2)$ 인코더, $O(NKC^2)$ 디코더)을 원문 그대로 인용하되, 유도 전 과정을 전부 옮기지는 않고 결론과 직관 위주로 축약했습니다(전체 유도가 필요하시면 원문 Appendix A.1을 별도로 같이 보시길 권합니다).
- ✅ "$L=1, K=1, W'_m$=identity일 때 deformable convolution으로 퇴화한다"는 원문의 핵심 이론적 주장(4.1절 본문)을 정확히 반영했습니다 — 이 부분이 두 논문을 잇는 가장 중요한 다리입니다.
- ✅ Table 1/2/3의 수치는 원문 표 값을 그대로 사용했습니다.
- ✅ Two-stage와 iterative refinement는 각각 별개의 옵션이며 Table 1에서 "+"와 "++"로 순차 누적되는 것으로 표기되어 있음을 명확히 반영했습니다 (둘을 합쳐야 46.2 AP).
- ⚠️ **주의(원문에서 다소 헷갈릴 수 있는 부분)**: Two-stage의 1단계는 "디코더 없이 인코더만 사용"한다고 명시되어 있는데, 이는 self-attention 기반 object-query 상호작용의 계산 비용($O(N^2)$)이 픽셀 수만큼 많은 query에서는 감당이 안 되기 때문입니다 — 이 이유를 원문에서 놓치기 쉬워 본 자료 6.2절에 명시적으로 설명을 추가했습니다.
- ⚠️ Deformable RoI Pooling(DCN 3.3절)은 Deformable DETR 본문에서는 사용되지 않습니다(DETR류는 RoI 개념 자체가 없음). 다만 Table 3의 SOTA 비교에서 backbone에 "DCN"을 적용한 결과가 등장하므로, 완전히 무관하지는 않다는 점을 7.4절에 짚어두었습니다.
- ⚠️ 원문 Figure 5, 6, 7(정성적 시각화)은 이미지 자체를 재현할 수 없어 캡션과 본문 설명을 근거로 **글로 해석**했습니다. 실제 그림을 함께 보시면서 이 설명을 대조하시길 권장합니다(PDF 원문의 해당 페이지 참고).
- 누락 없음 확인: Related Work(2장)의 세부 비교 대상들(Reformer, Linformer 등 효율적 attention 계열)은 "우리 접근법이 어느 카테고리에 속하는지"를 설명하는 문헌 리뷰 성격이라 본 스터디 자료에서는 실험/구조 이해에 필수적이지 않다고 판단해 요약만 하고 상세 나열은 생략했습니다 (필요하시면 알려주시면 추가하겠습니다).

---

### 참고: 표기법 대조표 (Appendix A.7, 원문 Table 4 재구성)

| 기호 | 의미 |
|---|---|
| $m$ | attention head 인덱스 |
| $l$ | key의 feature 레벨 인덱스 |
| $q, k$ | query / key 인덱스 |
| $M, L, K$ | head 개수 / feature 레벨 개수 / 레벨당 샘플링 점 개수 |
| $C, C_v$ | 전체 feature 차원 / head당 feature 차원 |
| $z_q, p_q, \hat p_q$ | query의 feature / 기준점 좌표 / 정규화된 기준점 좌표 |
| $x, x^l$ | (레벨별) 입력 feature map |
| $\Delta p_{mqk}, \Delta p_{mlqk}$ | 샘플링 오프셋 |
| $A_{mqk}, A_{mlqk}$ | attention weight |
| $\sigma, \sigma^{-1}$ | sigmoid / inverse sigmoid |
