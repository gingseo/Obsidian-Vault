---
pm-task: true
projectId: "paperwiki-small-object-detection"
parentId:
id: "t-rtp-net-9fyytojodp"
title: "Collaborative Optimization of Receptive Field and Texture Preservation for Remote Sensing Small Object Detection"
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
  "1frf59rymtcjvske": "IEEE Journal of Selected Topics in Applied Earth Observations and Remote Sensing (JSTARS)"
subtaskIds: []
dependencies: []
year: 2026
venue: "IEEE Journal of Selected Topics in Applied Earth Observations and Remote Sensing (JSTARS)"
jcr_quartile: Q1
task: [small-object-detection]
direction: [improvement]
paper_tags: [paper, small-object-detection, remote-sensing, receptive-field, texture-preservation, attention-mechanism, feature-fusion, lightweight]
source: "/Users/GyeongSeo/Workspace/논문_pdf/Small_Object_Detection/2026_JSTARS_RTP-Net.pdf"
createdAt: "2026-08-24T03:31:00.000Z"
updatedAt: "2026-08-24T03:31:00.000Z"
---

Project: [[논문 읽기|논문 읽기]]
#paper #small-object-detection #remote-sensing #receptive-field #texture-preservation #attention-mechanism #feature-fusion #lightweight

# 한 줄 요약
<mark style="background: #FFF3A3A6;">수용영역 확장과 texture 보존이 근본적으로 상충한다는 관찰에서 출발해, backbone 단계부터 대·소 커널 병렬 depthwise convolution으로 전역 문맥과 국소 texture를 동시에 추출하는 GLEM, 다중 pooling으로 약화된 texture 응답을 적응적으로 복원하는 AWEM, 채널·공간 attention으로 배경-전경을 분리하며 스케일 간 융합을 정제하는 MSAF 세 plug-and-play 모듈을 YOLOv8 기반으로 결합해, DIOR mAP 84.1%·NWPU VHR-10 95.3%·AI-TOD mAP50:95 0.166을 달성하면서 GFLOPs는 오히려 27.2% 줄인 RTP-Net.</mark>

# 문제 정의

### 기존 방법의 한계
- **수용영역 확장과 texture 보존의 근본적 상충**:
  대형 객체의 전역 문맥 의존성을 모델링하려면 large-kernel convolution이나 깊은 계층적 다운샘플링으로 수용영역을 넓혀야 하는데, 이 연산들은 필연적으로 공간 스무딩 효과를 동반해 소형 객체 표현에 필수적인 고주파 texture 응답을 약화시킨다. 반대로 texture 보존을 위해 수용영역을 제한하면 대형 객체에 필요한 전역 문맥 포착 능력이 떨어진다.
- **Scale variation·texture 저하를 독립적 문제로 취급**:
  기존 방법들은 스케일 변화 대응(multi-scale fusion)과 texture 저하 문제를 서로 독립적인 최적화 목표로 다루지만, 실제로는 두 요인이 강하게 얽혀 있다 — 수용영역이 객체의 최적 지각 스케일을 크게 초과하면, 객체 영역 내의 판별적 texture 응답이 주변 배경 잡음의 통계적 특성에 점차 압도된다.
- **주파수/wavelet 기반 접근의 일반화 한계**:
  MFDAFF-Net(주파수 인식), SFFNet(wavelet 기반)처럼 frequency-domain 방법이 시도되었지만, 필터 설계가 까다롭고 서로 다른 ground sampling distance(GSD, 지상 표본 거리)에 대한 일반화가 어렵다.

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Feature pyramid 기반 multi-scale fusion**
- FPN [7], PANet [8], BiFPN [9]: hierarchical top-down pathway로 고레벨 semantic을 저레벨 고해상도 feature에 전파 — scale variation 대응은 발전했으나 texture 보존은 별도로 다루지 않음.

**갈래 2 — Attention 기반 feature 정제**
- Channel-wise·spatial attention [10]: 정보성 영역을 선택적으로 강조 — texture 저하의 근본 원인(수용영역 확장 시의 스무딩)은 건드리지 않고 사후에 재가중할 뿐.

