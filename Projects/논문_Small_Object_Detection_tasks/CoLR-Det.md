---
pm-task: true
projectId: "paperwiki-small-object-detection"
parentId:
id: "t-colr-det-d6nrnr1pzn"
title: "CoLR-Det"
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
  "1frf59rymtcjvske": "arXiv"
subtaskIds: []
dependencies: []
year: 2026
venue: "arXiv"
jcr_quartile: arXiv
task: [small-object-detection]
direction: [improvement]
paper_tags: [paper, small-object-detection, remote-sensing, low-resolution, super-resolution, detr, latent-regularization, token-routing, saliency]
source: "/Users/GyeongSeo/Workspace/논문_pdf/Small_Object_Detection/2026_arXiv_CoLR-Det.pdf"
createdAt: "2026-08-24T03:36:00.000Z"
updatedAt: "2026-08-28T17:00:00.000Z"
---

#paper #small-object-detection #remote-sensing #low-resolution #super-resolution #detr #latent-regularization #token-routing #saliency

> [!quote] 원제
> **CoLR-Det: Collaborative Latent Restoration for Small Object Detection in Low-Resolution Remote Sensing Images**
> Ruo Qi, Linhui Dai, Yusong Qin, Chaolei Yang, Yanshan Li — Shenzhen University, arXiv 2026

# 한 줄 요약
<mark style="background: #FFF3A3A6;">Super-resolution(SR)을 추론 시 이미지 복원 단계가 아니라 학습 시에만 encoder를 정규화하는 latent 제약으로 재정의하고, saliency 기반 비파괴적 token routing으로 배경 텍스처가 DINO 기반 detection encoder를 오염시키지 않게 막는 CoLR-Det — 저해상도 원격탐사 3개 벤치마크에서 Swin-T DINO baseline 대비 AP 최대 7.0%p, AP_s 최대 16.2%p 개선.</mark>

> [!info] 내 메모
> 

# 정리

| | 문제 ① — Restoration-first 인터페이스의 목표 불일치 | 문제 ② — 배경 텍스처의 detection feature 오염 | 문제 ③ — SR·detection 최적화 선호 충돌 |
|---|---|---|---|
| **문제 정의** | 기존 SR 보조 탐지기는 이미지·영역·feature를 명시적으로 복원해 화질을 높인 뒤 detector에 넣는다. 이는 "시각적으로 충실한 복원이 인식에 도움된다"는 암묵적 전제인데, SR은 조밀한 텍스처·edge fidelity를 우선하는 반면 detection은 sparse한 instance-level semantic 증거에 의존해 두 목표가 근본적으로 어긋난다(Fig. 1). | SR supervision은 텍스처가 풍부한 배경 구조·edge를 강조하는 경향이 있어, 이 복원 신호를 그대로 detection feature에 주입하면 배경 텍스처가 객체 semantic을 오히려 오염시킨다(Fig. 1(d)에서 SR-지향 응답이 배경 영역과 크게 겹침). | Dense한 pixel-level 복원 목표와 sparse한 semantic 목표는 서로 다른 gradient 방향을 요구한다. Stepwise(독립 전처리)는 closed-loop 최적화가 없어 SR이 화질은 올려도 detection 성능은 개선 못 하거나 pseudo-texture로 악화시키고, joint 최적화는 두 loss를 처음부터 함께 최적화해 gradient 불일치로 수렴을 저해한다. |
| **풀고자 하는 문제** | SR을 명시적 이미지 복원이 아니라 순수 latent 정규화로 재구성해, 추론 시 SR 연산 자체를 완전히 제거하는 것 | 복원 신호가 배경 텍스처로 detection feature를 오염시키지 않도록, 정보는 잃지 않으면서 선택적으로 필터링하는 것 | Dense SR supervision과 sparse detection supervision의 최적화 선호도를 학습 스케줄로 조율하는 것 |
| **선행 연구 접근** | - Stepwise(Shermeyer & Van Etten [35], Yei et al. [36]): SR을 독립 전처리로 사용<br>- Joint(Kim et al. [37] SR4IR, Wu & Xu [38]): SR-detection을 직렬 파이프라인으로 묶어 end-to-end 학습<br>- Selective(HSOD-Net [39]): 키포인트로 관심 영역만 SR<br>**갭**: 세 갈래 모두 "SR이 이미지/영역/feature를 명시적으로 향상시킨다"는 restoration-first 인터페이스를 공유 — 이 인터페이스 자체가 detection-optimal이 아닐 수 있다는 문제는 다루지 않음. | 위와 동일한 restoration-first 계열 전체가 배경 텍스처 억제 메커니즘을 갖고 있지 않음 — HSOD-Net도 hard region cropping으로 관심 영역만 골라낼 뿐, 선택되지 않은 영역의 정보를 남겨두면서 계산만 아끼는 비파괴적 필터링은 없음. | Joint 최적화 계열(SR4IR 등)이 두 loss를 동시에 최적화하려 시도했으나, pixel fidelity와 semantic invariance 간 내재적 충돌을 학습 스케줄 분리가 아니라 단일 손실 함수/가중치 설계로 풀려 해서 gradient 불일치 문제가 남음. |
| **해결 방법** | 학습 시에만 존재하는 latent restoration branch가 공유 encoder에 SR reconstruction loss를 역전파해 encoder representation을 정규화하고, 추론 시 이 브랜치를 통째로 제거한다. | Saliency-guided object-preserving token routing — 고saliency 토큰만 비싼 attention 정제를 받고, 저saliency 토큰은 영구 폐기 대신 bypass 경로로 정보 흐름만 유지한다. | Detection-prioritized 2단계 학습 — 먼저 SR 브랜치를 얼리고 detection semantic만 안정화(Stage 1)한 뒤, SR 브랜치를 보수적인 학습률(scaling factor ρ)로 풀어 공동 최적화(Stage 2). |
| **예상되는 문제점** | SR 브랜치가 학습에만 관여하므로, encoder가 실제로 SR-유용한 표현과 detection-유용한 표현을 얼마나 잘 분리해 담아내는지는 간접적으로만(ablation으로) 확인 가능하다. | Saliency 예측 자체가 학습 초기 불안정할 수 있어(저자 인정), 이 시기에 실제 소형 객체가 저saliency로 잘못 분류되면 bypass되어 충분한 attention 정제를 못 받을 위험이 있다. | 2단계 학습 스케줄의 `T_det`(1단계 길이) 등 하이퍼파라미터에 성능이 민감하며(Table VII), 이 민감도가 데이터셋마다 다를 수 있는지는 NWPU VHR-10-Split에서만 검증되었다. |

