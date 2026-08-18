---
title: "Breathing New Life into Small Object Detection with Detection-Oriented Rectification"
authors: [Xiang Yuan, Junwei Han, Gong Cheng]
year: 2026
venue: "IEEE TPAMI"
jcr_quartile: null
task: [small-object-detection]
direction: [improvement, novel-approach]
tags: [paper, small-object-detection, restoration, degradation-modeling, mixture-of-experts, multi-task-learning]
status: read
user_read: false
added: 2026-07-01
source: "raw/small-object-detection/2026_TPAMI_Detection-Oriented-Rectification.pdf"
created: 2026-08-04
---

# 한 줄 요약
<mark style="background: #FFF3A3A6;">Small object의 열화(degradation) 패턴을 학습 가능한 basis로 명시적으로 분해·학습한 뒤, 그 지식을 동적 prompt로 삼아 탐지 지향적(task-oriented)으로 feature를 rectify하는 DORA(Detection-Oriented RectificAtion) 프레임워크로, 다양한 detector에 plug-in 방식으로 결합해 SOD 성능을 일관되게 끌어올린다.</mark>


# 문제 정의

### 기존 방법의 한계
- **Feature space collapse**:
  잘 학습된 Faster R-CNN으로 COCO val GT 박스에서 추출한 region feature를 t-SNE로 시각화하면(Fig. 1), non-small instance는 클래스별로 뚜렷한 manifold를 형성하지만 small instance는 inter-class semantic entanglement(bird-kite, bottle-cup 등)와 intra-class distributional divergence라는 이중 붕괴를 보인다. GT로도 이렇다면 노이즈 섞인 실제 proposal 상황은 더 심각할 것.
- **Restoration 기반 방법의 degradation modeling 단절**:
  super-resolution, reconstruction, feature-imitation 세 갈래 모두 "복원"을 보조 과제로 쓰지만, 좁은 synthetic corruption에서만 학습돼 실제(compound) 열화 양상을 포착하지 못한다. 추론 시 명시적 시뮬레이션이 없어 학습-추론 간 distribution shift 발생.
- **Restoration과 detection 간 task conflict**:
  restoration은 pixel-level fidelity, detection은 semantic 전체 이해를 요구해 목표가 충돌한다. 기존 방법들은 이를 복잡한 stage-by-stage 학습으로 회피해 end-to-end 최적화의 우아함을 해친다.

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Super-Resolution(SR) 기반**
- Perceptual GAN [2], SOD-MTGAN [6], Finding Tiny Faces [3], Noh et al. [5], EFPN [1], SRD [19]: 사전학습 SR 모델을 signal magnifier로 붙임 — 별도 SR 네트워크의 구조적 복잡성, GAN 학습 특유의 불안정성.

**갈래 2 — Reconstruction 기반**
- SR-TOD [20](difference map), DirNet [15](degradation-independent representation), DFR-Det [10], UniRestore [13](diffusion prior), Liu et al. [21][22], Huang et al. [23], DRENet [24]: 병렬 reconstruction과 detection 공동 최적화 — pixel-space 제약에만 의존해 restoration-detection gap을 오히려 키울 위험.

**갈래 3 — Feature-imitation 기반**
- SML [18], Kim et al. [17], FMD [25], Yang et al. [26], InterNet [27], CFINet [8]: 열화된 representation이 고품질 representation을 모사 — dense pixel-level 정렬 강제로 공간적으로 뒤틀리기 쉬운 small object에서 잉여 노이즈에 overfitting(TABLE XI에서 L2/PKD [46]와 직접 비교로 검증).

**갈래 4 — High-level task-oriented restoration(일반 저수준 비전)**
- Liu et al. [22], URIE [33], IA-YOLO [14]·MAET [16], VRD-IR [35]·UniRestore [13], DIRNet [15]: recognition accuracy가 restoration을 이끌거나 열화 변환을 명시적으로 학습 — SOD에 특화되지 않았고 보조/주 과제 간 최적화 gap 미해소.

