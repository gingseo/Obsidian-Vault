---
title: "Vision-Language Guided Semantic Diffusion Sampling for Small Object Detection in Remote Sensing Imagery"
authors: [Jian Ma, Mingming Bian, Fan Fan, Hui Kuang, Lei Liu, Zhibing Wang, Ting Li, Running Zhang]
year: 2025
venue: "Remote Sensing (MDPI)"
jcr_quartile: Q2
task: [small-object-detection]
direction: [novel-approach]
tags: [paper, small-object-detection, sar, remote-sensing, vision-language-model, clip, diffusion-model, label-assignment, bounding-box-regression]
status: read
user_read: false
added: 2026-08-05
source: "raw/small-object-detection/2025_RemoteSensing_CDATOD-Diff.pdf"
created: 2026-08-05
---

# 한 줄 요약
<mark style="background: #FFF3A3A6;">CLIP의 이미지-텍스트 의미 정보를 조건으로 diffusion denoising 과정을 통해 GT 박스 주변 anchor 샘플링 포인트를 생성함으로써 소형 객체의 양성 샘플 부족을 완화하고, corner distance와 IoU를 객체 크기에 따라 적응적으로 가중합하는 BC-IoU loss로 회귀 불안정성을 줄이는 CDATOD-Diff 프레임워크.</mark>


# 문제 정의

### 기존 방법의 한계
- **정적 샘플링 패러다임과 소형 객체의 근본적 불일치**:
  소형 객체는 크기가 극도로 작아 고정된 grid 형태의 균일 샘플링 포인트와 유효한 대응(correspondence)을 형성하지 못하거나, 형성하더라도 아주 미미한 매칭에 그친다(Fig. 1).
- **양성-음성 샘플 불균형**:
  기존 anchor 샘플링 과정이 맥락적 prior를 충분히 반영하지 않아 학습 단계에서 양성-음성 샘플 불균형이 심화되고, 이는 네트워크 최적화 효율을 크게 저해한다.
- **회귀 손실 함수의 스케일 민감도**:
  IoU 기반 손실은 소형 객체의 미세한 위치 오차에 극도로 민감(overlap이 쉽게 0에 가까워짐)한 반면, 중심점 거리 손실은 예측 중심과 GT 중심이 일치하면 박스 크기와 무관하게 0이 되어 버려, 소형 객체의 크기 회귀에 대한 supervision이 사라지는 문제가 있다.

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Label assignment/샘플링 개선**
- S3FD [34]: 낮은 초기 IoU 임계값으로 2단계 매칭 + 미매칭 GT에 대한 Top-N 랭킹.
- Zhang et al. [35](ATSS): 타겟 통계 특성에 기반해 할당 임계값을 동적으로 조정.
- Zhu et al. [36](AutoAssign): 이진 분류 대신 dual-weight 할당으로 양성/음성 기여도를 균형.
- Xu et al. [37](RFLA): Gaussian receptive field 기반 매칭 — 거리-점수 순위 top-K 후보 1차 선정 + decayed field radius로 2차 정제. 이 논문이 직접 확장하는 가장 가까운 선행 연구.
- 공통 한계: 샘플링 밀도나 고정된 할당 기준(fixed assignment criteria) 개선에 집중해, "의미적으로 무엇이 실제 객체인지"에 대한 사전 지식은 반영하지 못함.

**갈래 2 — 확산 모델 기반 탐지**
- DiffDet4SAR [23]: 항공기 타겟 탐지를 바운딩 박스 디노이징 과정으로 모델링하고 산란(scattering) feature 강화 모듈로 clutter 억제 — bounding box 자체를 diffusion target으로 삼음.
- 갭: DiffDet4SAR와 달리 이 논문은 Gaussian 생성적 샘플링 과정과 적응형 BC-IoU loss를 결합해, 명시적 anchor 분포 모델링과 소형 객체에 대한 견고성을 함께 다룬다는 점에서 다르다.

**갈래 3 — 비전-언어 모델의 원격탐사 응용**
- Qiu et al. [41], Basso [42], Bazi et al. [43]: CLIP을 feature extraction·검색·VQA 등에 활용 — 탐지의 anchor 샘플링 과정 자체에 CLIP을 결합한 사례는 없음.

**갭**: <mark style="background: #FFF3A3A6;">Label assignment 계열은 기하학적/통계적 기준(거리, IoU, Gaussian receptive field)만으로 후보를 정제할 뿐 "이 위치가 실제로 어떤 의미를 갖는 객체인가"라는 의미적 정보를 활용하지 않고, VLM 계열은 탐지의 anchor 샘플링 프로세스 자체에는 개입하지 않는다. CLIP의 크로스모달 의미 정보를 diffusion 기반 생성적 샘플링의 조건(condition)으로 직접 결합해 "의미적으로 타당한 위치에 편향된" 샘플을 생성하는 접근은 없었다.</mark>