**갭 종합**: <mark style="background: #FFF3A3A6;">세 문제 모두 "SR이 이미지/영역/feature를 명시적으로 향상시켜야 한다"는 restoration-first 인터페이스를 공유 전제로 깔고 있다는 공통 원인에서 나온다. 이 논문의 통찰은 "복원이 탐지를 보조해야지 지배해서는 안 된다(restoration should assist detection, rather than dominate it)"는 원칙을 인터페이스 자체의 재설계(latent 정규화 + 비파괴적 라우팅 + 학습 스케줄 분리)로 구현하면, 명시적 이미지 복원 없이도 SR supervision의 이득만 취할 수 있다는 것이다.</mark>

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">Swin-T 기반 파라미터 공유 multiscale encoder 위에, 학습 시에만 존재하는 <span style="color:#c0392b; font-weight:bold;">latent restoration branch</span>로 SR reconstruction loss를 encoder에 역전파해 detection-oriented representation을 정규화하고, <span style="color:#c0392b; font-weight:bold;">saliency-guided object-preserving token routing</span>으로 고saliency 토큰만 비싼 attention 정제를 받게 하되 저saliency 토큰은 영구 폐기 대신 bypass 경로로 정보를 유지하며, <span style="color:#c0392b; font-weight:bold;">detection-prioritized 2단계 학습</span>으로 먼저 detection semantic을 안정화한 뒤 SR supervision을 보수적인 학습률로 추가한다.</mark>

## 전체 파이프라인 (Fig. 2 기준)

```
입력 저해상도 이미지 I_LR (3, H, W)
       │
       ▼
① Parameter-Shared Multiscale Encoder (Swin-T, 4 stage)
   Patch Embedding → [Patch Merging + Swin Transformer] × 4
       │
       ├──────────────────────────────────────────┐
       ▼                                            ▼
   {F_s}, s=1..4  (F_s: (C_s, H_s, W_s),           (학습 시에만 사용)
    C_s∈{96,192,384,768}, 다운샘플 2^(s+1))
       │
       ├─────────────────────────────┐
       ▼                             ▼
② Latent Restoration Branch      ③ Saliency-Guided Detection Branch
   (학습 시에만, 추론 시 제거)         │
       │                             ▼
   SR Decoder                  Multiscale Saliency Prediction
   (Patch-Expanding + Swin      → 레벨별 saliency map S_l, (1, H_l, W_l)
    Block, 반복)                     │
       │                             ▼
       ▼                        Query Filtering
   Image Reconstruction         (활성 집합 Φ_t vs bypass 집합으로 분리)
   (Conv→LeakyReLU→                  │
    PixelShuffle→Conv)               ▼
       │                        DINO-based Encoder Layer × N
       ▼                        (활성: top-k pre-attn + deformable
   I_SR (3, H_HR, W_HR)          self-attn + FFN / bypass: 위치임베딩만
   ↓ L1 loss (L_sr)               추가) → (HW, d)
   encoder로 역전파                    │
                                      ▼
                                 DINO-based Decoder (object query)
                                      │
                                      ▼
                                 Detection Output (클래스 + 박스)
```

> [!info] 내 메모
> 

### ① Parameter-Shared Multiscale Encoder
- **역할**: 저해상도 입력 이미지를 detection과 SR 재구성 양쪽에서 **공유해서 쓰는** 다중 스케일 latent feature로 변환한다. 하나의 encoder를 두 task가 함께 쓰기 때문에("cross-task parameter-shared"), SR 쪽에서 오는 학습 신호가 이 encoder 자체를 detection에도 이롭게 바꿔놓을 수 있다는 것이 이 논문 전체의 전제다.
- **구현**: Swin Transformer Tiny(Swin-T) 백본. Patch Embedding 이후 4개 계층 stage, 각 stage는 patch merging(공간 해상도를 반으로 줄이고 채널을 늘림) + Swin Transformer 블록(window-based multi-head self-attention(W-MSA)과 shifted-window multi-head self-attention(SW-MSA)을 번갈아 사용, 자세한 attention 동작은 [[Multi_Head_Self_Attention]] 참고, window 방식이라 전체 이미지가 아니라 지역 window 내에서만 self-attention을 계산해 비용을 줄인다는 점이 표준 self-attention과 다르다)로 구성.
- **입출력 shape**: `F_0 = I_LR (3, H, W)` → stage `s`마다 `F_s (C_s, H_s, W_s)`, `s∈{1,2,3,4}`, 공간 해상도는 `2^(s+1)`배 다운샘플, 채널 `C_s∈{96, 192, 384, 768}`.

```python
# 논문 Eq.(1) 기반 의사코드
F = [I_LR]  # F_0
for s in range(1, 5):
    F_s = SwinBlock_s(F[s-1])   # patch merging + Swin Transformer layers
    F.append(F_s)
# F_s: (C_s, H_s, W_s), C_s in {96,192,384,768}, 다운샘플 2^(s+1)
# 학습 중 F_s는 detection loss와 SR loss(L_sr) 양쪽으로부터 동시에 역전파를 받음
```

<mark style="background: #FFF9D6A6;">이 encoder를 SR과 detection이 공유하기 때문에, SR reconstruction loss가 이 encoder의 파라미터를 detection loss와 동시에 갱신할 수 있다 — "정리" 표 문제①에서 언급한 두 목표(복원 vs 인식) 사이의 협업이 여기서 물리적으로 가능해진다(Table IV에서 SR 브랜치 추가만으로 AP가 개선되는 근거).</mark>

> [!info] 내 메모
> 

