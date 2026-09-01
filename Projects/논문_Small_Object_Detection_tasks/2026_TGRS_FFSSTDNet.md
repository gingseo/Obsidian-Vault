---
pm-task: true
projectId: "paperwiki-small-object-detection"
parentId:
id: "t-ffsstdnet-xbe5j57ns9"
title: "From Fuzzy Global to Clear Local: A Focus and Super-Resolution-Guided Tiny Target Detection Method for Full-Scene Images"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-08-05"
due:
progress: 0
assignees: []
tags: []
customFields:
  "7l8l795xmtcjvlf6": 2026
  "1frf59rymtcjvske": "IEEE Transactions on Geoscience and Remote Sensing (TGRS)"
subtaskIds: []
dependencies: []
year: 2026
venue: "IEEE Transactions on Geoscience and Remote Sensing (TGRS)"
jcr_quartile: Q1
task: [small-object-detection]
direction: [improvement]
paper_tags: [paper, small-object-detection, remote-sensing, full-scene-image, focus-detection, super-resolution, region-filtering]
source: "Projects/논문_pdf/Small_Object_Detection/2026_TGRS_FFSSTD-Net.pdf"
source_type: personal
createdAt: "2026-08-18T11:00:00.000Z"
updatedAt: "2026-08-31T00:00:00.000Z"
---

#paper #small-object-detection #remote-sensing #full-scene-image #focus-detection #super-resolution #region-filtering

> [!quote] 원제
> **From Fuzzy Global to Clear Local: A Focus and Super-Resolution-Guided Tiny Target Detection Method for Full-Scene Images**
> Yucong He, Gui Gao, Zhenghuan Xia, Dunyun He, Gang Yang, Xi Zhang, Gaosheng Li — Southwest Jiaotong University / Beijing Institute of Space Information / First Institute of Oceanography, Ministry of Natural Resources / Hunan University, IEEE Transactions on Geoscience and Remote Sensing (TGRS) 2026
> https://doi.org/10.1109/TGRS.2026.3676397

# 한 줄 요약
<mark style="background: #FFF3A3A6;">전체 장면(full-scene) 위성 이미지를 겹치는 패치로 나눈 뒤, 경량 grid 스코어링 모듈(CFD)로 타겟이 있을 만한 패치만 걸러 배경 연산·오탐을 줄이고, 학습 시에만 존재하는 super-resolution 보조 브랜치(FSR)로 backbone이 고해상도 지향적 특징을 학습하도록 유도해, 정보 손실·RONI 연산 비용·샘플 불균형 세 문제를 모델 크기 증가 없이 동시에 완화하는 FFSSTD-Net.</mark>

> [!info] 내 메모
> 

# 정리

## 기존 방법의 한계
- **정보 손실**:
  Tiny object는 backbone의 다중 convolution·다운샘플링을 거치며 공간·의미 정보가 심하게 손상된다. Scale-aware·contextual modeling 전략은 backbone/detector 구조 개선에 그쳐, 약화된 feature를 보강할 보조 정보 없이는 예측 단계에서 정확한 제약을 걸기 어렵다.
- **RONI로 인한 추가 연산 비용**:
  Full-scene 위성 이미지는 규모가 크고 타겟 분포가 불균일해, 배경 영역(RONI)까지 연산해야 해 효율이 떨어지고 오탐 가능성도 커진다.
- **불균형 샘플**:
  기존 sample-oriented 전략은 데이터셋 규모 확장에만 집중해, 저품질·저해상도 샘플로 인한 불균형은 방치한다.

## 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Scale-aware / Contextual modeling**
- Scale-aware(멀티브랜치 hierarchical feature fusion): 오버딥 네트워크의 "black box" 특성상 tiny object feature가 깊은 신호에 묻힘.
- Contextual modeling: 과도한 문맥 정보가 오히려 영역 경계를 흐림.
- **타겟/해결**: 정보 손실(문제①) — 둘 다 network 설계 개선에 그쳐, 이미 약화된 feature를 보강할 별도 보조 정보가 없다.

**갈래 2 — Focused detection (RONI 필터링)**
- 단순 tiling: 서브 영역 분할·리스케일 — 불균일 분포에 취약, 전체 서브 이미지 처리로 시간 소모 큼.
- ClusDet, DMNet 등 clustering/density map: 클러스터 영역 근사 localization — end-to-end 통합 안 됨, 클러스터링 품질에 정확도 좌우.
- PRDet: 영역 grid semantic+contextual learning 결합 — 최적화·전처리 의존으로 모델 복잡도·연산 비용 증가.
- **타겟/해결**: RONI로 인한 추가 연산 비용(문제②) — 연산 절감이라는 목표 자체를 정확히 달성 못하거나(수작업 라벨/클러스터링 의존) end-to-end가 아니다.