**갭**: <mark style="background: #FFF3A3A6;">네 갈래 모두 "복원"을 보조 과제로 쓰지만, 어느 쪽도 실제 small object의 복합적 열화 양상을 명시적으로 모델링하지 않고, restoration과 detection이라는 서로 다른 목표를 조화시키는 통합적 최적화 구조를 갖추지 못했다.</mark> 이 논문은 이 갭을 "열화를 먼저 명시적으로 이해한 뒤 그 지식으로 탐지 지향적 교정을 수행한다"는 degradation-then-rectification 패러다임으로 메운다.

### 이 논문이 풀고자 하는 문제
1. Small object에 내재된 실제 열화 패턴을 구조적으로 이해하고, 추론 시에도 이 지식을 활용해 distribution shift를 완화하는 방법
2. Restoration(pixel-level fidelity)과 detection(semantic discriminability) 간 task conflict를 근본적으로 해소하는 통합 최적화 구조
3. 특정 detector에 종속되지 않는 paradigm-agnostic 설계 (proposal-based/free, horizontal/oriented 전반)

# 제안 방법

<mark style="background: #FFF3A3A6;">핵심 아이디어: "무엇이 열화를 일으키는지 알아야, 어떻게 교정할지 안다"는 원칙 아래, 학습 가능한 degradation basis로 복잡한 corruption을 명시적으로 분해·학습(Degradation-aware Learning)한 뒤, 이 지식을 동적인 degradation-conditioned prompt로 변환해 detection 지향적 rectification(Task-oriented Rectification)을 수행하고, entity 단위 reconstruction과 self-correction으로 restoration-detection 간 task conflict를 완화한다.</mark> 전체 구조는 원본 이미지를 처리하는 primary branch와 합성 열화 이미지를 처리하는 auxiliary branch로 구성된 weight-sharing dual-branch다.

### ① Degradation Engine + Degradation Simulation (Degradation-aware Learning)
- Degradation Engine이 매 iteration마다 photometric distortion, 노이즈/블러, digital artifact 중 하나를 mild-to-moderate 강도로 확률적으로 샘플링해 원본 이미지의 열화 버전을 만든다.
- 학습 가능한 degradation basis `B_d`를 원본 feature에 modulation해 구조화된 열화 feature 라이브러리를 만든다 — 전역 문맥 기반 holistic 경로와 content-aware convolution 기반 spatially-variant 경로를 합쳐 최종 simulated feature 생성.
- 최적화는 pixel-wise 정합 대신 **prediction-level 정렬**(classification logit KL divergence + box IoU loss)을 핵심으로 삼는다.
- InfoNCE 기반 representation constraint와 basis 간 직교 정규화를 추가해 소수 패턴으로의 collapse를 방지한다.

> [!example]- 구현 디테일
> ```
> F_deg = F_o^sg ⊙ θ_1×1(B_d)                         (Eq. 1)
> F_s = s · F_deg + θ_1×1(F_o^sg) ⊛ W_ca              (Eq. 2)
> ```
> `s`: 전역 문맥에서 뽑은 affinity score (holistic 경로). `W_ca`: `B_d`를 변환한 content-aware convolution kernel (spatially-variant 경로).
>
> ```
> L_deg = Σ D_KL(σ(z_d), σ(z_s)) + Σ(1 − IoU(t_d, t_s))     [Prediction Alignment]
>       + λ1 · Σ InfoNCE(φ(F_s), φ(F_d))                      [Representation Constraint]
>       + λ2 · ||B_d·B_d − I||²_F                              [Regularization Term]
> ```
> TABLE VII: representation constraint·orthogonality regularization을 함께 켰을 때 30.0%→30.4%로 시너지, 어느 하나만 켜면 30.0~30.2%에 그침.