### ② Latent Restoration Branch
- **역할**: 학습 중에만 존재하며, 공유 encoder가 만든 latent feature로부터 고해상도 이미지 `I_SR`을 재구성해 SR reconstruction loss를 만든다. 이 loss가 encoder로 역전파되면서 encoder가 "저해상도 열화 속에서도 복원 가능한 구조적 단서"를 유지하도록 정규화한다. **추론 시에는 이 브랜치 전체가 제거**되어, detection은 항상 이 학습된 latent representation에서 직접 수행되며 명시적 SR 재구성을 거치지 않는다.
- **구현**: U-Net의 progressive upsampling 경로 + Swin Transformer의 feature modeling을 결합한 계층적 재구성 decoder.
  - Patch-Expanding 모듈(transposed convolution으로 공간 해상도를 2배로 키우고 채널을 재배열)과 residual Swin block을 여러 단 쌓아 latent feature를 점진적으로 업샘플링(Eq. 2).
  - 최종 decoder feature `F_dec`을 Conv → LeakyReLU(비선형 매핑) → PixelShuffle(채널을 공간으로 재배열해 추가 업샘플링) → Conv로 이미지 도메인으로 매핑해 `I_SR` 생성(Eq. 4).
  - Loss: `I_SR`과 GT 고해상도 이미지 사이의 `L1` reconstruction loss(`L_sr`).
- **입출력 shape**: `{F_s}` (encoder의 4단 multiscale feature) → 점진적 업샘플링 → `I_SR (3, H_HR, W_HR)` (`H_HR, W_HR`은 원래의 고해상도 GT 크기, 즉 `I_LR`의 2배 — 2× bicubic downsampling을 되돌리는 크기).

```python
# 논문 Eq.(2), Eq.(4) 기반 의사코드
def patch_expanding(Z_in):
    x = R_in(Z_in)             # 시퀀스 layout으로 재배열
    x = LN(x)
    x = TransConv(x)           # stride 2, 공간 해상도 2배
    return R_out(x)            # 다시 표준 feature map layout으로

z = F_4  # 가장 깊은 latent feature에서 시작
for stage in decoder_stages:
    z = patch_expanding(z)
    z = residual_swin_block(z)
F_dec = z

I_SR = Conv(PixelShuffle(LeakyReLU(Conv(F_dec))))   # 이미지 도메인 복원
L_sr = L1(I_SR, I_HR)   # GT 고해상도 이미지와 비교
```

<mark style="background: #FFF9D6A6;">"정리" 표 문제①(restoration-first 인터페이스가 detection-optimal이 아닐 수 있음)을, SR을 "detector가 받는 출력"이 아니라 "학습 시 encoder에 가하는 보조 제약"으로 재정의해 해결한다 — Table IV에서 SR 브랜치 추가만으로 AP가 0.766→0.816(DINO), 0.778→0.799(Cascade R-CNN)로 개선되어, 명시적 이미지 복원 없이도 SR supervision이 순수하게 표현 학습 정규화로서 유효함을 확인한다. Table V의 Detached 대조 실험(SR gradient를 encoder로 전파하지 않으면 AP가 0.770으로 baseline과 거의 동일)은 이 이득이 "SR 브랜치 자체"가 아니라 "SR gradient가 encoder를 함께 갱신한다"는 데서 나온다는 것을 직접 증명한다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> SR 브랜치가 추론 시 제거되므로, 학습이 끝난 뒤 encoder가 실제로 "SR에 유용한 구조 정보"를 얼마나 유지하고 있는지 직접 검사할 방법이 없다 — Fig. 7의 attention 시각화로 간접 확인할 뿐이다. 또한 Table V의 Infer 변형(추론 시에도 SR head를 실행)이 채택 방식과 성능이 거의 같다는 것은 "SR 브랜치가 추론 시 불필요하다"는 저자 주장의 근거이자, 동시에 "이 브랜치가 encoder 표현에 남긴 영향이 SR head 자체의 실행 여부와 무관하다"는 뜻이므로 그 영향의 정체(어떤 표현이 바뀌었는지)는 여전히 블랙박스로 남는다.

> [!info] 내 메모
> 

### ③ Multiscale Saliency Prediction
- **역할**: Encoder의 각 stage feature마다 "여기가 객체(전경)일 가능성이 얼마나 되는지"를 나타내는 saliency map을 예측한다. 이 saliency map이 다음 단계(④ token routing)에서 어떤 토큰에 비싼 attention을 쓸지 결정하는 근거가 된다.
- **구현**: 각 레벨 feature `F_l`을 local branch(공간 디테일 보존, 별도 conv/MLP 경로)와 global branch(GAP(global average pooling)로 장면 레벨 문맥을 뽑은 뒤 다시 공간 크기로 broadcast)로 나눠 처리한 뒤 채널 방향으로 concat, MLP로 차원을 줄여 단일 채널 saliency 점수 `S_l`을 만든다(Eq. 5).
  - Top-down propagation: 고레벨(deep, semantic이 강한) saliency를 upsample해 저레벨(shallow, 공간 디테일이 강한) saliency 예측과 학습 가능한 modulation 계수 `α`로 결합, 레벨 간 saliency 일관성을 확보(Eq. 6). `α`는 `U(-0.3, 0.3)`에서 초기화되는 학습 파라미터.
  - Saliency confidence target `C_l`: 단순 이진 전경/배경 마스크 대신, 쿼리 위치 `(i,j)`에서 자신을 감싸는 GT 박스의 좌/상/우/하 경계까지의 정규화된 상대 거리(`δ_x, δ_y`)로 정의되는 **연속값 centrality 신호**(Eq. 7) — 객체 중심에 가까울수록 confidence가 1에 가깝고 경계 근처에서 0에 수렴. 각 쿼리는 자신의 stride/receptive field에 대응하는 크기 범위 `(τ_{l-1}, τ_l]`의 객체에만 배정되어, 작은 객체는 얕은 레벨, 큰 객체는 깊은 레벨에 배정됨으로써 스케일 간 간섭을 없앤다.
  - Supervision: sigmoid focal loss 기반 saliency constraint loss `L_sa`(Eq. 8), 예측 saliency `S_l`을 GT confidence `C_l`에 맞추도록 학습.
- **입출력 shape**: `{F_l}_{l=1}^4` → 레벨별 saliency map `{S_l}` (각 `(1, H_l, W_l)`, 원래 feature와 같은 공간 해상도).