**갈래 3 — Frequency-domain 접근**
- MFDAFF-Net [18](Tian et al.): dual attention 기반 주파수 인식 UAV 객체 탐지.
- 언어 가이드 spatial-frequency 네트워크 [19](Wu et al.): 원격탐사 change detection.
- SFFNet [20](Yang et al.): wavelet 기반 spatial-frequency fusion, semantic segmentation.
- 공통 한계: 필터 설계가 정교해야 하고 GSD가 바뀌면 일반화가 어려움.

**갈래 4 — Implicit neural representation**
- Zhang et al. [21]: implicit neural representation 기반 적대적 예제 생성 — fine detail 모델링에 잠재력이 있지만 최적화가 복잡해 실시간 탐지 프레임워크에 통합하기 어려움.

**갭**: <mark style="background: #FFF3A3A6;">선행 연구들은 scale variation과 texture 저하를 독립적 목표로 다루거나, frequency-domain·implicit representation처럼 계산 비용이 크거나 일반화가 어려운 우회 경로를 택했다. 두 문제의 내재적 결합(coupling)을 spatial domain에서 직접, 그리고 경량으로 다루는 협업적 최적화(collaborative optimization) 프레임워크는 없었다.</mark>

### 이 논문이 풀고자 하는 문제
1. Backbone 단계에서부터 전역 문맥(수용영역 확장)과 국소 texture를 동시에 보존하는 것 — 단일 수용영역 확장으로 인한 texture 손실 자체를 원천 방지
2. 확장 과정에서 불가피하게 발생한 texture 저하를 스케일 적응적으로 복원·보상하는 것
3. 서로 다른 스케일의 feature를 융합하는 단계에서도 전경-배경을 명시적으로 분리해 오탐·미검출을 줄이는 것

# 제안 방법

<mark style="background: #FFF3A3A6;">GLEM(Global-Local Feature Extraction Module)이 feature 추출의 원천(source) 단계에서부터 대·소 커널 depthwise convolution을 병렬로 적용해 texture 손실 자체를 예방하고, AWEM(Adaptive Weighted Enhancement Module)이 그럼에도 발생한 texture 약화를 다중 pooling 기반 통계로 적응적으로 복원하며, MSAF(Multi-Scale Attention Fusion Module)이 이중 attention(CBAM)으로 스케일 간 융합 시 전경-배경을 정제한다 — GLEM은 소스에서, AWEM은 적응된 feature 공간에서, MSAF는 융합 단계에서 순차적으로 개입하는 3단계 파이프라인.</mark>

### ① Global-Local Feature Extraction Module (GLEM)
- **Multi-scale 추출**: 입력에 5×5, 7×7 depthwise separable convolution(DWConv)을 병렬 적용 — 5×5는 중간 스케일 국소 구조·edge, 7×7은 대규모 전역 문맥(배경 layout 사전 정보)을 포착, 두 출력을 element-wise 합산 후 1×1 conv로 융합(`F_ms`).
- **Attention 가중 예비 강화**: SE 채널 attention으로 소형 객체에 유의미한 고주파 채널을 증폭하고 저주파(배경) 채널을 억제, residual 연결(GELU+LayerNorm)로 vanishing gradient 방지 및 깊은 레이어에서의 소형 객체 디테일 손실 예방.
- **문맥+정제 브랜치 융합**: 1×1 conv로 차원 축소 후 두 브랜치로 분기 — 상단 브랜치(7×7 DWConv)는 대수용영역 전역 문맥, 하단 브랜치는 Feature Refinement Module(CRM)로 고/저주파 성분을 명시적으로 분리(5×5 DWConv 다운/업샘플링으로 저주파 배경 정보 추출 + residual 경로로 고주파 texture 보존)해 두 브랜치를 concat+1×1 conv로 통합.