**갈래 3 — 샘플 증강/SR 기반 불균형 완화**
- <mark style="background: #FFF3A3A6;">데이터 증강(멀티스케일 타겟 복제): 저해상도·중복 샘플 유입으로 데이터 품질 저해 가능. GAN 기반 SR: 고해상도 증강하지만 가짜 텍스처·아티팩트 생성, 모델 크기 확장으로 연산 비용 급증.</mark>
- **타겟/해결**: 불균형 샘플(문제③) — 정보 손실 완화와 경량성·아티팩트 없는 복원을 동시에 만족하는 end-to-end 통합 프레임워크는 없었다.

**갭**: <mark style="background: #FFF3A3A6;">Focused detection 계열은 연산 절감이라는 목표 자체를 정확히 달성하지 못하거나(수작업 라벨/클러스터링 의존) end-to-end 통합이 안 되고, SR 기반 계열은 정보 손실을 완화하는 대신 새로운 아티팩트나 무거운 연산 비용을 대가로 치른다.</mark>

## 이 논문이 풀고자 하는 문제
1. 정보 손실을 보완할 보조 신호를 추가 라벨 없이 backbone에 주입하는 것.
2. 수작업 라벨링·클러스터링 없이 경량 네트워크만으로 배경 patch를 걸러 연산을 없애는 것.
3. 모델 크기를 키우지 않으면서 저해상도 샘플의 구조적 세부 정보를 보완하는 것.

**갭 종합**: <mark style="background: #FFF3A3A6;">"고해상도 표현 보존"과 "연산 오버헤드 절감"을 동시에, end-to-end로, 추가 라벨 없이 달성하는 통합 프레임워크는 없었다는 것이 이 논문의 통찰이다.</mark>

> [!info] 내 메모
> 

# 해결 방법 요약

| | 문제 ① — 정보 손실 | 문제 ② — RONI로 인한 추가 연산 비용 | 문제 ③ — 불균형 샘플 |
|---|---|---|---|
| **해결 방법** | FSR 모듈이 학습 시에만 원본 이미지 재구성을 강제해 backbone이 고해상도 지향적 특징을 학습하도록 유도 | CFD 모듈이 저수준+고수준 feature 융합 후 grid 단위 이진 분류로 confirmed patch만 통과 | FSR이 auxiliary reconstruction branch로 구조적 세부 정보를 보완하되 추론 시 제거되어 모델 크기 불변 |
| **예상되는 문제점** | FSR이 학습에만 관여하고 추론 시 제거되므로 학습·추론 간 feature 분포 불일치 가능성 | CFD가 실제 타겟 patch를 잘못 배제(false negative)할 위험 | CFD/FSR이 라벨된 관심 영역(GT box)에 의존해 학습되므로, 라벨이 부족·편향된 시나리오에서 적용성 제한 |

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">"흐릿한 전역에서 명확한 지역으로(From Fuzzy Global to Clear Local)"라는 이름처럼, A-ConvNeXt backbone이 만든 다중레벨 feature 위에서 먼저 <span style="color:#c0392b; font-weight:bold;">Convolution Focus Detection(CFD)</span> 모듈이 전역적으로 타겟이 있을 만한 patch를 빠르게 좁히고, <span style="color:#c0392b; font-weight:bold;">Feature Super-Resolution(FSR)</span> 모듈이 그 patch 안에서 backbone이 고해상도 특징을 학습하도록 유도한다. 두 모듈 모두 추론 시 연산량을 늘리지 않는 plug-in 구조다.</mark>

## 전체 파이프라인 (Fig. 2 기준)