```python
# 논문 Eq.(5)~(7) 기반 의사코드
def saliency_prediction(F_l):
    local = local_branch(F_l)                     # 공간 디테일 보존
    global_ctx = broadcast(GAP(F_l))               # 장면 레벨 문맥
    fused = MLP_2(concat([MLP_1(local), global_ctx], dim=channel))
    return fused                                   # S_l: (1, H_l, W_l)

# top-down propagation (Eq.6)
S_prev = alpha * upsample(S_l) + saliency_head(F_{l-1})

# confidence target (Eq.7), (i,j)가 GT 박스 B_GT 안에 있을 때만
delta_x = (l - r) / (l + r)   # 좌우 경계 불균형
delta_y = (t - b) / (t + b)   # 상하 경계 불균형
C_l = 1 - 0.5 * sqrt(delta_x**2 + delta_y**2)   # 객체 중심에서 1, 경계에서 0

L_sa = sigmoid_focal_loss(S_l, C_l).sum() / max(N_pos, 1)
```

<mark style="background: #FFF9D6A6;">단순 이진 전경/배경 마스크가 아니라 연속값 centrality를 supervision target으로 쓰기 때문에, saliency 예측이 "이 영역이 객체 중심에 얼마나 가까운가"라는 gradient 정보를 갖게 된다 — 이는 다음 단계(④)에서 어떤 토큰을 우선 정제할지 순위를 매기는 데 필요한 세밀한 신호이며, "정리" 표 문제②(배경 텍스처 오염)를 막기 위한 선행 조건이다.</mark>

> [!info] 내 메모
> 

### ④ Saliency-Guided Object-Preserving Token Routing
- **역할**: DINO 계열 DETR-style encoder는 이미지 전체의 토큰(HW개)을 전부 self-attention으로 정제하는데, 원격탐사 고해상도 feature에서는 이 중 상당수가 배경 지배적 영역이다. 이 토큰들까지 전부 비싼 attention에 태우면 계산이 낭비될 뿐 아니라, 배경 텍스처가 정제 과정에서 오히려 객체 semantic을 희석시킬 위험이 있다. 이를 막기 위해, ③에서 예측한 saliency 순위에 따라 상위 토큰만 attention 정제를 받게 하고 나머지는 **폐기하지 않고 우회(bypass)**시킨다.
- **구현**:
  - Pyramid-level 유지 비율 `β_t`(레벨별 0.6/0.8/1.0/1.0)와 Transformer layer-wise 유지 비율 `γ_l`(레이어가 깊어질수록 1.0→0.8→0.6→0.6→0.4→0.2로 감소)을 곱해, 매 레벨·레이어마다 top `β_t γ_l N`개 토큰만 활성 집합 `Φ_t`로 선정.
  - 활성 집합 토큰: top-k pre-attention(우선 후보 축소) → deformable self-attention([[Multi_Head_Self_Attention]]의 변형으로, 전체가 아니라 학습된 소수의 샘플링 위치만 참조하는 attention) → FFN(→[[1x1_Convolution]]과 동일 구조) 순으로 비싼 정제를 받음(Eq. 9).
  - Bypass 집합 토큰: attention 계산에서 제외되고, 학습 가능한 위치 임베딩(positional embedding)만 더해져 feature stream에 그대로 유지됨 — **영구 폐기가 아니라 "정제만 건너뛰는" 비파괴적 라우팅**이라는 점이 이 논문이 강조하는 핵심 차별점.
  - 이 라우팅으로 계산 복잡도가 `O(NHK)`에서 `O(β_t γ_l NHK)`로 감소(`H`: attention head 수, `K`: deformable attention이 샘플링하는 key 개수).
- **입출력 shape**: encoder feature 토큰 `(N, d)` + saliency 순위 → 활성 집합 `(β_t γ_l N, d)`(attention 정제) + bypass 집합 `(N - β_t γ_l N, d)`(위치 임베딩만 추가) → 다시 합쳐 `(N, d)`.

```python
# 논문 Eq.(9) 기반 의사코드
def route_and_refine(q, pos, saliency_rank, beta_t, gamma_l):
    active_set = top_k(saliency_rank, k=int(beta_t * gamma_l * N))
    outputs = []
    for i, qi in enumerate(q):
        if i in active_set:
            qi = deformable_self_attention(qi + pos[i], q)   # top-k pre-attn + deform attn + FFN
        else:
            qi = qi + pos[i]   # bypass: 위치 임베딩만 더하고 attention은 생략
        outputs.append(qi)
    return outputs
# 배경 지배적 쿼리의 약 60%가 bypass 경로로 라우팅됨(Fig.8 caption 근거)
```

<mark style="background: #FFF9D6A6;">"버리지 않고 우회시킨다"는 비파괴적 설계로 "정리" 표 문제②(배경 텍스처 오염)를 해결한다 — 학습 초기 saliency 예측이 불안정할 수 있음을 저자가 인지해, 저saliency로 잘못 판정된 실제 소형 객체 토큰도 정보 흐름 자체는 유지되어 완전한 탐지 실패로 이어지지 않는다. Table VI에서 saliency 라우팅을 SR 브랜치와 결합했을 때만 AP_s가 크게 개선(0.552→0.649)되고, saliency 라우팅 단독으로는 오히려 AP_s가 소폭 하락(0.487→0.474)하는 것은 이 상호보완성을 뒷받침한다 — saliency만으로는 부정확한 순위 매김이 약한 소형 객체 응답을 저평가할 수 있지만, SR 브랜치가 만든 더 선명한 latent 표현과 결합하면 saliency 예측 자체의 신뢰도가 개선된다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> 라우팅의 정확도는 전적으로 ③의 saliency 예측 품질에 의존한다. Saliency 예측이 학습 초기에 불안정하면(저자 명시), 이 시기에는 bypass된 토큰이 정제 부족으로 detection에 기여하지 못할 가능성이 있다 — Discussion에서 다시 다룸.

> [!info] 내 메모
> 

### ⑤ DINO-based Encoder/Decoder — Detection Branch
- **역할**: ④에서 라우팅된 토큰을 받아 최종 (클래스, 박스) 예측을 만드는, DINO 구조를 그대로 계승한 detection branch. CoLR-Det 자체의 기여는 이 DINO encoder 내부에 saliency 기반 토큰 라우팅(④)을 끼워 넣은 것이고, decoder 자체는 DINO의 표준 구조를 그대로 사용한다.
- **구현**: DINO(2023)의 encoder-decoder — object query와 encoder memory 간 cross-attention, contrastive denoising training 등 DINO 고유 메커니즘은 이 논문에서 별도로 변경하지 않으므로 자세한 내부 동작은 DINO 원 논문 참고. [[DETR]] 노트의 ④⑤ 절이 이 계열 구조(encoder self-attention, decoder object query)의 기본 동작을 설명한다.
- **입출력 shape**: 라우팅된 encoder feature `(N, d)` → decoder object query 통과 → 클래스 + 박스 예측.