> [!example]- 구현 디테일
> ```
> F5 = DWConv5×5(X);  F7 = DWConv7×7(X);  F_ms = Conv1×1(F5+F7)
> Xd = Conv1×1(X + LN(GELU(f_att)))          # f_att: 채널 attention 응답
> ctx' = LN(GELU(DWConv7×7(Xd)))              # 상단: 전역 문맥
> lx = LN(DWConv3×3(GELU(Xd ⊙ U(Xd))))        # 하단 CRM 저주파
> hx = LN(DWConv3×3(GELU(Xd ⊖ U(Xd))))        # 하단 CRM 고주파(residual)
> ```
> U(·)는 large-kernel depthwise conv+bilinear interpolation 기반 다운·업샘플링. GLEM은 backbone 5개 레벨 모두와 neck 3개 레벨에 반복 삽입(Fig. 2).

<mark style="background: #FFF9D6A6;">"문제 정의"의 첫 번째 문제(수용영역 확장과 texture 보존의 상충)를, 두 목표를 순차가 아니라 병렬 브랜치로 동시에 추구함으로써 정면 돌파한다 — 대커널 브랜치가 전역 문맥을, 소커널 브랜치가 texture를 각자 전담하므로 어느 한쪽을 위해 다른 쪽을 희생시키는 단일 수용영역 확장의 trade-off 자체가 설계 단계에서 제거된다.</mark>

### ② Adaptive Weighted Feature Enhancement Module (AWEM)
- GLEM으로도 완전히 막지 못한 texture 약화를 보완하기 위해, 기존 SPPF 모듈을 대체하는 dual-branch pooling 구조를 도입.
- Max pooling 브랜치(5×5, stride 1, 3회 순차 적용)로 국소 최댓값(peak) 응답 보존, adaptive max/average pooling(출력 1×1, 전체 공간으로 broadcast)으로 전역 통계 문맥 추출 — 소형 객체가 로컬 노이즈에 압도되지 않도록 전역 기준을 함께 제공.
- 로컬 feature `X_r`과 5개 multi-scale context(`y1~y5`)를 concat+1×1 projection으로 결합한 뒤 채널 attention(SENet 방식)으로 최종 재조정.

> [!example]- 구현 디테일
> ```
> Xr = CBS1×1(X)
> {y1,y2,y3} = MaxPool5×5^(3)(Xr)             # stride1, 3회 순차 = 5×5/9×9/13×13 유효 수용영역
> {y4,y5} = Expand(AdaptivePool1×1(Xr))        # adaptive max+avg pooling
> Z = CBS1×1(Concat(Xr,y1,y2,y3,y4,y5))
> F_AWEM = ChannelAttention(Z)
> ```
> 파라미터 추가 없이 순수 pooling 계층 반복으로 5×5/9×9/13×13 유효 수용영역을 구현 — 학습 가능한 파라미터 증가 없이 multi-scale context 확보.

<mark style="background: #FFF9D6A6;">"문제 정의"의 두 번째 문제를, 국소 peak 응답(texture 디테일)과 전역 통계(배경과의 상대적 대비)를 함께 제공해 소형 객체 신호가 배경 잡음에 묻히지 않도록 재조정함으로써 해결한다 — Table III에서 AWEM 단독 추가만으로 mAP50 91.1→91.9(+1.7%p 상대), 특히 mAP50:95가 63.9→65.6(+1.7)로 개선되어 정밀한 localization에 기여함을 확인.</mark>

### ③ Multi-Scale Attention Fusion Module (MSAF)
- 서로 다른 3개 레벨(P1/P2/P3)의 feature를 표준 convolution+atrous convolution(대형 스케일 브랜치)을 섞어 처리한 뒤 소/중/대 3개 스케일 feature(F1/F2/F3)로 재구성.
- 재구성된 feature를 concat 후 CBAM(채널+공간 attention 순차 적용)으로 이중 정제 — 전경-배경 분리를 명시적으로 강화해 오탐(false positive)·미검출(false negative)을 줄임.

<mark style="background: #FFF9D6A6;">"문제 정의"의 세 번째 문제를, 단순 concat 기반 융합이 아니라 CBAM의 이중 attention으로 융합 이후에도 전경-배경 경계를 다시 한번 정제함으로써 해결한다 — Table III에서 MSAF 단독 추가 시 recall이 89.3%로 최고치를 기록해(다른 단일 모듈 조합보다 높음), 놓치는 객체를 줄이는 데 특히 효과적임을 시사.</mark>