```
원본 이미지 (3, ~2900, ~2900급) — Gaofen-2 등 full-scene 위성 영상
       │
       ▼
Crop: 1024×1024 겹침 패치(100px overlap)로 분할       → 다수의 "unconfirmed patches"
       │
       ▼
① A-ConvNeXt Backbone (4 stage, SE 포함)              → F¹(96,·) F²(192,·) F³(384,·) F⁴(768,·)  [j=1..4]
       │              │
   F^sf(=F¹, 저수준)  F^df(=F⁴, 고수준)
       │              │
       ▼              ▼
② CFD 모듈: CR(F^df) 업샘플 + CR(F^sf) concat → 1×1 conv×2 → grid(128×128) 점수 F^A
       │
       ▼
   patch 내 최대 grid 점수 z^mx > 적응 임계값 Z ?  ──아니오──▶ 배경 patch로 폐기(연산 제외)
       │ 예
       ▼
   "confirmed patches"만 통과
       │
       ├──────────────────────────────┐
       ▼                              ▼ (학습 시에만)
③ Detector 경로: PAFPN → 3개 검출 헤드   ③' FSR 모듈: CR(F^df)+upsample(F^sf) concat
       │                                 → Conv×3 → EDSR×3(deconv 포함) → G_FSR (원본 이미지 크기로 복원)
       ▼                                        │
   출력: 박스+클래스 예측                          ▼ L1 loss로 원본 G와 비교(추론 시 브랜치 제거)
```

> [!info] 내 메모
> 

### ① A-ConvNeXt Backbone
- **역할**:
  Full-scene 이미지에서 다중레벨 feature map을 추출한다. ViT 계열은 고해상도 이미지에서 연산량이 크고, 순수 CNN은 계층 깊이와 수용영역 사이 균형이 어렵다는 문제에 대응해, ConvNeXt를 개조한 A-ConvNeXt를 backbone으로 채택했다.
- **구현**:
  입력 patch를 4 stage(j=1..4)를 거쳐 다운샘플링, 각 stage는 A-ConvNeXt block(7×7 conv로 넓은 수용영역 확보) `B_j`(3,3,9,3)개 반복 + 채널 `C_j`(96,192,384,768). 첫 다운샘플링은 표준 ConvNeXt(kernel 4, stride 4) 대신 kernel 3·stride 2로 축소해 초반 과도한 압축을 방지. 각 A-ConvNeXt block 내부에 [[Squeeze_And_Excitation_Channel_Attention]](SE block)을 삽입해 채널별 중요도를 적응적으로 재조정. Normalization은 소배치·고해상도 원격탐사 영상에서 안정적인 LayerNorm 사용.
- **입출력 shape**:
  patch `(3, 1024, 1024)` → stage별 feature `F^j (C_j, H_j, W_j)`, j=1..4. 이 중 `F^sf = F¹`(저수준, 고해상도), `F^df = F⁴`(고수준, 저해상도)를 이후 CFD·FSR이 사용.

```python
# 논문 Eq.(1) 기반 의사코드
F_i_j = AConvNext_j(g_i)   # j = 1,2,3,4, g_i: 입력 patch
```

> [!info] 내 메모
> 

### ② CFD (Convolution Focus Detection) 모듈
- **역할**:
  <span style="color:#c0392b; font-weight:bold;">CFD(Convolution Focus Detection)</span>는 backbone이 이미 만든 저수준·고수준 feature를 재사용해, 각 patch를 grid 단위로 스코어링한 뒤 타겟이 있을 가능성이 낮은 배경 patch를 사전에 제거하는 모듈이다. 클러스터링·밀도 맵 추정 같은 별도 전처리 없이, 경량 conv 몇 개만으로 objectness를 판단한다.
- **구현**:
  `F^sf`를 3×3 stride-2 conv+ReLU(CR block)로 다운샘플링, `F^df`는 같은 해상도로 업샘플링한 뒤 concat, 다시 CR block으로 융합. 융합된 feature map을 S×S(128×128, ablation으로 결정) grid로 나누고, 두 개의 1×1 conv를 activation layer로 통과시켜 grid별 objectness 점수 `F^A`를 산출. 각 grid가 타겟 중심점을 포함하는지를 이진 분류로 프레임화하고, patch 내 grid 점수의 최댓값 `z^mx`가 통계 기반 적응 임계값 `Z`(직전 라운드들의 평균 z^mn·z^mx 기반, ε=0.25)를 넘으면 confirmed patch로 채택.
- **입출력 shape**:
  `F^sf(96, 256, 256)` + `F^df(768, 32, 32)` → 융합 후 S×S(128×128) grid의 objectness map `F^A(1, 128, 128)` → patch 단위 이진 판정(confirmed / discarded).

