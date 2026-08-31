---
pm-task: true
projectId: "paperwiki-salient-object-detection"
parentId:
id: "t-aimrinet-96b444e0cy"
title: "Attention interaction and multiple residual integration network for salient object detection in remote sensing images"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-08-24"
due:
progress: 0
assignees: []
tags: []
customFields:
  "5t6guexamtck1e8y": 2026
  "njv4e7krmtck1e8z": "Image and Vision Computing (Elsevier)"
subtaskIds: []
dependencies: []
year: 2026
venue: "Image and Vision Computing (Elsevier)"
jcr_quartile: Q1
task: [salient-object-detection]
direction: [improvement]
paper_tags: [paper, salient-object-detection, remote-sensing, transformer, spatial-attention, residual-fusion, multi-scale-feature]
source: "/Users/GyeongSeo/Workspace/논문_pdf/Salient_Object_Detection/2026_Image-and-Vision-Computing_AIMRINet.pdf"
createdAt: "2026-08-24T03:40:00.000Z"
updatedAt: "2026-08-24T03:40:00.000Z"
---

Project: [[논문 읽기|논문 읽기]]
#paper #salient-object-detection #remote-sensing #transformer #spatial-attention #residual-fusion #multi-scale-feature

# 한 줄 요약
<mark style="background: #FFF3A3A6;">PVT-v2 backbone의 얕은/깊은 feature를 channel shuffle+4분할 progressive spatial attention으로 점진적으로 상호작용시켜 공간 디테일과 semantic을 동시에 강화하는 SAI 모듈과, element-wise 곱으로 여러 레벨의 공통 saliency 정보를 추출한 뒤 다중 residual 연결로 원본 디테일까지 보존하는 MRFI 모듈을 결합해, ORSSD/EORSSD/ORSI-4199 세 원격탐사 SOD 벤치마크에서 18개 기존 방법 대비 평균 MAE −0.0002, maxF +0.31%p를 달성한 AIMRINet.</mark>

# 문제 정의

### 기존 방법의 한계
- **전역 문맥과 국소 디테일의 균형 부재**:
  광학 원격탐사 이미지는 객체 스케일 변화가 극심하고 공간 분포가 불균일하며 배경 간섭이 복잡해, 전역 문맥(global context)과 국소 디테일(local detail) 사이의 균형을 잡기 어렵다. CNN 기반 방법은 convolution 커널의 국소성 때문에 전역 의존성 모델링이 부족하고, Transformer 기반 방법은 self-attention의 전역 모델링 특성 때문에 오히려 국소 공간 구조·경계 디테일 인지력이 약화된다.
- **다중 레벨 feature 통합 메커니즘의 부재**:
  Backbone에서 추출되는 얕은 feature(디테일 정보 풍부)와 깊은 feature(semantic 정보 풍부)는 서로 다른 특성을 갖는데, 이를 충분히 탐색·활용하는 효과적인 통합 메커니즘이 부족해 다중 스케일 feature 간 상호보완 정보를 완전히 활용하지 못한다.
- **위 두 문제의 결과**:
  기존 방법 대다수가 이 문제들을 완전히 해소하지 못해, feature 표현이 불완전해지고 salient object의 경계가 모호해지는 결과로 이어진다.

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — 자연 이미지 SOD의 attention/edge 강화**
- Gao et al. [20](SAM): multi-scale adaptive learning으로 fine-grained feature enhancement, 극소 파라미터.
- Ren et al. [21]: shallow global attention으로 전역 semantic+국소 디테일 동시 포착, redundant 정보 완화.
- Yang et al. [22]: salient object와 edge를 end-to-end feature fusion으로 공동 최적화.
- Wu et al. [23], Jiang et al. [26], Nam et al. [27], Wu et al. [28], Chen et al. [29], Liu et al. [30]: 계층적/dual-path/DETR 기반 확장, boundary 정제 등 다양한 시도.
- 공통 한계: 자연 이미지 도메인 특화라, 극심한 스케일 변화·복잡 배경을 특징으로 하는 원격탐사 이미지에는 그대로 적용하기 어려움.