# 실험 결과

### 핵심 결과 (RTP-Net 완전체, 4개 벤치마크)
| 벤치마크 | 지표 | Before(경쟁 SOTA/baseline) | After(RTP-Net) |
|---|---|---|---|
| DIOR | mAP | 66.9(Faster R-CNN)~83.3(YOLOv8n) | 84.1 |
| NWPU VHR-10 | mAP | 94.57(GLSANet, 2위) | 95.3 |
| AI-TOD | mAP50:95 | 0.148(QueryDet, 최고 경쟁) | 0.166 |
| RSOD(ablation) | mAP50 / Recall | 91.1(baseline) / 88.3 | 94.3 / 91.2 |

> [!note]- 세부 결과 및 Ablation
> #### RSOD 메인 ablation(Table III·IV)
> | AWEM | GLEM | MSAF | mAP50 | Recall |
> |---|---|---|---|---|
> | - | - | - | 91.1 | 88.3 |
> | ✓ | - | - | 91.9 | 89.0 |
> | - | ✓ | - | 92.9 | **92.0**(최고 recall) |
> | - | - | ✓ | 92.9 | 89.3 |
> | ✓ | ✓ | - | 93.0 | 89.2 |
> | ✓ | - | ✓ | 93.6 | 90.7 |
> | - | ✓ | ✓ | 93.9 | 88.9 |
> | ✓ | ✓ | ✓ | **94.3** | 91.2 |
> - GLEM 단독이 recall에서 가장 큰 단일 기여(88.3→92.0) — 저자는 이를 GLEM이 "receptive field 모델링을 최적화"하기 때문이라고 해석.
> - AWEM 단독은 mAP50:95를 63.9→65.6로 개선(정밀 localization에 특화).
> - MSAF 단독은 CBAM 기반 배경 억제 효과.
>
> #### DIOR SOTA 비교(Table V, 20클래스 mAP)
> | 방법 | mAP |
> |---|---|
> | Faster R-CNN | 66.9 |
> | Cascade R-CNN | 67.4 |
> | YOLOv7 | 79.6 |
> | YOLOv8 | 83.3 |
> | GLSANet | 79.9 |
> | LGDA | 74.1 |
> | **RTP-Net(ours)** | **84.1** |
> - Chimney(CM)처럼 배경 간섭이 강한 밀집 소형 객체 클래스와, Dam(DM)·항만 서비스 구역(EA)·톨스테이션(ES) 같은 대형 객체 클래스 모두에서 최고/2위권 — 저자는 이를 "대소 객체 모두에 균형 잡힌 적응력"으로 해석.
>
> #### NWPU VHR-10 SOTA 비교(Table VI)
> RTP-Net 95.3% mAP로 LGDA(94.08%), YOLOv8(93.61%), Faster R-CNN(88.30%) 상회. Bridge(86.1%)는 GLSANet(97.51%)에 못 미치는 유일한 클래스로 남음.
>
> #### AI-TOD SOTA 비교(Table VII) — Deformable-DETR·DN-Deformable-DETR 포함
> | 방법 | mAP50 | mAP50:95 |
> |---|---|---|
> | YOLOv8 | 0.325 | 0.149 |
> | DetectoRS | 0.328 | 0.148 |
> | QueryDet | 0.293 | 0.122 |
> | Deformable-DETR | 0.305 | 0.101 |
> | DN-Deformable-DETR | 0.325 | 0.127 |
> | **RTP-Net** | **0.403** | **0.166** |
> - Transformer 계열(Deformable-DETR류)까지 포함해 전 지표 최고 — CNN 기반 경량 설계로 DETR 계열을 상회한다는 점이 [[BAFNet]]과 유사한 패턴.
>
> #### 세밀 스케일 분석(Table VIII, AI-TOD "Small" 카테고리 세분화)
> | 스케일 구간 | 절대 픽셀 크기 | YOLOv8 mAP50:95 | RTP-Net mAP50:95 |
> |---|---|---|---|
> | Very Tiny | 2-8px | 0.0845 | 0.1023 |
> | Tiny | 8-16px | 0.1997 | 0.2032 |
> | Small | 16-32px | 0.3306 | 0.3423 |
> - Very Tiny 구간에서 가장 큰 상대 개선(0.0845→0.1023, +21%) — GLEM·AWEM이 극도로 공격적인 다운샘플링에서도 판별적 feature를 보존함을 시사.
>
> #### 경량성 비교(Table IX)
> | 모델 | GFLOPs | Params(M) | FPS | mAP50 |
> |---|---|---|---|---|
> | YOLOv8 | 8.1 | 3.01 | 72.84 | 0.911 |
> | **RTP-Net** | **5.9(−27.2%)** | 1.85(−38.5%) | **76.92** | **0.943** |
> - 정확도 개선과 동시에 연산량·파라미터·FPS 모두 개선 — "성능 향상이 반드시 연산 비용 증가를 동반한다"는 이 위키의 다른 다수 feature 강화 논문과 달리, RTP-Net은 경량화와 정확도를 동시에 달성한 예외적 사례.
>
> #### Attention 메커니즘 비교(Table X, RSOD)
> Baseline(YOLOv8) 91.1 < SE Attention 91.4 < CoordAtt 92.4 < CBAM 93.2 < **RTP-Net(전체) 94.3** — 범용 attention 모듈을 단순 삽입하는 것보다 GLEM+AWEM+MSAF의 협업적 설계가 우수함을 대조.