```python
# 의사코드 — DINO 표준 구조, 이 논문은 encoder 내부 토큰 처리(④)만 교체
memory = dino_encoder_with_routing(tokens, pos)   # ④의 라우팅이 여기 내장됨
outputs = dino_decoder(object_queries, memory)    # 표준 DINO decoder
cls_logits, boxes = prediction_heads(outputs)
```

<mark style="background: #FFF9D6A6;">DINO의 검증된 detection 구조를 그대로 유지하면서 encoder 내부 토큰 처리만 교체했기 때문에, "정리" 표에서 다루는 세 문제(복원-인식 불일치, 배경 오염, 최적화 충돌) 해결책을 detection 성능 자체를 낮추는 구조 변경 없이 얹을 수 있다 — Table I~III에서 CoLR-Det이 DINO 대비 항상 AP·AP_s 모두 개선되는 것이 이 "비침습적" 설계의 근거다.</mark>

> [!info] 내 메모
> 

### ⑥ Detection-Prioritized Two-Stage Optimization
- **역할**: SR reconstruction loss(dense, pixel-level)와 detection loss(sparse, instance-level)를 처음부터 동시에 최적화하면 gradient 방향이 어긋나(문제③) 수렴이 저해된다. 이를 학습 스케줄을 두 단계로 나눠서 해결한다 — "언제 어떤 loss를 얼마나 강하게 반영할지"를 시간 축으로 분리하는 방식.
- **구현**(Algorithm 1, Fig. 4):
  - **Stage 1 — Semantic Stabilization**(epoch 1~`T_det`): SR 브랜치 파라미터 `θ_sr`를 고정(`requires_grad=False`)하고, detection loss(`L_cls+L_bbox+L_giou`)와 saliency constraint loss(`L_sa`)만으로 encoder(`θ_feat`)+detector(`θ_det`)를 학습률 `η_feat^(1), η_det^(1)`로 먼저 안정화. `L_stage1 = L_det + β_sa·L_sa`(Eq. 11, `L_det = β_cls·L_cls + β_bbox·L_bbox + β_giou·L_giou`).
  - **Stage 2 — Collaborative Refinement**(epoch `T_det`+1~`T_tot`): `θ_sr`을 unfreeze하되, SR 브랜치 학습률에 scaling factor `ρ∈(0,1)`를 곱해 detection 브랜치보다 보수적으로 갱신(`η_sr^(2) = ρ·η_det^(2)`, Eq. 10). `θ_feat, θ_det, θ_sr` 전체를 `L_stage2 = β_sr·L_sr + L_det + β_sa·L_sa`(Eq. 12)로 공동 최적화.
  - 실험에서 `T_det=16`, `T_tot=52`, `ρ=0.1`, loss 가중치 `β_cls=1.0, β_bbox=5.0, β_giou=2.0, β_sa=2.0, β_sr=1.0`.
- **입출력**: (파라미터 갱신 스케줄이므로 tensor shape 없음 — 학습 절차 자체가 산출물).

```python
# Algorithm 1 요약
for t in range(1, T_det + 1):                       # Stage 1
    L = L_det(batch) + beta_sa * L_sa(batch)
    update(theta_feat, theta_det, lr=lr1, grad=L)   # theta_sr은 고정(freeze)

theta_sr.requires_grad = True
for t in range(T_det + 1, T_tot + 1):               # Stage 2
    lr_sr = rho * lr_det2                            # rho in (0,1), 보수적 학습률
    L = beta_sr * L_sr(batch) + L_det(batch) + beta_sa * L_sa(batch)
    update(theta_feat, lr=lr_feat2, grad=L)
    update(theta_det,  lr=lr_det2,  grad=L)
    update(theta_sr,   lr=lr_sr,    grad=L)
```

<mark style="background: #FFF9D6A6;">"정리" 표 문제③(dense SR supervision과 sparse detection supervision의 최적화 선호 차이)을, 두 손실을 처음부터 동시에 최적화하지 않고 순서(먼저 detection 안정화)와 학습률(SR을 보수적으로)을 분리함으로써 해결한다 — Table VII에서 단일 단계 균등 가중(Single-stage EW)이나 불확실성 기반 자동 가중(Single-stage UW) 모두 이 2단계 전략보다 AP75·AP_s가 낮게 나와, "손실 재가중만으로는 불충분하고 학습 스케줄 자체의 분리가 필요하다"는 것을 실증한다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> `T_det`가 너무 짧으면(예: 8) encoder가 안정적인 semantic을 확립하기 전에 SR gradient가 들어와 개선폭이 제한되고, 너무 길면(예: 24) 그만큼 SR-guided refinement에 쓸 학습 예산이 줄어 성능이 오히려 하락한다(Table VII) — 최적 `T_det`를 찾는 추가 튜닝 비용이 필요하고, 이 최적값이 데이터셋마다 다를 가능성은 검증되지 않았다.

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| ① Multiscale Encoder | (3, H, W) | {F_s}, (C_s, H_s, W_s), s=1..4 | 공유 latent feature 추출 | Swin-T (patch merging + W-MSA/SW-MSA) |
| ② Latent Restoration Branch | {F_s} | I_SR (3, H_HR, W_HR) (학습 시만) | SR loss로 encoder 정규화, 추론 시 제거 | U-Net 업샘플링 + residual Swin block, PixelShuffle |
| ③ Saliency Prediction | {F_l} | {S_l}, (1, H_l, W_l) | 레벨별 전경 centrality 예측 | Local+Global branch → MLP, top-down propagation(α) |
| ④ Token Routing | encoder 토큰 (N, d) + saliency | 활성 (β_tγ_lN, d) + bypass (나머지, d) | 배경 토큰의 attention 계산 생략(정보는 유지) | top-k pre-attn + deformable self-attn + FFN / bypass는 pos.embed만 |
| ⑤ DINO Encoder/Decoder | 라우팅된 토큰 (N, d) | 클래스 + 박스 | 최종 detection 예측 | DINO 표준 encoder-decoder(변경 없음) |
| ⑥ 2-Stage Optimization | (파라미터 갱신 스케줄) | (파라미터 갱신 스케줄) | SR·detection 최적화 선호 충돌 완화 | Stage1: SR frozen / Stage2: SR lr×ρ 공동 최적화 |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table I (NWPU VHR-10-Split, 2× bicubic downsampling, Swin-T DINO baseline 대비)
**표를 보는 법**: 각 행이 하나의 detector, `AP`가 전체 성능, `AP_s`가 소형 객체(small) 성능이다 — CoLR-Det(Swin-T)를 같은 backbone의 DINO 행과 비교하면 이 논문이 얹은 이득만 볼 수 있다.