**갈래 2 — 원격탐사 SOD (ORSI-SOD) 특화**
- Li et al. [34]: dynamic semantic matching 압축 + edge self-alignment.
- Liu et al. [35]: multi-decoder + super-resolution 지식 전이.
- Luo et al. [36]: multi-scale attention interaction, edge-enhanced semantic feature.
- Zhao et al. [37]: 전역 pixel coordination + multi-level feature interaction + multi-scale receptive field.
- Quan et al. [38], Liang et al. [39], Di et al. [40], Lee et al. [41], Fang et al. [42]: global semantic enhancement, edge-embedded attention, 2단계 weighted fusion, dual-branch, edge-aware interaction 등.
- Yan et al. [43], Zhao et al. [44], Liu et al. [45]: CNN+Transformer 결합, weight 최적화, hybrid encoder.
- Sun et al. [46](주파수 도메인 global receptive field), Liu et al. [47](계층적 branch+deep supervision+ensemble), Dong et al. [48](large kernel decoding+multi-gate), Wang et al. [49](경량 boundary-supervised), Zhao et al. [50], Yang et al. [51](Mamba 기반), Meng et al. [52], Ge et al. [53](dual-stream cross-attention).
- 공통 한계: 저자들이 정리한 두 가지 미해결 문제 — (1) 얕은 feature(디테일)와 깊은 feature(semantic)를 충분히 탐색·활용하는 메커니즘 부족, (2) 다중 스케일 feature 간 효과적 통합 메커니즘 부재로 인한 불완전한 검출·모호한 경계.

**갭**: <mark style="background: #FFF3A3A6;">기존 ORSI-SOD 연구들은 CNN/Transformer 혼합, edge 강화, multi-scale fusion 등 다양한 전략을 시도했지만, "얕은/깊은 feature 간 점진적 공간 상호작용"과 "다중 레벨 feature의 공통 saliency 정보를 명시적으로 추출하면서 원본 디테일도 다중 residual로 보존"을 하나의 통일된 프레임워크에서 결합한 시도는 충분히 탐구되지 않았다.</mark>

### 이 논문이 풀고자 하는 문제
1. 얕은 feature(공간 디테일)와 깊은 feature(고수준 semantic) 각각의 표현력을 레이어 간 점진적 공간 상호작용으로 강화하는 것
2. 서로 다른 레벨의 feature가 공유하는 saliency 정보를 명시적으로 추출하면서, 동시에 각 레벨 고유의 디테일 정보도 잃지 않는 것
3. 위 두 메커니즘을 결합해 전역 문맥과 국소 디테일 사이의 균형을 개선하는 것

# 제안 방법

<mark style="background: #FFF3A3A6;">PVT-v2를 backbone으로, SAI(Spatial Attention Interaction) 모듈이 얕은/깊은 feature 각각에 channel shuffle 후 4분할한 sub-feature들을 순차적으로 spatial attention 상호작용시켜 국소 구조 인지력을 강화하고, MRFI(Multiple Residual Feature Integration) 모듈이 SAI로 강화된 shallow/middle/deep 세 feature의 공통 saliency를 element-wise 곱으로 추출한 뒤 다중 residual 연결로 원본 정보를 보존해 최종 saliency map을 생성한다.</mark>

### ① Spatial Attention Interaction (SAI) Module
- Backbone 4개 스케일(P1~P4) 중 가장 얕은 `f1`과 가장 깊은 `f4`에 SAI를 적용(중간 `f2`, `f3`는 conv block만 거쳐 단순 합산 `Y`로 결합).
- 입력 feature `f'_i`(채널 32)를 channel shuffle 후 4개 그룹(각 8채널)으로 균등 분할, 각 그룹에 spatial attention(SA)을 순차 적용 — 이전 그룹의 attention 강화 결과를 다음 그룹 처리에 element-wise로 더해 그룹 간 정보를 점진적으로 전파.
- 4개 그룹의 attention 강화 결과를 concat 후 conv block+sigmoid로 attention 응답 맵 생성, 원본 shuffled feature와 residual 연결로 최종 출력 `X`(또는 `Z`) 생성.