# Discussion

### 이 아이디어의 잠재적 부작용
- GLEM이 backbone 5개 레벨과 neck 3개 레벨 모두에 반복 삽입되어(Fig. 2), 모듈이 아키텍처 전반에 걸쳐 광범위하게 개입 → <mark style="background: #FF5582A6;">이렇게 많은 삽입 지점에도 불구하고 GFLOPs가 오히려 감소한 이유(depthwise separable convolution의 효율성, 파라미터 비공유 등)에 대한 정량적 분해(breakdown) 분석은 제시되지 않는다.</mark>
- AWEM의 채널 attention이 SENet 방식(global average pooling+FC)에 의존 → <mark style="background: #FF5582A6;">SENet류 채널 attention이 미세한 공간적 위치 정보를 압축해버리는 일반적 한계를 이 논문도 그대로 상속하는지, 별도로 검증하지 않는다.</mark>

### 한계
- <mark style="background: #FF5582A6;">NWPU VHR-10의 Bridge 클래스에서 GLSANet(97.51%)에 뚜렷이 못 미침(86.1%) — 저자는 이 격차의 원인을 분석하지 않는다.</mark>
- <mark style="background: #FF5582A6;">저자가 Conclusion에서 직접 명시: 향후 transformer 기반 아키텍처와의 통합으로 장거리 의존성 모델링을 개선할 계획, 도메인 적응(domain adaptation)으로 센서/촬영 조건 간 일반화를 개선할 계획, 멀티모달 데이터 융합 확장 계획 — 현재 버전은 순수 CNN 기반이며 단일 센서·단일 모달리티에 국한됨을 인정.</mark>
- 실험이 모두 단일 GPU(RTX 3080) 환경에서 수행되어, 더 큰 배치·분산 학습 환경에서의 재현성은 검증되지 않음.
- GLEM의 대·소 커널 조합(5×5/7×7)이 고정되어 있어, 이 커널 크기 선택 자체에 대한 ablation(예: 3×3/9×9 등 다른 조합과의 비교)이 제시되지 않는다 — 왜 5×5/7×7이 최적인지 근거가 약함.