### 이 논문이 풀고자 하는 문제
1. 소형 객체의 양성 샘플 부족 문제를 CLIP의 의미적 prior로 완화하는 것
2. Gaussian 생성 + diffusion denoising으로 anchor 샘플링 포인트 자체를 반복적으로 정제하는 것
3. 객체 크기에 따라 IoU와 corner distance의 기여도를 동적으로 조정해 회귀 불안정성을 줄이는 것

# 제안 방법

<mark style="background: #FFF3A3A6;">GT 박스를 2D Gaussian 분포로 모델링해 그 분포에서 샘플링 포인트를 생성하고, 이 포인트들을 CLIP의 이미지-텍스트 의미 임베딩을 조건으로 한 diffusion denoising 과정으로 반복 정제함으로써, 순수 기하학적 기준보다 의미적으로 더 신뢰할 수 있는 양성 샘플을 생성한다. 여기에 스케일에 따라 corner distance와 IoU의 비중을 적응적으로 바꾸는 BC-IoU loss를 결합해 회귀 안정성을 높인다.</mark>

### ① CLIP-Driven Dynamic Anchor Point Sampling
- RFLA [37]의 Gaussian receptive field 매칭을 계층적으로 확장: 1차로 Wasserstein distance 기반 점수(WDS)로 후보를 선정(`R1`), 유효 receptive field 반경을 줄여 재랭킹한 2차 결과(`R2`)를 결합(`R = R1·m + R2·(1-m)`)해 GT당 양성 샘플 수를 늘림.
- Deformable convolution 기반 offset prediction 모듈이 샘플링 위치에 학습 가능한 변위(∆p)를 추가로 적용, 두 해상도가 가장 높은 feature map에서 동작.

> [!example]- 구현 디테일
> ```
> WDS = 1 / (1 + ||[x_n,y_n,e^{r_n},e^{r_n}]^T - [x_g,y_g,w_g/2,h_g/2]^T||_2^2)
> (x', y') = (x + s·dx/2, y + s·dy/2)   # s=stride, (dx,dy)=학습된 offset
> ```

<mark style="background: #FFF9D6A6;">Gaussian receptive field 기반 거리 점수만으로는 여전히 기하학적 유사도만 반영하므로, "문제 정의"에서 언급한 정적 샘플링-소형 객체 불일치 문제를 완전히 해소하지 못한다. Deformable offset으로 샘플링 위치 자체를 학습 가능하게 만들어, 고정 grid의 한계를 넘어서는 유연성을 확보한다.</mark>

### ② Diffusion-Based Anchor Point Sampling + CLIP 조건화
- GT 박스 면적에 비례하는 개수의 샘플 포인트를 Gaussian sampler로 무작위 생성 후, DDPM forward process로 노이즈를 주입(`x_t = √α_t·x_0 + ε√(1-α_t)`).
- DDIM으로 가속화된 역방향(디노이징) 과정을 거쳐 노이즈로부터 정제된 샘플링 위치를 복원.
- CLIP 텍스트 인코더가 "an image of [CLASS]" 프롬프트를 인코딩하고, ViT-B/16 이미지 인코더가 12개 레이어의 계층적 feature를 추출 — 이 두 임베딩을 modality-specific mapping으로 정렬한 뒤 diffusion U-Net의 각 단계에 조건(condition)으로 residual 방식으로 주입.
- 디노이징 후 위치가 원래 feature map의 격자와 어긋나므로, spatial calibration 모듈이 위치 편차를 측정해 deformable convolution으로 feature map을 보정.

> [!example]- 구현 디테일
> ```
> σ² = η · (1-α_p)/(1-α_t) · (1-α_t/α_p)
> x_p = √α_p·((x_t - √(1-α_t)·ε_t)/√α_t) + √(1-α_p-σ²)·ε_t + σε
> F_fusion_clip = σ(W_text ⊙ f_ta) ⊕ (W_image ⊙ f_is)   # 텍스트-이미지 융합
> F_CD = Conv1x1(ReLU([F_trans_clip ‖ F_diff]))          # CLIP 조건 결합
> ```
> 텍스트 임베딩은 768차원, ViT-B/16은 16×16 패치·12 레이어. 학습 시 diffusion timestep 1000단계 중 3회의 순차적 샘플링 반복.