> [!example]- 구현 디테일
> ```
> si = Split(CS(f'i))                                    # channel shuffle 후 4분할, i=1,4
> s'i1 = si1 ⊗ SA(si1)
> s'ij = sij ⊗ SA(sij ⊕ s'i(j-1)),  j=2,3,4               # 점진적 그룹 간 전파
> X(Z) = si + Sigmoid(ReLU(BN(Conv(Concat(s'i1,...,s'i4)))))
> ```
> 채널 분할 수 ablation(Table 7): 2/4/8분할 비교 시 4분할(채택)이 최적 — 분할 수가 늘수록 그룹당 채널(8→4)이 줄어 의미있는 feature 추출이 어려워짐. Channel shuffle 제거 시 전 지표 하락 확인(비인접 채널 결합 효과 입증).

<mark style="background: #FFF9D6A6;">"문제 정의"의 첫 번째 문제(전역-국소 균형 부재)를, 얕은/깊은 feature 모두에 공간적 상호작용을 도입하되 이를 한 번에 전체가 아니라 4개 그룹으로 나눠 점진적으로 전파시킴으로써 해결한다 — 그룹별 순차 처리가 서로 다른 feature 부분공간(subspace)을 학습하도록 강제해(저자 표현: "네트워크가 상보적 feature 부분공간을 학습하도록 강제"), 단순 전체 attention보다 더 다양하고 견고한 공간 표현을 얻는다.</mark>

### ② Multiple Residual Feature Integration (MRFI) Module
- SAI로 강화된 얕은 feature `X`, 중간 feature `Y`, 깊은 feature `Z`를 동일 해상도로 업샘플링 정렬 후 element-wise 곱으로 세 feature의 공통 saliency 정보만 추출한 융합 feature `F_m` 생성 — 곱셈 연산이 비공통(redundant) 정보와 노이즈를 자연스럽게 억제.
- `F_m`을 conv block으로 처리한 뒤, `X`·`Y`·`Z` 각각에 residual로 더해 3개의 개별 강화 feature `F_X`, `F_Y`, `F_Z` 생성 — 공통 saliency 정보와 레벨별 고유 정보를 함께 보존.
- 세 강화 feature를 concat+conv로 결합한 뒤, 디테일 정보가 가장 풍부한 `X`를 다시 한번 residual로 더해 최종 통합 feature `F_XYZ` 완성.

> [!example]- 구현 디테일
> ```
> Fm = X ⊗ UP(Y) ⊗ UP²(Z)
> FX = X + ReLU(BN(Conv(Fm)));  FY = Y + ReLU(BN(Conv(Fm)));  FZ = Z + ReLU(BN(Conv(Fm)))
> F'XYZ = Concat(FX, FY, FZ)
> FXYZ = ReLU(X + ReLU(BN(Conv(F'XYZ))))
> ```
> 중간 residual block 제거(MRFI_wo_mid, 입력을 바로 concat) 시 성능 하락, 전체 residual 연결 제거(MRFI_wo_res) 시 성능이 크게 하락(Table 7) — residual 연결이 원본 feature를 최종 출력에 통합하는 역할이 실제로 유효함을 확인.

<mark style="background: #FFF9D6A6;">"문제 정의"의 두 번째 문제(다중 레벨 feature 통합 메커니즘 부재)를, 곱셈으로 "공통점(진짜 salient 신호)"을 뽑아내고 residual로 "차이점(레벨별 고유 디테일)"을 보존하는 이중 전략으로 해결한다 — 곱셈만으로는 개별 레벨의 고유 정보가 사라질 위험이 있는데, 다중 residual 연결이 이를 방지해 salient object의 완전성과 경계 디테일을 함께 유지한다.</mark>

# 실험 결과