<mark style="background: #FFF9D6A6;">기존 방법들은 좁은 synthetic corruption 하나만 흉내 내거나 픽셀 단위로 정확히 재현하려 해 사소한 열화 매핑까지 학습하는 비효율에 빠졌다. 이 설계는 "task에 실제 영향을 주는 열화만" 선택적으로 학습하도록 prediction-level supervision을 핵심으로 삼아, basis가 실제 corruption 유형별로 구분되는 해석 가능한 primitive로 특화됨을 Fig. 9로 확인했다 — "무엇이 열화를 일으키는지"를 구조적으로 이해해 추론 시에도 재사용 가능해진다.</mark>

### ② Task-oriented Rectification (Basis → Prompt → Rectified Feature)
- degradation basis에 대한 spatially-varying affinity map으로 feature를 재문맥화한다.
- task-specific router(경량 네트워크, Top-ρ sparse gating)가 D개 expert 중 일부만 활성화해 task별 rectification prompt를 합성한다. Expert network는 branch 간 공유하되 prompt 생성은 scale·task별로 분리.
- prompt는 K개의 rectification block을 반복 통과하며 feature를 점진적으로 정제 — content-aware sub-sampling 후 prompt를 query, feature를 key/value로 하는 linear cross-attention과, prompt에서 유도한 per-pixel scale/shift modulation을 결합.

> [!example]- 구현 디테일
> Prompt 합성은 Eq. 5(Top-ρ sparse gating), rectification block 내부는 Eq. 7(linear cross-attention), Eq. 8(scale/shift modulation).
> 라우팅은 "얽힌 원본 feature"가 아니라 "열화 semantic으로 재문맥화된 feature"에 기반해 수행(TABLE IX: uniform routing 29.7% → degradation-aware routing 30.4%).

<mark style="background: #FFF9D6A6;">router가 현재 위치에서 지배적인 열화 유형을 먼저 파악한 뒤 그에 맞는 expert를 선택하므로, 추론 시 실제 입력의 corruption을 몰라도 학습된 degradation 지식을 조건으로 동적 prompt를 만들 수 있다 — "추론 시 명시적 corruption 시뮬레이션이 없어 distribution shift가 생긴다"는 근본 문제를 완화한다.</mark>

### ③ Entity Reconstruction + Self-correction (Task Conflict 완화)
- Image reconstruction만으로는 restoration-detection 간 granularity gap이 남아, query 기반 entity embedding을 도입한다.
- 각 embedding을 classification score(semantic identity)와 grounding map(spatial grounding)으로 투영하고, bipartite matching으로 GT 인스턴스와 짝짓는다.
- association loss로 기본 정렬을 학습하고, margin 기반 contrastive alignment loss로 detection-friendly exemplar(원본 branch에서 quality score가 가장 높은 positive prior의 feature)에 정렬시킨다.
- 추가로 self-correction term을 detection loss에 넣어, rectified prediction이 원본 prediction보다 항상 더 좋아지도록 강제한다.

> [!example]- 구현 디테일
> ```
> L_rec = L_ent_rec + λ_img · L_img_rec     (Eq. 14, λ_img=0.5)
> L_det = λ3/|S_pos| · Σ [J(p_sg, p̃) + J(IoU_sg, IoU_tilde)] + L_base(P_o) + L_base(P̃_o)   (Eq. 17)
> L_dora = L_det + λ_deg · L_deg + λ_rec · L_rec     (Eq. 19)
> ```
> Entity projection은 Eq. 10~12, association/contrastive loss는 Eq. 13(FL+QFL 항 + margin contrastive 항), self-correction은 Eq. 17~18.
>
> 주요 기본값: `D=8`(basis 수), `ρ=4`(sparse gating top-k), `K=3`(rectification block 수), `λ1=λ2=0.1`, `λ_img=0.5`, `λ3=0.5`, `α=0.6`·`ε=0.15`(entity matching), `T=0.10`(자연 영상)/`0.08`(항공 영상, 추론 시 sparse inference threshold).