```python
# 논문 Eq.(2)-(5) 기반 의사코드
F_A = conv1x1_2(CR(concat(CR(F_sf), upsample(F_df))))   # objectness map, S×S grid
z_mx = max(F_A[i,1], ..., F_A[i,s])                       # patch 내 최대 grid 점수
Z = eps * ((mean(z_mn_hist) + mean(z_mx_hist)) / 2) ** 2  # eps=0.25, 적응 임계값
confirmed = z_mx > Z
```

<mark style="background: #FFF9D6A6;">Full-scene 이미지는 대부분 배경(RONI)이고 타겟은 희소하게 분포하므로, 이 grid 단위 스코어링으로 배경 patch를 사전에 배제하면 별도 클러스터링·밀도 맵 추정 없이도 "문제 정의"의 RONI 연산 비용 문제를 해결하고, 배경 노이즈로 인한 오탐도 함께 줄인다 — Table III에서 A-ConvNeXt+CFD가 mAP 52.14→52.61, Fps 16.1→23.8로 정확도·속도를 동시에 개선한 것이 이를 뒷받침한다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> Grid 단위 이진 분류가 배경으로 오판한 patch는 이후 연산에서 완전히 배제되므로, 실제 타겟이 있었다면 회복 불가능한 false negative가 된다. 논문은 "잘못 필터링된 patch는 대체로 객체 수가 매우 적어 원래도 탐지가 어려웠을 patch"라고 설명하지만, Table IV의 정량 지표는 필터링된 patch 안의 recall 손실이 아니라 precision(올바르게 걸러낸 비율)만 보고한다.

> [!info] 내 메모
> 

### ③ FSR (Feature Super-Resolution) 모듈
- **역할**:
  <span style="color:#c0392b; font-weight:bold;">FSR(Feature Super-Resolution)</span>은 CFD가 통과시킨 confirmed patch에 대해서만, backbone이 원본 이미지를 재구성하도록 학습 시에만 지도하는 auxiliary 브랜치다. 목적은 이미지 자체의 해상도 복원이 아니라, backbone이 업샘플링 보간의 blur 대신 실제 고해상도 구조·텍스처를 latent feature에 담도록 유도하는 것이다.
- **구현**:
  `F^df`를 CR block으로 처리 후 `F^sf`를 업샘플링해 concat, 3개의 3×3 conv(채널 점차 축소)를 거쳐 중간 feature `F^mf`를 얻고, EDSR(BatchNorm 제거된 residual conv, 1×1+3×3 conv 조합) 3회 + deconvolution을 거쳐 원본 크기 이미지 `G_FSR`을 복원. 원본 이미지 `G`와의 L1 loss로 지도 — L2 대신 L1을 쓴 이유는 HR feature의 multimodal 분포를 L2가 뭉개 blurry한 예측을 만들기 때문. 추론 시 이 브랜치 전체가 제거되어 backbone·detector 구조는 변경되지 않는다.
- **입출력 shape**:
  `F^sf(96, 256, 256)` + `F^df(768, 32, 32)` → concat·conv 후 `F^mf(·, 256, 256)` → EDSR+deconv → `G_FSR(3, 1024, 1024)` (원본 patch 크기로 복원, L1 loss만 계산하고 추론 시 폐기).

```python
# 논문 Eq.(5)-(6),(10) 기반 의사코드
c_EDSR(F) = c3(c3(F)) + c1(F)                                  # BN 제거된 residual block
G_FSR = EDSR_x3( c3_x3( concat(CR(F_df), upsample(F_sf)) ) )   # 3x deconv 포함
L_s = ||G - G_FSR||_1                                           # L1 loss, 추론 시 브랜치 제거
```