| 벤치마크 | 지표 | Before(Swin-T DINO) | After(CoLR-Det, Swin-T) |
|---|---|---|---|
| NWPU VHR-10-Split | AP / AP50 / AP_s | 0.766 / 0.986 / 0.487 | 0.836(+7.0%p) / 0.991 / 0.649(+16.2%p) |
| DOTAv1.5-Split | AP / AP_s | 0.402 / 0.192 | 0.419(+1.7%p) / 0.250(+5.8%p) |
| HRSSD-Split | AP / AP_s | 0.691 / 0.315 | 0.701(+1.0%p) / 0.371(+5.6%p) |

> [!note]- 세부 결과 및 Ablation
> #### Table I~III — SOTA 비교 (각 데이터셋 전체 detector 목록)
> **보는 법**: 각 표가 한 데이터셋을 다루고, 위 블록이 일반 object detector(Faster R-CNN, RetinaNet, Cascade R-CNN, YOLOX-s, Sparse R-CNN, SR4IR, D-FINE, DAB-DETR, DN-DETR, Deformable-DETR, DINO), 아래 블록이 원격탐사 특화 detector(EESRGAN, FSANet, SuperYOLO, FFCA-YOLO, LEGNet-T, MGAM, Strip-RCNN)다. CoLR-Det(ResNet50), CoLR-Det(Swin-T) 두 행이 항상 표 맨 아래에 굵게 강조되어 있다.
> - **Table I(NWPU VHR-10-Split)**: CoLR-Det(Swin-T) AP 0.836·AP_s 0.649로 전체 1위(2위는 DINO Swin-T AP 0.766, AP_s는 MGAM 0.349/Strip-RCNN 0.310보다도 크게 우위).
> - **Table II(DOTAv1.5-Split)**: CoLR-Det(Swin-T) AP 0.419(2위 DINO Swin-T 0.402), AP_s 0.250(2위 DINO Swin-T 0.192, FFCA-YOLO 0.221).
> - **Table III(HRSSD-Split)**: CoLR-Det(Swin-T) AP 0.701(2위 DINO Swin-T 0.691), AP_s 0.371(2위 DINO ResNet50 0.356).
> - 세 데이터셋 모두에서 CoLR-Det(ResNet50) 변형도 대부분 2위권에 든다 — 즉 이득이 특정 backbone에 국한되지 않음을 시사.
>
> #### Table IV — SR 브랜치 단독 효과(Cascade R-CNN·DINO, NWPU VHR-10-Split과 DOTAv1.5-Split)
> **보는 법**: "SR 브랜치" 열의 체크 유무로 같은 detector에 SR 브랜치만 추가했을 때의 순수 효과를 본다.
>
> | Dataset | Detector | SR 브랜치 | AP | AP_s |
> |---|---|---|---|---|
> | NWPU VHR-10-Split | Cascade R-CNN | 미포함 | 0.778 | 0.418 |
> | NWPU VHR-10-Split | Cascade R-CNN | 포함 | 0.799 | 0.452 |
> | NWPU VHR-10-Split | DINO | 미포함 | 0.766 | 0.487 |
> | NWPU VHR-10-Split | DINO | 포함 | 0.816 | 0.552 |
> | DOTAv1.5-Split | Cascade R-CNN | 미포함 | 0.357 | 0.142 |
> | DOTAv1.5-Split | Cascade R-CNN | 포함 | 0.362 | 0.183 |
> | DOTAv1.5-Split | DINO | 미포함 | 0.402 | 0.192 |
> | DOTAv1.5-Split | DINO | 포함 | 0.412 | 0.221 |
> - 두 detector 계열·두 데이터셋 모두에서 SR 브랜치가 일관되게 AP_s를 개선 — 아키텍처에 무관하게 latent 정규화 효과가 일반화됨을 시사.
>
> #### Table V — Detached/Infer 대조 실험(DINO 기준, NWPU VHR-10-Split)
> **보는 법**: "Encoder Grad 전파" 열이 SR gradient가 encoder로 흐르는지, "Infer 시 SR 실행" 열이 추론 시에도 SR head를 켜두는지를 나타낸다.
>
> | 구성 | Train SR | Encoder Grad 전파 | Infer 시 SR 실행 | AP | AP_s |
> |---|---|---|---|---|---|
> | DINO(baseline) | × | – | × | 0.766 | 0.487 |
> | DINO-SR(Detached) | ✓ | × | × | 0.770 | 0.491 |
> | **DINO-SR Branch(채택)** | ✓ | ✓ | × | **0.816** | **0.552** |
> | DINO-SR(Infer) | ✓ | ✓ | ✓ | 0.815 | 0.551 |
> - Detached(SR gradient가 encoder로 전파되지 않음)는 baseline과 거의 동일 — SR 브랜치 자체보다 "encoder를 함께 업데이트한다"는 것이 핵심 메커니즘임을 증명.
> - Infer(추론 시에도 SR head 실행)는 채택 방식과 성능이 거의 동일 — 추론 시 SR 브랜치가 실질적으로 불필요함을 재확인, "latent 정규화만으로 충분하다"는 핵심 주장의 근거.
>
> #### Table VI — Saliency 라우팅 vs SR 브랜치 상호작용(NWPU VHR-10-Split)
> **보는 법**: Saliency와 SR 브랜치를 독립적으로 켜고 꺼서 4가지 조합의 AP·AP_s를 비교한다.
>
> | Saliency | SR 브랜치 | AP | AP_s |
> |---|---|---|---|
> | - | - | 0.766 | 0.487 |
> | ✓ | - | 0.776 | 0.474(하락) |
> | - | ✓ | 0.816 | 0.552 |
> | ✓ | ✓(전체, CoLR-Det) | **0.836** | **0.649** |
> - Saliency 라우팅 단독은 AP_s를 오히려 낮춤(약한 객체 응답이 낮은 saliency 순위를 받을 수 있음) — 그러나 SR 브랜치와 결합하면 시너지(AP +2.0%p, AP_s +9.7%p) 발생.
>
> #### Table VII — 학습 스케줄 ablation(NWPU VHR-10-Split)
> **보는 법**: Single-stage(EW/UW) 두 baseline과, 2단계 전략에서 `T_det` 값(8/12/16/20/24)을 바꿔가며 AP·AP_s·AP_l을 비교한다.
>
> | Setting | AP | AP50 | AP75 | AP_s | AP_m | AP_l |
> |---|---|---|---|---|---|---|
> | Single-stage(EW) | 0.811 | 0.992 | 0.945 | 0.603 | 0.789 | 0.870 |
> | Single-stage(UW) | 0.800 | 0.991 | 0.949 | 0.586 | 0.780 | 0.858 |
> | T_det=8 | 0.815 | 0.991 | 0.947 | 0.593 | 0.791 | 0.862 |
> | T_det=12 | 0.830 | 0.992 | 0.959 | 0.621 | 0.810 | 0.878 |
> | **T_det=16(채택)** | **0.836** | 0.991 | 0.954 | **0.649** | **0.816** | 0.890 |
> | T_det=20 | 0.819 | 0.992 | 0.956 | 0.662 | 0.804 | 0.876 |
> | T_det=24 | 0.829 | 0.992 | 0.957 | 0.586 | 0.810 | 0.870 |
> - Single-stage 두 baseline은 AP50은 준수하지만 AP75·AP_s가 2단계 전략보다 낮음 — SR supervision을 처음부터 섞으면 detection-oriented semantic 형성이 방해받음을 시사.
> - `T_det`가 너무 짧으면(8) 개선폭이 제한, 너무 길면(24) 이후 SR-guided refinement 이득이 줄어 성능 하락 — `T_det=16`이 AP·AP_s·AP_m·AP_l 전반에서 최적 균형.
>
> #### Fig. 6 — 학습 가능한 modulation 계수 α의 효과
> **보는 법**: α를 제거(=1 고정)/고정값(0.5)/학습 가능 세 설정으로 비교한 두 그래프(전체 지표, 스케일별 지표).
> - AP: 제거 0.825 → 고정 0.831 → 학습 가능 0.836. 학습 가능한 α가 항상 최고 성능 — 상위 레벨 semantic을 얼마나 강하게 하위 레벨에 주입할지를 데이터에 맞춰 적응적으로 조절하는 것이 고정 계수보다 유리함을 보여줌.
>
> #### Fig. 7 — Backbone feature 응답 시각화(DINO vs DINO-SR Branch)
> **보는 법**: 같은 입력 이미지에 대해 DINO와 DINO-SR Branch의 3단계 backbone feature response를 heatmap으로 나란히 비교.
> - DINO baseline은 클러터가 많은 공항 주변 영역에서 응답이 흩어지는(diffuse) 반면, SR-증강 모델은 소형 항공기 등 실제 객체에 더 정밀하게 집중하고 배경 응답을 억제 — SR 브랜치가 학습 시 표현 정규화자로 작동한다는 정성적 근거.
>
> #### Fig. 8 — 효율성 분석
> **보는 법**: (a)는 AP-FLOPs trade-off 산점도, (b)는 SR4IR/CoLR-Det-Serial/CoLR-Det 파이프라인 비교 막대그래프, (c)(d)는 hierarchical query routing 유무·방식별 AP-FLOPs·정확도 비교.
> - 직렬 SR-then-detection 파이프라인(SR4IR AP 0.745, FLOPs 45.8G; CoLR-Det-Serial AP 0.766, FLOPs 88.22G) 대비 CoLR-Det은 FLOPs 16.94G로 대폭 절감하면서 AP는 오히려 0.836으로 개선 — 공유 latent 공간에서의 협업 최적화가 순차 처리보다 효과적이라는 결론의 핵심 근거. 본문은 CoLR-Det-Serial의 계산 중복(88.22G FLOPs)을 "이미지-레벨 SR 재구성을 detection 이전에 수행하기 때문"이라고 설명한다.
> - Hierarchical routing 전체 제거("No filtering") 시 FLOPs가 크게 증가하면서도 정확도(AP 0.824)는 오히려 joint filtering(0.836)보다 낮음 — 모든 토큰을 정제하는 것이 redundant 배경 토큰의 attention 희석 때문에 비효율적임을 시사. Joint 라우팅은 배경 지배적 쿼리의 약 60%를 bypass 경로로 보냄.
>
> #### Fig. 5 — 정성적 비교(NWPU VHR-10-Split/DOTAv1.5-Split/HRSSD-Split)
> **보는 법**: GT, Faster R-CNN, YOLOX-s, Deformable DETR, MGAM, DINO, CoLR-Det 순으로 같은 이미지의 탐지 결과를 나란히 배치 — CoLR-Det 열에서 밀집된 소형 객체(테니스장, 선박 등)의 누락·오탐이 다른 방법보다 적은지 눈으로 비교.

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- Saliency 예측이 학습 초기 불안정할 수 있음을 저자가 직접 인정 → <mark style="background: #FF5582A6;">"학습된 saliency는 초기 학습 단계에서 신뢰하기 어려울 수 있다"고 명시하며 이를 이유로 비파괴적(bypass) 라우팅을 설계했다고 밝히지만, 학습이 충분히 진행된 이후에도 saliency 예측 오류가 남아있는지에 대한 정량적 분석(saliency 예측 정확도 자체의 지표)은 제시되지 않는다.</mark>
- Detection-prioritized 2단계 전략에서 `T_det`가 너무 길면 SR-guided refinement의 이점이 감소 → <mark style="background: #FF5582A6;">Table VII에서 `T_det=16`이 최적이고 `T_det=24`에서는 AP·AP_s가 하락하는 것으로 확인되나, 이 민감도가 데이터셋마다 다를 수 있는지(NWPU VHR-10-Split에서만 검증됨)는 별도로 논의되지 않는다.</mark>