### 핵심 결과 (ORSSD/EORSSD, Table 1)
| 벤치마크 | 지표 | Before(2위, UGNet26) | After(AIMRINet) |
|---|---|---|---|
| ORSSD | MAE / maxF / wFm / Sm | 0.0065 / 0.9244 / 0.9080 / 0.9031(2위 값들 조합) | 0.0063 / 0.9308 / 0.9180 / 0.9434 |
| EORSSD | MAE / maxF / wFm / Sm | 0.0049 / 0.9080 / 0.8919 / 0.9031(2위 값들 조합) | 0.0044 / 0.9080 / 0.8919 / 0.9031(동률/근소 우위) |

> [!note]- 세부 결과 및 Ablation
> #### ORSSD/EORSSD 세부 (Table 1, 19개 방법 비교)
> - ORSSD: AIMRINet MAE 0.0063(2위 대비 −0.0002), maxF 0.9308(+0.01%p), wFm 0.9180(+0.16%p), Sm 0.9434(+0.23%p) — 4개 지표 모두 최고.
> - EORSSD: MAE 0.0044(2위 UGNet 0.0049 대비 −0.0005), maxF 0.9080(+0.52%p 대비 2위), wFm 0.8919, Sm 0.9031 — 저자 서술상 maxF/wFm/Sm 모두 2위 대비 개선(+0.52%/+0.34%/+0.02%).
>
> #### ORSI-4199 (Table 2, 11개 방법 비교)
> AIMRINet MAE 0.0266(공동 최고), maxF 0.8948(최고, 2위 UDCNet-R24 0.8899 대비 +0.39%p), wFm 0.8634(2위), Sm 0.8799(2위) — MAE·maxF에서 최고, wFm·Sm은 근소하게 2위.
>
> #### 연산 효율 비교(Table 3)
> | 방법 | Params(M) | FLOPs(G) | Speed(fps) |
> |---|---|---|---|
> | GSANet24 | 49.46 | 25.270 | 28.16 |
> | TSCNet24 | 103.56 | 159.414 | 16.39 |
> | UDCNet-R24 | 72.33 | 140.433 | 19.94 |
> | UGNet26 | 38.52 | 22.483 | 12.33 |
> | **Ours** | 25.24 | 5.537 | **64.88** |
> - 파라미터·FLOPs가 비교 대상 중 최소 수준이면서 추론 속도(64.88fps)는 최고 — 정확도와 효율을 동시에 달성.
>
> #### 속성별 성능(Table 4, ORSI-4199 세부 시나리오)
> BSO(경계 모호), CS(복잡 장면), CSO(복잡 salient object), ISO(고립 객체), LCS(저대비), MSO/NSO(다중/비-salient object), OC(가림), SSO(소형 salient object) 9개 속성 중 CSO·ISO·OC·SSO에서 최고(적색), CS·LCS에서 2위(청색) — 저자는 이를 "고립 객체 탐지, 저대비 장면 처리, 가림 상황 완전성 보존, 소형 객체 정밀 localization에서의 우수한 일반화 능력"으로 해석.
>
> #### 메인 ablation(Table 5, ORSSD·EORSSD)
> | Base | SAI | MRFI | ORSSD MAE/maxF/wFm/Sm | EORSSD MAE/maxF/wFm/Sm |
> |---|---|---|---|---|
> | ✓ | | | 0.0076/0.9242/0.9116/0.9355 | 0.0059/0.892/0.8692/0.8943 |
> | ✓ | ✓ | | 0.0069/0.9273/0.9138/0.9369 | 0.0053/0.894/0.8754/0.8982 |
> | ✓ | | ✓ | 0.0066/0.9298/0.9166/0.942 | 0.0051/0.9042/0.887/0.9025 |
> | ✓ | ✓ | ✓ | **0.0063/0.9308/0.918/0.9434** | **0.0044/0.908/0.8919/0.9031** |
> - MRFI 단독 기여(base+MRFI)가 SAI 단독 기여(base+SAI)보다 전반적으로 큼 — 저자는 "MRFI가 SAI보다 더 유의미한 개선을 가져온다"고 명시.
>
> #### 손실 함수 ablation(Table 6)
> BCE 단독(maxF 0.9313으로 오히려 최고지만 다른 지표는 열세) < IOU 단독 < BCE+IOU(채택, 종합 최적) — IOU 손실 제거 시 성능 저하가 더 커, IOU가 상대적으로 더 중요한 역할.
>
> #### SAI/MRFI 세부 ablation(Table 7)
> - SAI 분할 수(SAI_1/2/8 vs 채택 SAI_4): 4분할이 최적, 8분할은 채널당 4개로 과소해 성능 저하.
> - Spatial attention을 channel attention으로 대체(SAI_CA): 파라미터·속도는 거의 동일하나 성능은 원본(spatial attention)보다 열세 — 공간 정보가 이 맥락에서 더 중요.
> - Channel shuffle 제거(SAI_wo_shuffle): 전 지표 하락 — 비인접 채널 결합의 실질적 기여 확인.
> - MRFI 중간 residual block 제거(MRFI_wo_mid), 전체 residual 제거(MRFI_wo_res): 둘 다 성능 저하, 특히 전체 residual 제거 시 하락폭이 큼.

