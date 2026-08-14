---
title: "QueryDet: Cascaded Sparse Query for Accelerating High-Resolution Small Object Detection"
authors: [Chenhongyi Yang, Zehao Huang, Naiyan Wang]
year: 2022
venue: "CVPR"
jcr_quartile: Q1
task: [small-object-detection]
direction: [novel-approach, foundational]
tags: [paper, small-object-detection, sparse-convolution, inference-acceleration, feature-pyramid, query-mechanism]
status: read
user_read: false
added: 2026-08-05
source: "raw/small-object-detection/2022_CVPR_QueryDet.pdf"
created: 2026-08-05
---

# 한 줄 요약
<mark style="background: #FFF3A3A6;">저해상도 feature map에서 소형 객체가 있을 만한 대략적 위치를 먼저 예측(query)하고, 그 위치 주변에만 고해상도 feature map에서 sparse convolution으로 detection head를 적용해, 정확도 손실 없이 고해상도 feature의 연산 비용을 99% 가까이 줄이는 Cascade Sparse Query(CSQ) 프레임워크.</mark>

원문 요약(Abstract/Introduction/Main Contribution/Conclusion 번역): [[QueryDet-source]]

# 문제 정의

### 기존 방법의 한계
- **고해상도 feature의 연산 비용 폭증**:
  FPN에 고해상도 레벨(P2)을 추가하면 RetinaNet 기준 detection head 연산량(FLOPs)이 300% 증가하고, 2080Ti GPU 기준 추론 속도가 13.6 FPS에서 4.85 FPS로 급락한다. P3까지만 써도 이미 전체 연산의 43%, P2까지 추가하면 74%를 차지한다.
- **Dense 연산의 공간적 낭비**:
  소형 객체는 이미지·feature map 상에서 극히 일부 영역에만 sparse하게 분포하는데, 기존 detection head는 고해상도 feature map 전체에 dense하게 동일 연산을 적용해 대부분의 연산이 배경(background) 영역에 낭비된다.

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — 입력/feature 해상도를 높여 소형 객체 성능 개선**
- FPN [26], SSD [29], DSSD [10], HyperNet [22]: 다양한 CNN 레이어의 multi-scale feature를 재사용해 고해상도 정보를 확보 — 여전히 고해상도 레벨에서의 dense 연산 비용 자체는 해결하지 못함.
- Scale-aware Trident Networks [25]: receptive field가 객체 크기와 mismatch되는 문제 지적 — 연산 가속과는 무관한 문제의식.

**갈래 2 — 공간적 중복성(spatial redundancy)을 활용한 sparse 연산**
- PerforatedCNN [9], Dynamic Convolution [47], SACT [8], SBNet [38]: Gumbel-Softmax나 별도 gating network로 sparse mask를 학습해 연산을 줄임 — 대부분 결정론적 샘플링이나 추가 sparsity loss/decision network가 필요해 학습이 복잡.
- AutoFocus [33]: coarse scale에서 관심 영역을 예측해 crop 후 고해상도로 재처리 — image pyramid 상에서 동작해 backbone 자체를 다시 돌려야 하는 중복 연산이 남음.
- PointRend [19]: 불확실한 위치를 골라 고해상도 segmentation을 sparse하게 계산 — MLP 기반 point-wise 분류라 단일 위치 feature만 쓰고, object detection이 필요로 하는 주변 context 정보를 반영하기 어려움.

**갭**: <mark style="background: #FFF3A3A6;">기존 sparse 연산 기법들은 별도의 gating network·복잡한 학습 목표를 필요로 하거나(갈래 2 전반), feature pyramid가 아닌 image pyramid 상에서 동작해(AutoFocus) backbone 연산까지 반복되거나, point-wise 처리로 context 정보를 놓친다(PointRend). Feature pyramid의 "레벨 간 강한 구조적 상관관계"(저해상도 예측이 고해상도 위치를 가리킬 수 있다는 성질) 자체를 직접 활용해, 단순한 GT 박스 supervision만으로 sparse 연산을 유도하는 방법은 없었다.</mark>

### 이 논문이 풀고자 하는 문제
1. 고해상도 feature map의 어느 위치에 소형 객체가 있을지 저비용으로 미리 알아내는 방법
2. 알아낸 위치에만 detection head 연산을 sparse하게 적용해 실제 속도 이득으로 연결하는 방법

# 제안 방법

<mark style="background: #FFF3A3A6;">Feature pyramid는 레벨 간 강한 구조적 상관관계를 가진다는 관찰에서 출발한다 — 저해상도 feature map만으로 정확한 박스 회귀는 어렵지만, "이 근방에 소형 객체가 있는지" 정도는 높은 신뢰도로 추론할 수 있다. 이 대략적 위치(query key)를 이용해 고해상도 feature map(query value)에서 해당 위치 주변만 sparse convolution으로 계산한다.</mark>