### 한계
- <mark style="background: #FF5582A6;">고정된 2배(2×) bicubic downsampling 배율에서만 검증됨 — 실제 저해상도 원격탐사 영상은 다양한 열화 정도(4×, 비균일 blur, 센서 노이즈 등)를 가질 수 있는데, 이런 조건에서의 일반화는 검증되지 않는다.</mark>
- <mark style="background: #FF5582A6;">저자가 Conclusion에서 직접 명시: 현재 방법이 pixel-level fidelity와 semantic-level invariance 간 간극을 "좁혔을 뿐" 완전히 해소하지 못했으며, 향후 더 정교한 양방향(bidirectional) guidance 메커니즘과 적응적 가중 스킴이 필요하다고 인정.</mark>
- Saliency confidence target(`C_l`)이 GT 박스 경계까지의 거리 기반이라, 밀집된 소형 객체가 서로 겹칠 때(경계가 모호할 때)의 confidence 계산은 "여러 후보 박스 중 최댓값을 사용한다"고만 규정되어 있어, 실제로 겹침이 심한 밀집 소형 객체군에서 이 처리가 얼마나 안정적인지는 별도 검증이 없다.
- Swin-T 계열 backbone 위주로 실험되어(CoLR-Det ResNet50 변형도 있으나 ablation 대부분이 Swin-T 기준), 다른 backbone(ConvNeXt 등)과 결합 시에도 동일한 이득이 재현되는지는 부분적으로만 확인됨.