<mark style="background: #FFF9D6A6;">pixel-space 제약만 쓰는 기존 방법(SR-TOD 등)은 detection과 무관한 디테일까지 복원하려 해 목표가 충돌한다. Entity reconstruction은 "인스턴스의 semantic identity와 spatial grounding"이라는 detection이 실제로 필요로 하는 정보 단위로 supervision을 재구성해 granularity gap 자체를 줄인다(TABLE X: entity-only가 image-only보다 우수, 합치면 최적; TABLE XIX: 단순 detection sub-network 부착이나 binary classification bridging은 baseline 대비 개선 없어 entity reconstruction 설계가 필수임을 확인). Self-correction term은 이 개선이 실제로 detection 방향으로만 작동하도록 강제하는 안전장치다(TABLE XII: 없으면 30.4%→30.5%로 개선 미미하거나 COCO에서 소폭 하락).</mark>

# 실험 결과

- **벤치마크**: SODA-D, AITOD-R·SODA-A(oriented), COCO val, VisDrone val (총 5개). 백본 ResNet-50, 2×RTX 3090.

### 핵심 결과 — SODA-D test (TABLE I)

| Detector | AP (Before→After) | 비고 |
|---|---|---|
| Cascade R-CNN [49] | 31.2→32.8 (+1.6) | 전체 SOTA |
| Faster R-CNN [11] | 28.9→30.8 (+1.9) | SR-TOD는 동일 baseline에서 +0.4에 그침 |