# Discussion

### 이 아이디어의 잠재적 부작용
- SAI의 4분할 순차 처리가 그룹 수가 늘어날수록(8분할) 그룹당 채널 수가 급격히 줄어(32/8=4채널) 표현력이 부족해짐 → <mark style="background: #FF5582A6;">논문은 이를 Table 7에서 실험적으로만 확인할 뿐(8분할 성능 저하), 채널 수와 분할 수 사이의 최적 비율에 대한 일반 원리나 다른 backbone/해상도에서의 재현성은 다루지 않는다.</mark>
- MRFI의 element-wise 곱이 세 레벨 모두에서 낮은 응답을 보이는 극소 salient object의 신호를 과도하게 억제할 위험 → <mark style="background: #FF5582A6;">Table 4에서 SSO(소형 salient object) 속성은 최고 성능(적색)이라고 보고되지만, 이 곱셈 연산이 특히 작은 객체에서 신호 소실을 일으키는지에 대한 별도의 정성적 실패 사례 분석은 제시되지 않는다.</mark>

### 한계
- <mark style="background: #FF5582A6;">EORSSD의 maxF 지표는 2위와 완전히 동일한 값(0.9080)으로 보고되어(Table 1), 이 지표에서는 명확한 우위를 보이지 못했다 — 본문 서술("+0.52%")과 표의 실제 최고/2위 값 대응 관계가 다소 모호해 원문 표를 직접 재확인할 필요가 있다.</mark>
- <mark style="background: #FF5582A6;">SAI는 backbone의 가장 얕은(P1)과 가장 깊은(P4) 레벨에만 적용되고 중간 레벨(P2, P3)은 단순 conv+합산으로 처리된다 — 왜 중간 레벨에는 SAI를 적용하지 않았는지 설계 근거가 명시적으로 서술되지 않는다.</mark>
- 세 벤치마크(ORSSD, EORSSD, ORSI-4199) 모두 원격탐사 도메인 내에서만 검증되어, 이 설계가 일반 자연 이미지 SOD로 전이 가능한지는 다루지 않는다.
- Conclusion에서 저자가 별도의 향후 과제(예: 실시간 배포, 경량화 추가 검증)를 구체적으로 제시하지 않아, 이 논문이 스스로 인정하는 한계 목록이 명시적이지 않다.