### 생각할 점
- <mark style="background: #A6E3A1A6;">"복원이 탐지를 보조해야지 지배해서는 안 된다"는 이 논문의 원칙은, 이 위키의 [[ORFENet]]·[[FFSSTDNet]]·[[BAFNet]]이 공유하는 "학습 시에만 존재하는 auxiliary branch" 설계 철학과 정확히 같은 상위 원리를 공유한다 — 다만 이 논문은 그 원리를 이론적으로 가장 명시적으로 정식화(restoration-first vs latent-regularization 프레이밍)하고, saliency 라우팅이라는 추가 메커니즘으로 "복원 신호가 탐지를 지배하지 않도록" 능동적으로 차단한다는 점에서 한 단계 더 나아간다.</mark>
- <mark style="background: #A6E3A1A6;">Bypass 경로(제거가 아니라 attention만 건너뛰기)라는 설계는 이 위키의 [[QueryDet]]·[[YOFOR]]의 ALSM이 "배경을 아예 계산하지 않거나 크롭에서 제외"하는 것과 달리, "계산은 아끼되 정보는 잃지 않는다"는 절충안을 취한다 — 완전 제거(hard pruning) vs 비파괴적 우회(soft routing) 사이의 정확도-효율 트레이드오프를 세 방법 간 직접 비교하면 흥미로울 것으로 보인다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">Table VI의 "saliency 라우팅 단독으로는 AP_s가 오히려 하락하지만 SR 브랜치와 결합하면 크게 개선된다"는 결과는, 이 위키에서 반복 관찰되는 "단일 신호보다 다중 신호 결합이 크다"는 패턴의 특히 강한 사례다 — 두 모듈이 서로 상쇄가 아니라 시너지를 내는 조건(왜 SR 브랜치가 saliency 예측의 신뢰도 자체를 개선하는지)에 대한 메커니즘적 설명이 아직 없어, 후속 분석 가치가 있다.</mark>
- <mark style="background: #A6E3A1A6;">CoLR-Det의 saliency confidence(GT 박스 경계까지의 연속값 거리)는 이 위키의 [[ORFENet]] ORB(이진 마스크)보다 정보량이 많은 형태의 self-supervision 신호다 — "이진 vs 연속값 auxiliary target" 축에서 [[DQP-DETR]]의 RCS(margin ranking) 접근과 함께 비교하면, auxiliary supervision 신호의 표현력과 실제 성능 개선 간 관계를 체계적으로 정리할 수 있을 것으로 보인다.</mark>

> [!info] 내 메모
> 

# 관련 개념
- [[Latent_Restoration_Regularization]] — 이 논문의 핵심 기여. SR을 명시적 이미지 복원이 아니라 학습 시에만 encoder를 정규화하는 latent 공간 제약으로 재정의하는 프레임워크.
- [[Multi_Head_Self_Attention]] — DINO encoder의 self-attention, ④ token routing의 deformable self-attention 기반.
- [[1x1_Convolution]] — SR decoder의 이미지 재구성 conv, DINO encoder FFN의 기반 연산.

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]] — feature/latent 강화 계열에 새로 추가. Dynamic query DETR 계열(DQ-DETR 등)과 달리 DINO의 query 개수·구성이 아니라 encoder 표현 자체를 SR로 정규화한다는 점에서 다른 개입 지점. 다음 comparison 갱신 시 반영 예정.

# 읽어볼 만한 논문
- 참고문헌 기반: J. Wu, S. Xu, "From point to region: Accurate and efficient hierarchical small object detection in low-resolution images" (HSOD-Net, Remote Sens. 2021) [39] — 이 논문이 "selective 최적화"의 대표 선행 연구로 직접 비교·극복 대상으로 삼는 stepwise 파이프라인. CoLR-Det의 non-stepwise 설계와의 차이를 이해하는 데 필수.
- 참고문헌 기반: H. Zhang, F. Li, S. Liu 외, "DINO: DETR with improved denoising anchors for end-to-end object detection" (ICLR 2023) [50] — 이 논문의 detection branch가 그대로 기반하는 baseline 아키텍처. Saliency-guided token routing이 어디에 삽입되는지 이해하려면 필수.
- 참고문헌 기반: J. Kim, J. Oh, K. M. Lee, "Beyond image super-resolution for image recognition with task-driven perceptual loss" (SR4IR, CVPR 2024) [37] — Fig. 8에서 직접 비교하는 joint 최적화 선행 연구. Task-driven perceptual loss와 이 논문의 latent regularization 간 차이를 대조하며 읽으면 유용.
- 참고문헌 기반: Z. Liu, Y. Lin, Y. Cao 외, "Swin Transformer: Hierarchical vision transformer using shifted windows" (ICCV 2021) [40] — 이 논문의 공유 encoder가 그대로 채택하는 backbone 아키텍처. Window-based self-attention의 정확한 동작을 이해하려면 필수.
- 자유 추천(검증 필요): Vision Transformer의 token pruning/merging에서 "완전 제거"와 "bypass 경로 유지"를 비교하는 연구 — 검색 키워드: `vision transformer token pruning bypass non-destructive routing efficiency accuracy`. Saliency-guided routing의 비파괴적 설계가 일반적인 ViT 효율화 기법과 어떻게 다른지 배경 이해에 도움될 것으로 예상.