### ① Query Head + Cascade Sparse Query (CSQ)
- 분류·회귀 head와 병렬로 각 레벨 `P_l`에 query head를 추가, 그리드별로 "이 위치에 소형 객체가 있을 확률" heatmap `V_l`을 출력.
- 레벨별 소형 객체 기준 `s_l`(해당 레벨의 최소 anchor scale, anchor-free는 최소 regression range)보다 작은 객체의 중심과의 거리가 `s_l` 미만인 위치를 positive로 학습(Focal Loss).
- 추론 시 임계값 σ를 넘는 위치를 query key로 선택, 한 단계 아래 고해상도 레벨 `P_{l-1}`의 4개 최근접 위치로 매핑해 key position을 얻음.
- 이 key position들만 골라 sparse tensor(query value feature)를 구성하고, 4-conv dense head의 가중치를 그대로 sparse convolution(spconv) 커널로 사용해 해당 위치에서만 연산.
- 이 과정을 레벨마다 반복(cascade)해 `P_{l-2}`의 query는 오직 `P_{l-1}`의 key position에서만 생성 — 단일 레벨에서 직접 매핑할 때 레벨이 내려갈수록 key 수가 지수적으로 늘어나는 문제를 피함.

> [!example]- 구현 디테일
> ```
> # 레벨 l의 query 위치를 l-1의 4개 최근접 위치로 매핑
> {k_{l-1}^o} = {(2x_l^o + i, 2y_l^o + j) | i,j ∈ {0,1}}
>
> # 손실: 분류 + 회귀 + query(Focal Loss)를 레벨별 가중합
> L_l = L_FL(U_l, U_l*) + L_r(R_l, R_l*) + L_FL(V_l, V_l*)
> L_all = Σ_l β_l * L_l   (β_l은 P2→P7로 1에서 3까지 선형 증가)
> ```
> - Query threshold σ = 0.15, query 시작 레벨은 P4가 최적(P5/P6에서 시작하면 저해상도 연산 자체가 이미 빠르고 sparse tensor 구성 비용이 이를 못 넘음; P2/P3에서 시작하면 소형 객체 판별이 저해상도에서 어려움).
> - Context: query 위치 주변 5×5 패치까지 함께 sparse head에 포함해야 정확도 손실이 없음(1×1은 부족, 11×11 이상은 속도 이득 감소).
> - β_l 재조정이 필수적인 이유: P2를 추가하면 학습 샘플 수 분포가 급변(P2의 샘플 수가 P3~P7 전체보다 많아짐)해 재조정 없이는 AP가 오히려 1.34 하락.

<mark style="background: #FFF9D6A6;">이 설계는 "문제 정의"의 두 문제를 동시에 해결한다 — dense 연산이 낭비되는 배경 영역을 애초에 계산하지 않으므로 연산 비용이 소형 객체가 실제로 존재하는 sparse한 영역에 비례하게 되고(고해상도 P2/P3 연산량 74%→약 1%), cascade 구조 덕분에 이 sparse화가 레벨마다 반복 적용되어도 key 수가 폭증하지 않는다.</mark>

# 실험 결과

### 핵심 결과
| 벤치마크 | 지표 | Before(고해상도 dense) | After(CSQ 적용) |
|---|---|---|---|
| COCO (RetinaNet) | AP / APS / FPS | 38.53 / 24.64 / 4.85 | 38.36 / 24.33 / 14.88 |
| VisDrone (RetinaNet) | AP / FPS | 28.35 / 1.16 | 28.32 / 2.75 |

> [!note]- 세부 결과 및 Ablation
> #### Ablation (COCO mini-val, RetinaNet 기준)
> | HR | RB(재조정) | QH(query head) | CSQ | AP | APS | FPS |
> |---|---|---|---|---|---|---|
> | - | - | - | - | 37.46 | 22.64 | 13.60 |
> | ✓ | - | - | - | 36.10 | 21.94 | 4.83 |
> | ✓ | ✓ | - | - | 38.11 | 23.06 | 4.83 |
> | ✓ | ✓ | ✓ | - | 38.53 | 24.64 | 4.85 |
> | ✓ | ✓ | ✓ | ✓ | 38.36 | 24.33 | **14.88** |
>
> #### 다른 아키텍처로의 일반화
> - FCOS(anchor-free) 적용: AP 38.37→40.05(No CSQ)→39.49(CSQ), 고해상도 속도 1.8배 향상.
> - Faster R-CNN(2-stage, RPN에 적용): AP 38.47→38.20, FPS 17.57→19.03.
> - Light-weight backbone(MobileNetV2, ShuffleNetV2): 고해상도 검출 속도 각각 평균 4.1배, 3.8배 향상 — backbone 연산 비중이 작을수록 CSQ의 상대적 가속 효과가 커짐.
>
> #### Query 방식 비교
> - Crop Query(AutoFocus 유사), Complete Convolution Query 대비 제안한 CSQ가 가장 빠름(14.88 FPS vs 10.49/8.73 FPS), AP는 거의 동일.
> - Context 패치 크기: 5×5가 속도-정확도 균형점, 11×11 이상은 AP 이득 대비 속도 손실이 큼.
>
> #### Failure case
> - Query head가 위치를 맞춰도 detection head가 국소화에 실패하는 경우.
> - 대형 객체 위치가 잘못 활성화되어 불필요한 sparse 연산이 발생, 속도가 저하되는 경우.