<mark style="background: #FFF9D6A6;">학습 시에만 존재하는 보조 브랜치로 원본 이미지 복원을 강제함으로써, backbone이 다운샘플링으로 손실된 구조·텍스처 정보를 latent space에 담도록 유도한다. 추론 시 브랜치를 제거하므로 "문제 정의"의 정보 손실·샘플 불균형 문제를 모델 크기 증가 없이 완화한다는 것이 GAN 기반 SR 선행 연구와의 핵심 차이다 — Table I에서 FSR 단독 추가만으로 mAP가 43.88→45.78(FAIR1M)/53.65→56.23(DOTA)로 상승한 것이 이를 뒷받침한다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> FSR이 학습에만 관여하고 추론 시 완전히 제거되므로, 학습 때 재구성 loss로 다듬어진 feature 분포와 추론 때(재구성 gradient 없이) 실제로 나오는 feature 분포 사이에 괴리가 생길 수 있다. 논문은 이 학습·추론 간 분포 불일치를 별도로 검증하지 않는다.

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| ① A-ConvNeXt Backbone | (3, 1024, 1024) | F¹~F⁴ (96~768, ·, ·) | 다중레벨 feature 추출 | ConvNeXt 변형 + [[Squeeze_And_Excitation_Channel_Attention]] |
| ② CFD | F^sf(96,·) + F^df(768,·) | patch 단위 confirmed/discarded | 배경 patch 사전 제거(RONI 연산 절감) | CR block + 1×1 conv ×2, grid(128×128) 이진 분류 |
| ③ FSR (학습 시만) | F^sf + F^df | G_FSR (3, 1024, 1024) | backbone의 고해상도 특징 학습 유도 | CR + Conv×3 + EDSR×3(deconv), L1 loss |
| Detector | confirmed patch feature | 박스+클래스 예측 | 최종 검출 | PAFPN + 검출 헤드 3개 |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table I (CFD·FSR 유무, A-ConvNeXt backbone)
**표를 보는 법**: CFD·FSR 열의 체크 유무 조합 4가지 중 첫 행(둘 다 미적용)이 baseline, 마지막 행(둘 다 적용)이 FFSSTD-Net 완성형이다.

| 벤치마크 | 지표 | Before(A-ConvNeXt, CFD·FSR 미적용) | After(FFSSTD-Net) |
|---|---|---|---|
| FAIR1M | mAP / Fps | 43.88 / 18.7 | 46.25 / 22.5 |
| DOTA-v2.0 | mAP / Fps | 53.65 / 16.8 | 56.75 / 21.3 |