> [!note]- 세부 결과 및 Ablation
> #### 설정
> 지표: AP, AP50/75, 크기별 세분화 AP(SODA-D/A: APeS/APrS/APgS/APN, AITOD-R: APvt/APt/APs/APm), FLOPs, FPS(RTX 3090 단일 GPU). SODA-D/SODA-A/AITOD-R은 1× 스케줄(12 epoch), COCO 24 epoch, VisDrone 15 epoch.
>
> #### SODA-D test 전체 (TABLE I 발췌)
> | Detector | Paradigm | AP | AP50 | APeS | 비고 |
> |---|---|---|---|---|---|
> | Faster R-CNN [11] | proposal-based | 28.9→30.8 (+1.9) | 59.4→61.1 | 13.8→15.1 | SR-TOD는 +0.4 |
> | Cascade R-CNN [49] | proposal-based | 31.2→32.8 (+1.6) | 59.9→60.4 | 14.1→15.6 | 전체 SOTA |
> | DoubleHead [50] | proposal-based | 31.3→32.7 (+1.4) | 61.3→63.0 | 15.0→16.8 | |
> | CFINet [8] | proposal-based | 30.7→32.0 (+1.3) | 60.8→62.4 | 14.7→15.9 | |
> | RFLA [9] | proposal-based | 29.7→31.3 (+1.6) | 60.2→61.7 | 13.2→14.9 | |
> | FCOS [54] | proposal-free | 23.9→26.4 (+2.5) | 49.5→54.3 | 6.9→9.0 | 개선폭 최대. SR-TOD는 −0.4 |
> | GFL [65] | proposal-free | 29.0→30.7 (+1.7) | 57.3→59.2 | 12.8→14.1 | |
> | TOOD [56] | proposal-free | 30.5→31.9 (+1.4) | 58.0→59.9 | 12.2→13.5 | |
> | RetinaNet [53] | proposal-free | 28.2→29.7 (+1.5) | 57.6→59.7 | 11.9→13.4 | SR-TOD는 +0.1 |
>
> DiffusionDet [41] 등 diffusion 기반 detector는 SOD에서 정확도(21.4% AP)·효율(FPS 16.9) 모두 열세 — 반복적 denoising이 small object의 미세한 신호를 랜덤 노이즈에 묻히게 하는 것으로 추정.
>
> #### COCO val / VisDrone val / OSOD(SODA-A, AITOD-R)
> | 벤치마크 | Detector | AP | APS(또는 APeS/APvt) | 비고 |
> |---|---|---|---|---|
> | COCO val | TOOD | 42.3→44.3 (+2.0) | 24.7→26.9 | 대부분 detector에서 APL 유지/소폭 상승 |
> | COCO val | Faster R-CNN | 38.2→39.8 (+1.6) | 21.3→23.3 | |
> | COCO val | RetinaNet | 37.2→38.4 (+1.2) | 19.8→22.6 | |
> | VisDrone val | Faster R-CNN | 26.3→28.1 (+1.8) | 17.4→19.7(APS) | SR-TOD는 +0.3 |
> | VisDrone val | GFL | 27.2→28.7 (+1.5) | 17.9→19.7(APS) | |
> | SODA-A test | DCFL [4] | 36.6→37.9 (+1.3) | 13.9→15.0(APeS) | 경쟁 기법 중 최고 |
> | SODA-A test | Strip R-CNN [95] | 36.7→38.1 (+1.4) | 13.6→15.5(APeS) | |
> | AITOD-R test | Rotated FCOS | 12.4→14.2 (+1.8) | 4.1→4.5(APvt) | 매우 작은 객체에서도 일관된 개선 |
> | AITOD-R test | GRA [90] | 12.9→14.8 (+1.9) | 3.4→5.0(APvt) | |
>
> #### Ablation (TABLE VI, Faster R-CNN, SODA-D/COCO)
> | 구성 | SODA-D AP | COCO AP |
> |---|---|---|
> | Baseline | 28.9 | 38.2 |
> | Dual-branch만(모든 모듈 off) | 29.0 | 38.2 |
> | + Task-oriented Rectification(D-L 없이) | 29.4 | 38.9 |
> | + Degradation-aware Learning(D-L) | 30.4 | 39.6 |
> | + Detection-centric Reconstruction(D-R, 최종) | 30.8 | 39.8 |
>
> #### 세부 발견
> - Degradation-aware Learning 단독(TABLE VII): task-level supervision만으로 29.4%→30.0%. Rep.+Reg. 둘 다(30.4%)가 하나만(30.0~30.2%)보다 우수 — representation homogenization/pattern collapse가 실재하는 위험.
> - Task-oriented Rectification 구성(TABLE VIII): degradation-conditioning 단독 30.2%, modulation 단독 30.1%, 둘 다 30.4% — 상호 보완적.
> - Routing 메커니즘(TABLE IX): Uniform 29.7% < Random 30.0% < Degradation-aware(제안) 30.4% — sparsity만으론 불충분, semantic guidance가 핵심.
> - Reconstruction 설계(TABLE X, XI): Entity-only 30.5% > Image-only 30.3%, 둘 다 30.8%(λ_img=1.0으로 과하게 높이면 30.7%로 소폭 하락). Feature distillation 대안(L2 30.1%, PKD [46] 30.4%)은 entity reconstruction(30.8%)보다 열세.
> - Bridging task 선택(TABLE XIX): 단순 detection sub-network 부착(30.4%)이나 binary classification(30.5%)은 baseline과 큰 차이 없음 — entity reconstruction(30.8%)만 유의미한 개선.
> - Self-correction term(TABLE XII): 없으면 30.5%(SODA-D)/39.5%(COCO)로 오히려 소폭 하락하는 경우도 있음.
> - Basis 수(D)·sparse gating(ρ)(TABLE XIV): D=4→8에서 29.9%→30.4%로 크게 개선되지만 D=16은 +0.1%에 그침. ρ를 4→16으로 늘리면(D=16 기준) 오히려 30.5%→30.1%로 하락 — "다양한 basis, sparse 선택" 가설 뒷받침.
> - Rectification block 수(K)(TABLE XV): K=1(30.6%)→K=3(30.8%, APeS 15.1% 최고)→K=5(31.0%, FPS 18.2→16.5로 저하) — K=3을 정확도-효율 균형점으로 채택.
> - Sub-sampling ratio(TABLE XVI): 2× 다운샘플이 1× 대비 AReS 25.8%→25.6%로 거의 손실 없이 FLOPs 37.9G→28.4G 절감.
> - Query 수 N(TABLE XVIII): sparse한 SODA-D/COCO는 N=100으로 충분(포화). 밀집한 SODA-A/AITOD-R은 N=300이 AR을 유의미하게 개선(+0.6, +1.4) — 밀집 장면에서 recall 병목.
> - 다른 detector(Cascade R-CNN, GFL, DoubleHead 등)에도 이식해 일관된 개선(+0.8~+2.5%p) — architecture-agnostic 설계 검증.