# Discussion

### 이 아이디어의 잠재적 부작용
- Query head의 recall 실패 시 소형 객체를 원천적으로 놓칠 위험 → (빨강 하이라이트) <mark style="background: #FF5582A6;">논문은 매우 낮은 threshold(0.05)에서도 큰 속도 이득이 나는 것으로 이 위험을 완화했다고 주장하지만, 완전히 해소된 것은 아니며 여전히 query head 성능에 상한이 걸린다.</mark>
- 대형 객체 위치의 오탐(false positive) 활성화로 인한 속도 저하 → <mark style="background: #FF5582A6;">논문이 failure case로 명시했을 뿐 별도 해결책은 제시하지 않았다.</mark>

### 한계
- <mark style="background: #FF5582A6;">Query head가 정확한 위치를 찾아도 detection head 자체가 국소화(localization)에 실패하는 case가 존재(VisDrone 실패 사례).</mark>
- <mark style="background: #FF5582A6;">Query 시작 레벨, threshold σ, context 패치 크기 등 하이퍼파라미터에 성능-속도 트레이드오프가 민감하게 반응 — 데이터셋/해상도별 재튜닝이 필요해 보인다.</mark>

### 생각할 점
- <mark style="background: #A6E3A1A6;">"저해상도 예측으로 고해상도 연산 위치를 좁힌다"는 아이디어는 본질적으로 coarse-to-fine 접근이며, sr-tod/rs-tod 등 feature 강화 계열과는 직교적 — 두 성질(어디를 볼지 vs 어떻게 강화할지)을 함께 쓸 여지가 있다.</mark>
- <mark style="background: #A6E3A1A6;">Sparse convolution으로 dense head 가중치를 그대로 재사용하는 방식은 추가 파라미터 없이 기존 detector에 이식 가능하다는 점에서 실용적 — 저자들도 실제로 RetinaNet/FCOS/Faster R-CNN 세 아키텍처에 동일하게 적용해 일반성을 보였다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">[[Unc-SOD]]처럼 "어떤 prior를 positive로 볼지"를 동적으로 정하는 label assignment 계열과 CSQ의 query 메커니즘을 결합하면, sampling 품질과 연산 효율을 동시에 개선할 수 있을 것으로 보임.</mark>
- <mark style="background: #A6E3A1A6;">원격탐사/드론뷰 논문들([[RS-TOD]], [[FANet]])은 대부분 고해상도 입력에서 연산 비용 문제를 다루지 않는데, CSQ의 sparse 연산 아이디어를 결합하면 정확도 손실 없이 실시간성을 확보할 잠재력이 있다.</mark>

# 관련 개념
- [[cascade-sparse-query]] — 이 논문의 핵심 기여

# 관련 문서
- 비교: [[small-object-detection-approaches]]

# 읽어볼 만한 논문
- 참고문헌 기반: A. Kirillov, Y. Wu, K. He, R. Girshick, "PointRend: Image segmentation as rendering" (CVPR 2020) [19] — QueryDet과 유사하게 sparse한 위치만 골라 고해상도 예측을 하지만 point-wise MLP를 쓴다는 점에서 대조되는 접근. Sparse 연산 설계의 대안 이해에 도움.
- 참고문헌 기반: M. Najibi, B. Singh, L. S. Davis, "AutoFocus: Efficient multi-scale inference" (ICCV 2019) [33] — QueryDet이 직접 비교하는 가장 유사한 선행 연구(image pyramid 기반 coarse-to-fine). CSQ가 이를 feature pyramid로 옮겨 backbone 중복 연산을 없앤 차별점을 이해하는 데 필수.
- 참고문헌 기반: B. Zhu, J. Wang, Z. Jiang, F. Zong, S. Liu, Z. Li, J. Sun, "AutoAssign: Differentiable label assignment for dense object detection" (arXiv 2020) [36] — QueryDet의 query head 학습(거리 기반 positive 정의)과 비교할 만한 differentiable label assignment 접근.
- 자유 추천(검증 필요): Sparse convolution 기반 3D object detection(LiDAR) 연구 — 저자들이 Conclusion에서 향후 3D 확장을 명시적으로 언급함. 검색 키워드: `sparse convolution 3D object detection LiDAR point cloud CVPR`