> [!note]- 세부 결과 및 Ablation
> #### Table I — CFD·FSR 모듈별 기여 (FAIR1M/DOTA/SODA)
> | CFD | FSR | mAP-FAIR1M | Fps-FAIR1M | mAP-DOTAv2 | Fps-DOTAv2 | mAP-SODA | Fps-SODA |
> |---|---|---|---|---|---|---|---|
> | × | × | 43.88 | 18.7 | 53.65 | 16.8 | 33.81 | 16.9 |
> | √ | × | 44.35 | 22.5 | 53.17 | 21.3 | 34.27 | 22.7 |
> | × | √ | 45.78 | 18.7 | 56.23 | 16.8 | 37.71 | 16.9 |
> | √ | √ | **46.25** | **22.5** | **56.75** | **21.3** | **38.20** | **22.7** |
>
> CFD 단독은 DOTA에서 mAP를 오히려 53.65→53.17로 소폭 낮추지만(속도는 +4.5 Fps) FSR과 결합하면 정확도·속도 모두 최고. FSR의 기여가 정확도 축에서 더 크고, CFD의 기여는 속도 축에서 더 크다.
>
> #### Table II — 다른 backbone에 CFD 이식 (FAIR1M/DOTAv2)
> | Backbone | Params(M) | mAP-FAIR1M | Fps-FAIR1M | mAP-DOTAv2 | Fps-DOTAv2 |
> |---|---|---|---|---|---|
> | ResNet101 | 55.27 | 30.97 | 19.5 | 46.70 | 16.1 |
> | ResNet101+CFD | 55.78 | 31.44 | 22.6 | 47.28 | 21.3 |
> | CSP-DarkNet | 62.24 | 34.06 | 18.4 | 48.10 | 14.6 |
> | CSP-DarkNet+CFD | 62.75 | 34.74 | 20.6 | 48.70 | 19.3 |
> | A-ConvNeXt | 72.34 | 43.88 | 18.7 | 53.65 | 16.8 |
> | A-ConvNeXt+CFD | 72.85 | 44.35 | 22.5 | 53.17 | 21.3 |
>
> CFD는 파라미터를 0.5M 내외만 늘리면서 모든 backbone에서 Fps를 3~5 개선 — plug-and-play 범용성 확인. mAP는 A-ConvNeXt에서만 DOTAv2 기준 소폭 하락, 나머지는 모두 개선.
>
> #### Table III — Full-scene 이미지 실측(Gaofen-2 6장)
> CFD 적용 시 mAP 52.14→52.61, Fps(타일 처리량) 16.1→23.8로 대폭 향상 — 실제 대형 원본 이미지에서도 효과 유지.
>
> #### Table IV — Patch 제거 비율별 정확도/속도 (A-ConvNeXt+CFD)
> | 제거 비율(%) | Precision(%) | mAP | Fps |
> |---|---|---|---|
> | 5 | 99.51 | 52.23 | 17.3 |
> | 10 | 98.44 | 52.46 | 19.2 |
> | 15 | 97.21 | 52.55 | 21.4 |
> | 20 | 95.94 | 52.59 | 24.1 |
> | 30 | 90.78 | 51.66 | 26.2 |
> | 40 | 86.45 | 49.85 | 28.4 |
> | 50 | 80.89 | 45.21 | 31.8 |
>
> 제거 비율 20%(precision 95.94%) 지점이 속도·정확도 균형의 최적점 — 그 이상 공격적으로 제거하면 mAP가 급락(30%부터 하락 시작, 50%에서 45.21까지 급락).
>
> #### Table V — 다른 focused detection 방법과 비교 (CFD만, R-CNN 검출기 기준)
> | 방법 | mAP-FAIR1M | Fps-FAIR1M | mAP-DOTAv2 | Fps-DOTAv2 |
> |---|---|---|---|---|
> | Clusdet | 33.22 | 14.8 | 44.1 | 11.3 |
> | Dmnet | 35.64 | 15.4 | 49.4 | 13.1 |
> | Oan | 42.39 | 23.1 | 53.1 | 21.6 |
> | FFSSTD(CFD만) | 46.25 | 23.2 | 56.75 | 21.3 |
>
> Oan이 속도는 근소하게 앞서지만(23.1 vs 23.2 거의 동일) 정확도는 크게 뒤짐 — CFD가 클러스터링·밀도맵 연산을 없앤 것이 속도 우위의 핵심.
>
> #### Table VI — 다른 backbone에 FSR 이식 (mAP)
> | Backbone | mAP-FAIR1M | mAP-DOTAv2 |
> |---|---|---|
> | ResNet101 | 30.97 | 46.70 |
> | ResNet101+FSR | 31.98(+1.01) | 47.30(+0.60) |
> | CSP-DarkNet | 34.06 | 48.10 |
> | CSP-DarkNet+FSR | 36.06(+2.00) | 50.20(+2.10) |
> | A-ConvNeXt | 43.88 | 53.65 |
> | A-ConvNeXt+FSR | 45.78(+1.90) | 56.23(+2.58) |
>
> FSR은 CSP-DarkNet·A-ConvNeXt에서 특히 큰 이득 — 다중스케일 구조를 가진 backbone과 시너지가 큰 것으로 추정(논문이 명시적 원인 분석은 제공하지 않음).
>
> #### Table VII — 학습/테스트 해상도 조합별 성능 (DOTAv2)
> | 조건 | Train | Test | mAP | Fps |
> |---|---|---|---|---|
> | FFSSTD without SR | 640 | 640 | 48.66 | 24.7 |
> | FFSSTD without SR | 640 | 1024 | 15.47 | 24.7 |
> | FFSSTD without SR | 1024 | 640 | 28.73 | 21.3 |
> | FFSSTD without SR | 1024 | 1024 | 53.65 | 21.3 |
> | FFSSTD with SR | 640 | 640 | 53.27 | 24.7 |
> | FFSSTD with SR | 640 | 1024 | 18.91 | 24.7 |
> | FFSSTD with SR | 1024 | 640 | 34.81 | 21.3 |
> | FFSSTD with SR | 1024 | 1024 | 56.75 | 21.3 |
>
> 학습·테스트 해상도가 다를 때(mismatch) FSR 유무와 무관하게 mAP가 급락 — FSR은 같은 해상도 조건에서의 절대 성능과, mismatch 조건에서의 상대적 강건성(28.73→34.81)을 모두 개선하지만 mismatch 문제 자체를 해소하지는 못함.
>
> #### Table VIII — FSR 입력 feature 조합 비교 (DOTAv2, 1024/1024)
> | 조합 | mAP |
> |---|---|
> | C1, C4(채택) | **56.75** |
> | C1, C3 | 56.24 |
> | C2, C3 | 55.89 |
>
> 저수준(C1)+최고수준(C4) 조합이 최고 — 두 레벨 사이 정보 격차가 클수록 재구성 지도의 보완 효과가 큰 것으로 해석됨.
>
> #### Table IX — SOTA 비교 (FAIR1M/DOTAv2, 파라미터 포함)
> | 방법 | Params(M) | mAP-FAIR1M | Fps-FAIR1M | mAP-DOTAv2 | Fps-DOTAv2 |
> |---|---|---|---|---|---|
> | RetinaNet | 55.27 | 30.97 | 19.5 | 46.70 | 16.1 |
> | RetinaNet-O+CFD+FSR | 55.78 | 32.45 | 22.6 | 47.69 | 20.5 |
> | Faster R-CNN | 60.08 | 32.31 | 18.6 | 47.10 | 15.2 |
> | Faster R-CNN+CFD+FSR | 60.59 | 35.46 | 21.4 | 50.38 | 19.5 |
> | YOLOv3 | 62.24 | 34.06 | 18.4 | 48.10 | 14.6 |
> | YOLOv3+CFD+FSR | 62.75 | 37.13 | 20.6 | 50.84 | 19.2 |
> | Tood | 53.47 | 38.44 | 17.5 | 50.40 | 13.7 |
> | Tood+CFD+FSR | 53.98 | 40.57 | 20.0 | 53.88 | 18.3 |
> | RTMDet | 68.68 | 41.27 | 17.1 | 52.87 | 12.4 |
> | RTMDet+CFD+FSR | 69.19 | 44.47 | 20.1 | 55.36 | 15.8 |
> | Oriented R-CNN | 56.17 | 43.11 | 19.1 | 53.41 | 16.4 |
> | Oriented R-CNN+CFD+FSR | 56.68 | 45.36 | 22.3 | 56.22 | 21.3 |
> | Mamba | 41.24 | 43.23 | 19.2 | 53.42 | 16.4 |
> | Mamba+CFD+FSR | 42.1 | 46.12 | 21.7 | 56.66 | 19.2 |
> | **FFSSTD(A-ConvNeXt+CFD+FSR)** | 72.85 | **46.25** | 22.5 | **56.75** | 21.3 |
>
> 모든 기존 방법에 CFD+FSR을 이식하면 평균 mAP +10%, Fps 최대 +3 frames/s 개선 — plug-and-play 범용성의 핵심 근거. FFSSTD(A-ConvNeXt 기반)가 두 데이터셋 모두 최고 mAP.
>
> #### Table X — SODA-A 클래스별 성능 (9개 클래스, mAP)
> Airplane 56.3으로 타 클래스 대비 크게 높음. Helicopter(24.2)·Windmill(38.2)은 인스턴스 수 적어 상대적으로 낮음. L-vehicle(38.2)·S-vehicle(47.8) 간 혼동 존재하나 FFSSTD가 완화(mAP 38.2로 PKINet 다음 2위권). 전체 mAP 38.2로 RoI Transformer(36.0)·Oriented R-CNN(34.4)·Faster R-CNN(32.5) 상회.
>
> #### Fig. 9 — Feature map 시각화 (저/중/고레벨)
> **보는 법**: 위 행이 FSR 미적용, 아래 행이 FSR 적용 — 밝을수록 강한 활성화. FSR 적용 시 저수준 feature가 더 선명한 구조 정보를, 고수준 feature가 더 뚜렷한 소형 타겟 texture를 보임.
>
> #### Fig. 7, Fig. 8 — CFD 정성 비교
> CFD 적용 후 밀집 선박·차량 시나리오에서 오탐(false alarm)이 눈에 띄게 감소(Fig. 1 challenge 예시와 대응). 활성화된 grid 비율은 전체의 50% 미만으로, 절반 이상의 배경 영역이 연산에서 제외됨.

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- CFD가 실제 타겟을 포함한 patch를 잘못 배제(false negative)할 위험 → <mark style="background: #FF5582A6;">논문은 "잘못 필터링된 patch는 대체로 객체 수가 매우 적어 탐지 자체가 어려웠을 patch"라고 주장하며 정당화하지만, Table IV는 precision(제거 정확도)만 보고할 뿐 필터링으로 인한 recall 손실을 정량 분석하지 않는다.</mark>
- FSR이 학습에만 관여하고 추론 시 제거되므로, 학습·추론 간 feature 분포 불일치 가능성 → <mark style="background: #FF5582A6;">논문에서 별도로 검증하지 않았다.</mark>