<mark style="background: #FFF9D6A6;">CLIP의 크로스모달 의미 임베딩을 diffusion 조건으로 주입함으로써, 순수 기하학적 거리(①의 WDS)만으로는 구분할 수 없는 "의미적으로 타당한 객체 위치"를 반영한 샘플을 생성한다 — 이는 "문제 정의"의 양성 샘플 불균형을 기하학적 정제와 의미적 정제 두 층위에서 동시에 완화하는 설계다.</mark>

### ③ Balanced Corner-IoU (BC-IoU) Loss
- 중심점 거리 대신 예측 박스와 GT 박스의 두 모서리(좌상단, 우하단) 좌표 거리 합으로 `L_Corner = 1 - e^{-D_corn/S}` 정의 — 중심이 일치해도 크기 오차가 남아있으면 loss가 0이 되지 않도록 함.
- 객체 면적 `A`에 따라 지수적으로 감소하는 가중치 `w = e^{-A/β}`로 corner loss와 IoU loss를 혼합: 소형 객체일수록 corner loss 비중이 커지고, 큰 객체일수록 IoU loss 비중이 커짐.

> [!example]- 구현 디테일
> ```
> L_Corner = 1 - e^{-D/S}         # S=4
> L_BC-IoU = w·L_Corner + (1-w)·L_IoU,  w = e^{-A/β}   # β=12
> ```
> Figure 5 분석: IoU loss는 소형 객체 위치 이동에 극도로 민감하고 대형 객체에는 완만한 반면, corner loss는 스케일에 걸쳐 안정적 — 이 상반된 성질을 가중합으로 절충.

<mark style="background: #FFF9D6A6;">중심점 거리 손실이 중심 일치 시 크기 회귀 supervision을 잃는 문제와, IoU 손실이 소형 객체에서 과민 반응하는 문제를 각각 corner 기반 재정의와 스케일 적응 가중치로 해결해, "문제 정의"의 회귀 불안정성 문제를 스케일 전 구간에서 완화한다.</mark>

# 실험 결과

### 핵심 결과
| 벤치마크 | 지표 | Before(3SD-Net, 이전 SOTA) | After(CDATOD-Diff) |
|---|---|---|---|
| MSAR-1.0 | AP / APs | 63.4 / 51.6 | 64.1 / 58.9 |
| AI-TOD | AP / APt | 16.3(RFLA) / 18.5(RFLA) | 19.4 / 20.8 |

> [!note]- 세부 결과 및 Ablation
> #### SOTA 비교 (HRSID, VEDAI)
> | 데이터셋 | 지표 | 이전 최고(RFLA/3SD-Net) | CDATOD-Diff |
> |---|---|---|---|
> | HRSID | AP / APs | 51.9 / 36.9 | 55.4 / 39.7 |
> | VEDAI | mAP | 0.340(DiffusionDet) | 0.365 |
>
> #### 모듈별 Ablation (AI-TOD/USOD/VEDAI, FCOS 기준)
> | 구성 | AI-TOD AP | USOD AP | VEDAI mAP |
> |---|---|---|---|
> | FCOS(baseline) | 0.107 | 0.106 | 0.017 |
> | FCOS+RFLA | 0.163 | 0.192 | 0.324 |
> | FCOS+Diffusion(CLIP 미적용) | 0.179 | 0.232 | 0.313 |
> | FCOS+Diff-CLIP(전체) | 0.194 | 0.246 | 0.365 |
> - AI-TOD의 극소형(APvt)·타이니(APt) 객체에서 CLIP 조건화가 diffusion 단독 대비 각각 +2.1, +4.1 추가 향상.
>
> #### Loss ablation (AI-TOD)
> | 구성 | AP | AP50 | AP75 |
> |---|---|---|---|
> | IoU only | 10.6 | 26.8 | 6.3 |
> | Corner only | 11.2 | 27.7 | 6.6 |
> | BC-IoU(결합) | 12.5 | 30.5 | 8.0 |
> - GIoU/DIoU/CIoU/EIoU/SIoU 등 다른 IoU 계열 손실과 비교해도 BC-IoU가 AP 12.5로 최고(다음은 EIoU 11.4).
>
> #### 정성 결과
> - Grad-CAM 시각화에서 실제 소형 객체에 더 높은 attention 가중치 부여, 오탐/누락 감소(Fig. 9).
> - HRSID(SAR)에서 FCOS는 강한 배경 산란체 주변에 오탐, RFLA는 실제 선박을 놓치는 반면, 제안 방법은 배경 간섭을 억제하면서 더 완전한 탐지 달성.

# Discussion