### 생각할 점
- <mark style="background: #A6E3A1A6;">이 논문이 명시한 "수용영역 확장과 texture 보존의 근본적 상충"이라는 문제의식은, 이 위키의 [[Deformable_Convolutional_Networks]]가 다루는 "고정 grid 샘플링" 문제나 [[BAFNet]]의 "cross-scale fusion 중 경계 손실" 문제와 동일한 상위 딜레마(넓게 보되 세밀함을 잃지 않기)의 세 번째 변주로 볼 수 있다 — 세 논문이 각각 적응적 오프셋(deformable conv), 경계 auxiliary supervision(BAFNet), 병렬 다중 커널(RTP-Net)이라는 서로 다른 해법을 제시한다는 점에서, 이 딜레마를 관통하는 해법 분류학을 만들어볼 가치가 있다.</mark>
- <mark style="background: #A6E3A1A6;">Table IX에서 GFLOPs·파라미터·FPS·mAP 모두가 동시에 개선된 것은 이 위키에서 드문 사례다 — [[LSOD-YOLO]]·[[FFCA-YOLO]]가 경량화-정확도 trade-off를 관리하는 데 그친 반면, RTP-Net은 depthwise separable convolution과 파라미터 비공유 pooling(AWEM)만으로 둘 다 개선했다는 점에서 설계 효율이 특히 눈에 띈다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">GLEM의 CRM(고/저주파 성분 명시적 분리)은 이 위키의 [[UAV-DETR]]·[[FANet]]이 쓰는 FFT 기반 주파수 도메인 분리와 목적은 유사하지만, spatial domain에서 large-kernel downsampling/upsampling 차이(residual)로 고주파를 근사한다는 점에서 훨씬 저비용이다 — 두 접근(FFT 기반 vs spatial 근사 기반)의 정확도-비용 trade-off를 직접 비교하면 흥미로운 결과가 나올 것으로 보인다.</mark>
- <mark style="background: #A6E3A1A6;">Table VIII의 세밀 스케일 분석(Very Tiny/Tiny/Small)은 이 위키의 dynamic query DETR 계열이 다루는 "밀도"와는 다른 축(개별 객체의 절대 크기)에서 소형 객체 성능을 세분화한다 — 이 두 축(밀도 vs 절대 크기)을 교차한 세밀 분석은 이 위키에서 아직 이뤄지지 않았다.</mark>

# 관련 개념
- [[Collaborative_Receptive_Field_Texture_Optimization]] — 이 논문의 핵심 기여. 수용영역 확장(전역 문맥)과 texture 보존(국소 디테일)을 backbone 단계부터 병렬 브랜치로 동시에 추구해, 두 목표 간 근본적 trade-off 자체를 설계로 해소하는 프레임워크.

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]] — feature 강화 계열에 새로 추가. 다음 comparison 갱신 시 반영 예정. [[BAFNet]]과 함께 이번 배치에서 처리한 두 번째 원격탐사 feature 강화 논문이며, 유일하게 경량화(GFLOPs 감소)까지 동시에 달성.

# 읽어볼 만한 논문
- 참고문헌 기반: S. Tian, B. Zhang, L. Cao, S. Fan, K. Du, C. Fu, Y. Zhang, "Mfdaff-net: Multiscale frequency-aware dual attention-guided feature fusion network for uav object detection" (2025) [18] — 이 논문이 "필터 설계 어려움·GSD 일반화 한계"의 예로 직접 대조하는 주파수 인식 선행 연구.
- 참고문헌 기반: S. Woo, J. Park, J.-Y. Lee, I. Kweon, "CBAM: Convolutional block attention module" (ECCV 2018) [23] — MSAF가 채택한 CBAM의 원조 논문. Table X에서 직접 비교 대상으로도 등장.
- 참고문헌 기반: J. Hu, L. Shen, G. Sun, "Squeeze-and-excitation networks" (TPAMI 2020) [22] — AWEM·GLEM의 채널 attention이 기반하는 SENet 원조. Table X 비교 대상.
- 자유 추천(검증 필요): Depthwise separable convolution 기반 large-kernel 설계가 파라미터 효율성과 정확도를 동시에 달성하는 다른 경량 backbone 연구 — 검색 키워드: `large kernel depthwise separable convolution lightweight backbone efficiency accuracy tradeoff`. RTP-Net의 GFLOPs 감소와 정확도 개선 동시 달성이 어떤 일반 원리에서 오는지 배경 이해에 도움될 것으로 예상.

---
**보안 참고**: PDF 전체를 확인했으며, 프롬프트 인젝션이나 지시문처럼 보이는 텍스트는 발견되지 않았다.