### 한계
- <mark style="background: #FF5582A6;">CFD 모듈이 대규모 라벨링된 데이터셋에 강하게 의존한다 — attention을 가이드하려면 사전 라벨된 관심 영역이 필요해, 라벨이 부족하거나 공간적으로 편향된 시나리오에서 적용성이 제한된다고 저자가 명시(Discussion).</mark>
- <mark style="background: #FF5582A6;">FSR의 업샘플링 전략이 여전히 개선 여지가 있다 — 극도로 밀집하거나 저대비인 영역에서 재구성 아티팩트 가능성을 저자가 인정하며, transformer 기반 계층적 복원이나 diffusion 기반 개선을 향후 과제로 제시.</mark>
- <mark style="background: #FF5582A6;">대형/소형 차량(l-vehicle/s-vehicle) 클래스 간 혼동이 여전히 존재 — 크기 정의의 모호성과 학습 샘플의 불완전한 분할에서 기인한다고 분석.</mark>
- 학습·테스트 해상도가 일치하지 않으면(Table VII) FSR 유무와 무관하게 mAP가 급락 — FSR은 이 mismatch를 완화할 뿐 해소하지 못한다.

### 생각할 점
- <mark style="background: #A6E3A1A6;">CFD의 "저해상도 예측으로 고해상도 연산 위치를 좁힌다"는 구조는 [[2022_CVPR_QueryDet|QueryDet]]의 Cascade Sparse Query와 문제의식이 매우 유사하다 — 둘 다 coarse-to-fine 방식으로 배경 연산을 없애지만, QueryDet은 feature pyramid 레벨 간 sparse convolution을, FFSSTD-Net은 patch 단위 grid 스코어링을 쓴다는 점에서 구현 층위가 다르다.</mark>
- <mark style="background: #A6E3A1A6;">FSR의 "학습 시에만 존재하는 auxiliary reconstruction branch"는 [[2024_ECCV_SR-TOD|SR-TOD]]의 self-reconstruction difference map과 목적이 다르다 — SR-TOD는 재구성 오차 자체를 attention prior로 쓰는 반면, FFSSTD-Net은 재구성 학습이 backbone feature 품질을 간접적으로 끌어올리는 정규화 역할만 한다(추론 시 오차 맵을 쓰지 않음).</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">CFD의 patch 필터링과 [[2022_CVPR_QueryDet|QueryDet]]의 CSQ를 결합하면, patch 단위 1차 필터링 후 남은 patch 내부에서 다시 sparse query로 연산을 좁히는 2단계 가속이 가능할 것으로 보임.</mark>
- <mark style="background: #A6E3A1A6;">[[2025_RSASE_RS-TOD|RS-TOD]], [[2025_RemoteSensing_FANet|FANet]]처럼 원격탐사 도메인에서 attention 기반 feature 강화를 다루는 논문들과 달리 이 논문은 "연산 비용 절감"에 초점을 맞춘 유일한 원격탐사 소형 객체 탐지 논문 — MOC의 "아직 못 채운 빈틈"(경량화 계열은 [[2025_ESWA_LSOD-YOLO|LSOD-YOLO]] 하나뿐)을 보완하는 위치.</mark>