### 이 아이디어의 잠재적 부작용
- Diffusion 기반 샘플링은 1000 timestep 중 다단계 반복 디노이징을 거치므로 추론/학습 연산 비용이 커질 가능성 → <mark style="background: #FF5582A6;">논문은 DDIM으로 가속했다고 언급할 뿐, 실제 FPS·추론 시간 비교는 제시하지 않는다.</mark>
- CLIP 텍스트 프롬프트가 고정된 "an image of [CLASS]" 템플릿에 의존 → <mark style="background: #FF5582A6;">SAR 영상처럼 시각 도메인이 CLIP의 원 학습 분포(자연 이미지)와 크게 다른 경우 프롬프트-이미지 정렬 품질이 보장되는지에 대한 검증이 없다.</mark>

### 한계
- <mark style="background: #FF5582A6;">추론 속도(FPS)나 파라미터 수 등 연산 비용 지표가 논문 전체에 전혀 보고되지 않아, diffusion+CLIP 결합이 실제 배포 환경에서 어느 정도 비용을 요구하는지 알 수 없다.</mark>
- <mark style="background: #FF5582A6;">VEDAI에서 OT(오일탱크)·TK(트럭) 클래스는 다른 방법 대비 성능이 낮음(OT 0.162로 최저권) — 저자가 별도로 원인을 분석하지 않았다.</mark>

### 생각할 점
- <mark style="background: #A6E3A1A6;">RFLA의 Gaussian receptive field 매칭을 명시적으로 확장한다는 점에서, [[Gaussian_Box_Uncertainty_Modeling]]/[[Position_Gaussian_Saliency_Map]]과 마찬가지로 "Gaussian으로 무언가를 모델링"하는 이 위키의 세 번째 사례 — 다만 이 논문은 박스 좌표나 saliency가 아니라 "샘플링 포인트 자체의 분포"를 모델링한다는 점에서 또 다른 대상을 다룬다.</mark>
- <mark style="background: #A6E3A1A6;">CLIP을 label assignment/샘플링 단계에 결합한 시도는 이 위키에서 처음 등장 — 기존 feature 강화 계열([[SR-TOD]], [[FANet]] 등)과 직교적인 축(무엇을 강화할지가 아니라 어디를 양성 샘플로 볼지)이라 결합 가능성이 있다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">[[QueryDet]]의 Cascade Sparse Query가 "어디를 계산할지"를 저해상도 예측으로 좁히는 반면, 이 논문은 "어디를 양성 샘플로 볼지"를 diffusion으로 정제한다 — 두 메커니즘을 결합하면 연산 효율과 샘플 품질을 동시에 개선할 여지가 있다.</mark>
- <mark style="background: #A6E3A1A6;">SAR 영상은 원격탐사 도메인 중에서도 시각적으로 독특한(광학 이미지와 매우 다른) 특성을 갖는데, CLIP이 자연 이미지로 사전학습되었다는 점을 고려하면 SAR 특화 vision-language 사전학습으로 이 방법을 더 개선할 여지가 있어 보인다.</mark>

# 관련 개념
- (없음 — CLIP-Driven Diffusion Anchor Sampling은 이 논문 하나의 통합 기여이지만, 세부 구성 요소(BC-IoU, diffusion sampling)는 각각 이 논문 안에서만 의미 있는 구현으로 판단해 별도 concept 문서로 분리하지 않음. RFLA의 Gaussian receptive field 자체는 아직 원 논문[[rfla]]이 위키에 없어 이 논문만으로 독립 concept화하기엔 이르다고 판단)

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]]

# 읽어볼 만한 논문
- 참고문헌 기반: C. Xu, J. Wang, W. Yang, H. Yu, L. Yu, G.-S. Xia, "RFLA: Gaussian receptive field based label assignment for tiny object detection" (ECCV 2022) [37] — 이 논문이 직접 확장하는 baseline. 이미 `wiki/reading-list.md`에 [[Unc-SOD]] 출처로 등재되어 있으며, 여러 논문에서 반복 인용되는 만큼 우선순위가 매우 높음.
- 참고문헌 기반: A. Radford et al., "Learning transferable visual models from natural language supervision" (CLIP, ICML 2021) [4] — 이 논문의 CLIP 조건화 전체가 기반하는 원조 논문. Vision-language 정렬의 기본 원리 이해에 필수.
- 참고문헌 기반: S. Chen, P. Sun, Y. Song, P. Luo, "DiffusionDet: Diffusion model for object detection" (ICCV 2023) [57] — 바운딩 박스 자체를 diffusion target으로 삼는 원조 접근. 이 논문이 "박스가 아닌 anchor 샘플링 포인트를 diffusion으로 정제한다"는 차별점을 이해하려면 대조 비교가 필요.
- 자유 추천(검증 필요): SAR 도메인 특화 CLIP 사전학습(원격탐사 vision-language foundation model) 관련 연구 — 검색 키워드: `SAR remote sensing CLIP domain-specific pretraining vision-language foundation model 2025`