# Discussion

### 이 아이디어의 잠재적 부작용
- **Degradation basis의 학습 데이터 corruption 분포 과적합 위험**:
  Degradation Engine의 corruption family(밝기/대비, 노이즈/블러, jpeg/pixelate)는 curated suite다. <mark style="background: #FF5582A6;">목록 밖 열화(극단적 저조도, 대기 산란, 센서 특이적 노이즈)에 대한 stress test가 논문에 없어 일반화 정도는 미검증.</mark>
- **큰 객체 성능 하락 가능성**:
  rectification이 small object 최적화 설계라 큰 객체 feature를 왜곡할 위험. <mark style="background: #FF5582A6;">TABLE II·III에서 일부 조합(COCO RetinaNet, DoubleHead, Cascade R-CNN)은 APL이 0.0~1.0%p 하락 — 원인 분석은 논문에 없음.</mark>
- **N(entity query 수) 고정에 따른 밀집 장면 recall 병목**:
  TABLE XVIII에서 N을 늘리면 밀집 장면 AR이 개선됨은 역으로 N 부족 시 일부 인스턴스가 uncovered로 남는다는 뜻. <mark style="background: #FF5582A6;">본문도 이를 언급하지만(Fig. 10) 정량적 실패율은 미보고.</mark>

### 한계
- <mark style="background: #FF5582A6;">저자가 명시: degradation basis는 특정 데이터셋 내 학습 구조이며, 대규모 외부 데이터로 범용 task-friendly feature manifold를 학습해 더 근본적인 supervisory target을 확보하는 것은 미해결 과제.</mark>
- <mark style="background: #FF5582A6;">K를 늘리면 정확도는 계속 개선(K=5, SODA-D AP 31.0%)되지만 FPS가 18.2→16.5로 저하 — 정확도-효율 트레이드오프 존재.</mark> 논문은 K=3을 기본값으로 타협.
- Diffusion detector 대비 우위 분석은 DiffusionDet 한 사례에만 근거 — RediffDet [43], DiffuYOLO [44]와의 직접 비교는 없음.
- Entity reconstruction의 bipartite matching·contrastive alignment는 하이퍼파라미터(α, ε, N)에 민감(TABLE XVII, XVIII) — 새 데이터셋 적용 시 재튜닝 필요 가능성.

### 생각할 점
- <mark style="background: #A6E3A1A6;">"열화를 먼저 명시적으로 학습한 뒤 그 지식을 조건으로 교정한다"는 원칙은 SOD에 국한되지 않아 보인다 — 저조도 인식, 안개/우천 인식처럼 입력이 체계적으로 열화된 다른 태스크에도 이식 가능할 것 같다.</mark>
- Entity reconstruction이 pixel reconstruction보다 우수하다는 결과는 "복원 목표 단위를 detection이 실제 쓰는 단위(인스턴스)로 맞추는 것"이 pixel fidelity보다 중요하다는 교훈으로 읽힌다 — [[Unc-SOD]]의 uncertainty 기반 sampling과는 다른 축이지만 "detection이 필요로 하는 신호를 먼저 정의하고 보조 과제를 설계"한다는 점에서 상통.
- Degradation basis를 도메인 특화(항공/위성의 atmospheric haze, sensor noise)로 확장하면 basis가 도메인별로 더 해석 가능해질 수 있을 것 같다.

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">entity reconstruction을 [[Unc-SOD]]의 instance-level uncertainty와 결합할 여지 — uncertainty가 높은 인스턴스일수록 rectification을 더 강하게(K를 더 깊게) 적용하는 adaptive 확장이 가능할 것 같다. 두 논문 모두 K, T를 고정값으로 쓰는데, instance-level 신호로 동적 조절 시 추가 이득이 있을지 검증할 만하다.</mark>
- [[SR-TOD]]는 difference map(원본-재구성 차이)으로 small object를 강조하는 반면, 이 논문은 "왜 그 차이가 생겼는지"(degradation 유형)를 basis로 명시화한다 — 두 접근을 결합해 difference map을 degradation basis 활성화 패턴으로 재해석하는 방향도 흥미로운 후속 실험.
- Degradation basis의 domain-specific 확장 가능성은 [[FANet]], [[RS-TOD]], [[UAV-DETR]] 같은 원격탐사 특화 논문들과 비교하며 검증할 가치가 있다.