> [!info] 내 메모
> 

# 관련 개념
- (없음 — CFD/FSR은 이 논문 안에서만 의미 있는 구현 디테일로 판단, 별도 concept 문서로 만들지 않음)

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]]

# 읽어볼 만한 논문
- 참고문헌 기반: G. Cheng et al., "Towards large-scale small object detection: Survey and benchmarks" (IEEE TPAMI 2023) [14] — 이 논문이 참조하는 4갈래 분류(scale-aware/contextual/focused/sample-oriented) 체계의 원 서베이. [[2024_ECCV_SR-TOD|SR-TOD]] 노트에서도 동일 서베이가 이미 5갈래 분류로 인용되어 교차 확인 가치가 있음.
- 참고문헌 기반: C. Xu, J. Wang, W. Yang, H. Yu, L. Yu, G.-S. Xia, "RFLA: Gaussian receptive field based label assignment for tiny object detection" (ECCV 2022) [38] — 이미 `wiki/reading-list.md`에 [[2026_TIP_Unc-SOD|Unc-SOD]] 출처로 등재된 논문과 동일. Full-scene 원격탐사 타이니 객체 탐지에서도 반복적으로 인용되는 것으로 보아 우선순위가 높음.
- 자유 추천(검증 필요): Focus-and-Detect 계열의 최신 후속 연구(2025~2026) — 이 논문이 인용한 Koyun et al., "Focus-and-detect" (2022)의 최신 확장판 존재 여부. 검색 키워드: `focus and detect small object aerial images 2025 2026`