### 생각할 점
- <mark style="background: #A6E3A1A6;">SAI의 "채널을 분할해 점진적으로 attention을 전파"하는 설계는, 이 위키의 dynamic query DETR 계열이 다루는 "전역 신호를 어떻게 인스턴스/토큰 단위로 분해하는가"라는 문제의식과 다른 도메인(SOD)에서 유사한 해법(분할 후 순차 처리)에 도달한 사례로 볼 수 있다.</mark>
- <mark style="background: #A6E3A1A6;">MRFI의 "곱셈으로 공통 정보 추출 + residual로 고유 정보 보존"이라는 이중 전략은, 이 위키의 [[ORFENet]] MRFAFEM(다중 receptive field 동적 가중합)과 유사한 상위 패턴(여러 소스를 결합하되 원본 정보도 잃지 않는다)을 공유하지만, MRFAFEM이 학습된 가중치로 가중합하는 반면 MRFI는 곱셈(공통점 추출)과 덧셈(residual)을 명시적으로 분리한다는 점이 다르다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">Table 5에서 MRFI 단독 기여가 SAI보다 크다는 관찰은, 이 위키의 소형 객체 탐지 논문들에서 반복되는 "feature 통합/융합 메커니즘의 기여가 개별 attention 정교화보다 크다"는 패턴과 궤를 같이한다 — SOD와 tiny object detection이라는 서로 다른 task임에도 유사한 패턴이 재발견된다는 점에서, 이 관찰이 task를 초월한 일반 원리일 가능성을 시사한다.</mark>
- <mark style="background: #A6E3A1A6;">이 논문은 이 위키에서 salient object detection과 remote sensing이 교차하는 첫 사례로, 기존 [[Uncertainty_Guided_Refinement]](자연 이미지 SOD, 불확실성 기반 반복 정제)와 비교하면 "불확실성 기반 반복 정제" vs "다중 레벨 feature의 명시적 공통 정보 추출"이라는 서로 다른 접근 축을 형성한다 — 두 접근을 결합해 원격탐사 SOD에 불확실성 기반 정제를 추가하는 방향도 고려할 만하다.</mark>

# 관련 개념
- [[Progressive_Grouped_Spatial_Attention_Interaction]] — 이 논문의 SAI 핵심 기여. Channel shuffle 후 그룹 분할해 순차적으로 spatial attention을 전파시키는 기법.
- [[Multiplicative_Residual_Saliency_Integration]] — 이 논문의 MRFI 핵심 기여. Element-wise 곱으로 다중 레벨의 공통 saliency 정보를 추출하고 다중 residual로 레벨별 고유 정보를 보존하는 기법.

# 관련 문서
- 비교: (아직 없음 — 이 위키에서 원격탐사 SOD 논문은 이번이 처음이라 비교 문서를 만들 근거가 부족. [[Uncertainty_Guided_Refinement]]와 함께 향후 2편 이상이 쌓이면 Salient Object Detection 비교 문서 신설을 검토)

# 읽어볼 만한 논문
- 참고문헌 기반: W. Wang, E. Xie, X. Li 외, "PVT v2: Improved baselines with pyramid vision transformer" (Comput. Vis. Media 2022) [60] — 이 논문의 backbone 원조. SAI가 어떤 feature 위에서 동작하는지 이해하려면 필수.
- 참고문헌 기반: G. Li, Z. Bai, Z. Liu, "Texture-semantic collaboration network for ORSI salient object detection" (IEEE Geosci. Remote Sens. Lett. 2024) [61] — 원격탐사 SOD에서 texture와 semantic의 협업을 다루는 유사 문제의식의 선행 연구.
- 참고문헌 기반: L. Sun, H. Liu, X. Wang 외, "Local-global information perception network for ORSI salient object detection" (IEEE Trans. Geosci. Remote Sens. 2025) [58] — 국소-전역 정보 인지라는 이 논문과 정확히 같은 문제의식을 다루는 최신 경쟁 연구, 비교 참고 가치가 큼.
- 자유 추천(검증 필요): Channel shuffle을 attention 메커니즘에 결합해 비인접 채널 간 정보 교환을 강화하는 다른 vision task 연구 — 검색 키워드: `channel shuffle grouped attention feature interaction efficient network design`. SAI의 channel shuffle+분할 전략이 ShuffleNet류의 효율화 아이디어와 어떻게 연결되는지 배경 이해에 도움될 것으로 예상.

---
**보안 참고**: PDF 전체를 확인했으며, 프롬프트 인젝션이나 지시문처럼 보이는 텍스트는 발견되지 않았다.