# 관련 개념
- [[Degradation_Aware_Rectification]] — 이 논문이 제안하는 핵심 기법. 열화를 학습 가능한 basis로 명시적으로 모델링하고, 그 지식을 조건으로 한 동적 프롬프트로 task-oriented rectification을 수행하는 degradation-then-rectification 패러다임.

# 관련 문서
- 같은 저자 그룹(Xiang Yuan, Gong Cheng, Junwei Han)의 관련 논문: [[Unc-SOD]] — RPN sampling과 feature hierarchy 불일치를 다루는 다른 각도의 SOD 개선 연구. 두 논문 모두 "인스턴스 단위 신호를 어떻게 학습 신호로 쓸 것인가"를 다룬다는 점에서 상통.
- 직접 비교 대상(TABLE I~III에서 baseline 병기): [[SR-TOD]] — 동일한 restoration/reconstruction 계열이지만 difference map 기반이라는 점에서 이 논문(degradation basis 기반)과 접근이 다르며, 대부분의 벤치마크·detector 조합에서 DORA가 더 큰 개선폭을 보임.
- 비교: [[Small_Object_Detection_Approaches]] — feature 복원 축으로 분류되어 있으며, 이 문서의 비교표에 이미 이 논문 항목이 반영되어 있음.

# 읽어볼 만한 논문
- 참고문헌 기반: B. Cao, H. Yao, P. Zhu, Q. Hu, "Visible and clear: Finding tiny objects in difference map" (SR-TOD) [20] (ECCV 2024) — 이 논문이 TABLE I~III에서 직접 비교하는 핵심 baseline. 같은 reconstruction 계열이지만 difference map 기반 접근이라 DORA의 degradation basis 접근과 대조하며 읽기 좋음. 위키에 이미 [[SR-TOD]] 노트 있음.
- 참고문헌 기반: I. Chen, W.-T. Chen et al., "UniRestore: Unified perceptual and task-oriented image restoration model using diffusion prior" [13] (CVPR 2025) — perceptual quality와 downstream task를 함께 최적화하는 최신 통합 restoration 프레임워크. DORA가 "high-level task-oriented restoration" 갈래의 최신 대표작으로 인용하며 여전히 SOD 특화 최적화 gap이 남아있다고 지적한 대상이라, 이 논문의 구체적 통합 방식을 비교해볼 가치가 있음.
- 참고문헌 기반: T. Son, J. Kang, N. Kim, S. Cho, S. Kwak, "URIE: Universal image enhancement for visual recognition in the wild" [33] (ECCV 2020) — "recognition accuracy가 low-level restoration의 최적화를 이끈다"는 원칙의 초기 대표 사례. DORA의 task-oriented rectification 설계 철학의 계보를 이해하는 데 배경지식으로 유용.
- 참고문헌 기반: C. Xu, J. Ding et al., "Dynamic coarse-to-fine learning for oriented tiny object detection" (DCFL) [4] (CVPR 2023) — SODA-A에서 DORA와 결합했을 때 가장 좋은 결과(37.9% AP)를 낸 OSOD 특화 detector. Oriented SOD 자체의 대표 접근을 이해하는 데 필요.
- 자유 추천(검증 필요): 저조도/악천후 등 다른 "체계적으로 열화된 입력" 도메인에서 degradation을 명시적으로 모델링해 downstream task를 돕는 최신 연구 — 검색 키워드: `explicit degradation modeling low-light object detection task-oriented restoration 2025`. 위 Discussion의 "다른 저품질 비전 태스크로 이식 가능한가" 질문을 실제로 검증하려면 참고할 만한 방향.
