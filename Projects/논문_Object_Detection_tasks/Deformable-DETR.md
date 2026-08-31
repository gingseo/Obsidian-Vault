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
source: "/Users/GyeongSeo/Workspace/논문_pdf/Small_Object_Detection/2021_ICLR_Deformable-DETR.pdf"
createdAt: "2026-08-24T03:03:00.000Z"
updatedAt: "2026-08-24T03:03:00.000Z"
---

Project: [[논문_Object_Detection|Object Detection]]
#paper #object-detection #transformer #deformable-attention #multi-scale-feature #sparse-attention #end-to-end

# 한 줄 요약
<mark style="background: #FFF3A3A6;">DETR의 attention이 모든 픽셀을 균일하게 바라봐 느린 수렴과 소형 객체 성능 열세를 낳는다는 문제를, 각 query가 reference point 주변의 소수 sampling point만 attend하는 (multi-scale) deformable attention module로 대체해, FPN 없이도 멀티스케일 feature를 직접 통합하면서 10배 적은 학습 epoch로 DETR을 능가하는 Deformable DETR.</mark>

# 문제 정의

### 기존 방법의 한계
- **극도로 느린 수렴**:
  DETR은 COCO 기준 500 epoch가 필요해 Faster R-CNN 대비 10~20배 느리게 수렴한다. 초기화 시 cross-attention이 feature map 전체에 거의 균일한 attention을 주는데, 학습이 끝날 무렵에는 매우 sparse한(특정 위치에 집중된) attention map으로 바뀌어야 하므로, 이 극단적인 변화를 학습하는 데 오랜 시간이 걸린다.
- **소형 객체에서 낮은 성능**:
  최신 detector는 고해상도 feature map으로 소형 객체를 더 잘 탐지하는데, DETR의 encoder self-attention은 픽셀 수에 대해 이차(quadratic) 복잡도를 가져 고해상도 feature map을 입력으로 쓰는 것 자체가 연산·메모리 측면에서 감당 불가능하다.
- **Multi-head attention의 근본적 취약점**:
  Query·key 수가 많을 때(이미지 도메인처럼 key가 픽셀 단위) attention weight `A_mqk ≈ 1/N_k`로 초기에는 거의 균일해지고, 학습 초기엔 gradient가 모호해 특정 key에 집중하는 학습이 느리다.

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — 사전 정의된 sparse attention pattern**
- Local window 기반[Liu et al. 2018a; Parmar et al. 2018; Child et al. 2019 외 다수]: attention 범위를 고정된 로컬 이웃으로 제한 — 연산량은 줄지만 전역 정보를 잃음.
- 특수 토큰에 전역 접근 허용[Beltagy et al. 2020; Ainslie et al. 2020; Zaheer et al. 2020]: 일부 sparse 패턴을 추가로 결합 — 여전히 패턴 자체는 고정(입력에 무관).
- 이미지 도메인에서 이 갈래의 방법들[Parmar et al. 2018; Child et al. 2019 등]은 이론적 복잡도는 줄어도, 표준 convolution과 동일 FLOPs 대비 실제로는 3배 이상 느리다고 저자들 스스로 인정 — 메모리 접근 패턴의 근본적 한계.

**갈래 2 — 데이터 기반 학습된 sparse attention**
- LSH 기반 해싱[Kitaev et al. 2020], k-means 기반 클러스터링[Roy et al. 2020], block permutation[Tay et al. 2020a]: query·key를 유사도로 그룹화해 sparse 연산 — 여전히 이미지 특화 설계는 아니며, 도입 사례가 드묾.

**갈래 3 — Low-rank 근사**
- Linear projection[Wang et al. 2020b], kernelization[Katharopoulos et al. 2020; Choromanski et al. 2020]: attention의 저랭크 성질을 활용해 연산량 감소 — 근사 오차가 발생하고 이미지 feature map 처리에 특화되지 않음.

**갈래 4 — Deformable convolution [[Deformable_Convolutional_Networks]]**
- 이미지 인식에서 sparse spatial location에 강력하고 효율적으로 attend하는 메커니즘이지만, <mark style="background: #FFF3A3A6;">DETR의 성공 핵심인 element 간 관계 모델링(relation modeling) 메커니즘이 없다.</mark>

**갭**: <mark style="background: #FFF3A3A6;">이미지 특화 효율적 attention(갈래 1) 대다수는 이론적 복잡도만 줄일 뿐 실제 구현에서는 표준 convolution보다 느리고, deformable convolution(갈래 4)은 빠르고 sparse하지만 relation modeling이 없다. Deformable convolution의 sparse spatial sampling과 Transformer의 relation modeling을 동시에 갖춘 attention 메커니즘은 없었다.</mark>

### 이 논문이 풀고자 하는 문제
1. DETR의 극도로 느린 수렴(500 epoch) 문제를 해결하는 것
2. 고해상도 feature map을 이차 복잡도 없이 처리해 소형 객체 탐지 성능을 개선하는 것
3. 별도 FPN 없이 멀티스케일 feature를 attention 메커니즘 자체로 통합하는 것

# 제안 방법

<mark style="background: #FFF3A3A6;">Deformable convolution처럼 각 query가 reference point 주변의 고정된 소수(K개) sampling point만 attend하도록 제한하는 deformable attention module을 제안한다 — attention을 "전체 feature map에 대한 pre-filtering 없는 전역 연산"에서 "reference point 주변 후보만 훑는 sparse 연산"으로 바꾸되, attention weight를 학습해 요소 간 관계 모델링 능력은 유지한다.</mark>

### ① Deformable Attention Module
- Query feature `z_q`와 2D reference point `p_q`가 주어지면, `z_q`를 선형 투영해 K개의 sampling offset `Δp_mqk`와 이에 대응하는 attention weight `A_mqk`(softmax로 정규화, 합=1)를 동시에 예측.
- 각 attention head에서 reference point + offset 위치의 feature를 bilinear interpolation으로 읽어와 attention weight로 가중합.
- Query가 만들어내는 K개의 key 후보가 전체 HW 픽셀이 아니라 reference point 주변 고정 개수(기본 K=4)이므로, feature map 크기와 무관하게 query당 연산량이 상수에 가까움.

> [!example]- 구현 디테일
> ```
> DeformAttn(zq, pq, x) = Σ_m Wm[ Σ_k Amqk · W'm x(pq + Δpmqk) ]
> ```
> 복잡도: `O(Nq·C² + min(HWC², Nq·K·C²))` — encoder(Nq=HW)에서는 `O(HWC²)`로 spatial size에 선형, decoder(Nq=N=300)에서는 `O(NKC²)`로 spatial size와 무관. `M=8`(attention head), `K=4`(sampling point)가 기본값. Offset·weight 예측 선형 투영의 weight는 0으로, bias는 8개 head가 원형으로 서로 다른 방향을 향하도록 초기화(K개 지점을 균등 분산).

<mark style="background: #FFF9D6A6;">Query별 key 후보를 소수로 제한하는 것은 deformable convolution의 sparse sampling 원리를 그대로 가져온 것이지만, offset과 함께 attention weight도 학습해 sampling point 간 상대적 중요도를 결정하므로("relation modeling"), deformable convolution에는 없던 요소 간 관계 모델링이 유지된다 — "문제 정의"에서 지적한 "빠르지만 relation modeling이 없는 deformable convolution"과 "relation modeling은 있지만 느린 DETR attention" 사이의 간극을 정확히 메운다.</mark>

### ② Multi-Scale Deformable Attention
- ResNet의 C3~C5 stage feature(FPN 없이 1×1 conv로 채널만 통일)와 C5에 stride-2 conv를 추가로 적용한 C6까지 총 4개 레벨을 encoder 입출력으로 동시 사용.
- 각 query가 L개 레벨 각각에서 K개씩(총 LK개) sampling point를 attend — 레벨 간 정보 교환이 attention 메커니즘 자체로 이뤄져 top-down FPN 구조가 필요 없어짐.
- Encoder는 self-attention을, decoder cross-attention은 이 multi-scale deformable attention으로 교체(decoder self-attention은 기존 Transformer attention 유지 — object query 수가 적어 연산 부담이 없으므로).

> [!example]- 구현 디테일
> ```
> MSDeformAttn(zq, p̂q, {x^l}) = Σ_m Wm[ ΣlΣk Amlqk · W'm x^l(φl(p̂q) + Δpmlqk) ]
> ```
> `φl(p̂q)`는 정규화 좌표를 l번째 레벨의 실제 feature map 좌표로 재조정하는 함수. `K=1, L=1, W'm=I`로 축소하면 원조 deformable convolution으로 퇴화 — 이 논문의 메커니즘이 deformable convolution의 정확한 일반화(generalization)임을 수식으로 명시.

<mark style="background: #FFF9D6A6;">"문제 정의"의 세 번째 문제(FPN 없이 멀티스케일 통합)를, attention의 sampling location을 레벨마다 별도로 두는 것만으로 해결한다 — Table 2 ablation에서 FPN을 추가로 결합해도 성능이 개선되지 않음을 실험적으로 확인해, cross-level 정보 교환이 attention 메커니즘 자체로 이미 충분함을 뒷받침한다.</mark>

### ③ Iterative Bounding Box Refinement & Two-Stage
- Iterative refinement: optical flow의 반복적 정제 방식[Teed & Deng 2020]에서 착안해, 각 decoder layer가 이전 layer의 예측 박스를 기준으로 상대 offset만 추가 예측 — reference point가 예측 박스와 강한 상관관계를 갖게 되어 학습이 더 안정적.
- Two-stage: encoder 단독으로 각 픽셀을 object query 취급해 region proposal을 직접 생성(NMS 없이 top-K 선별)한 뒤, 이를 decoder의 초기 reference point로 사용 — 기존 DETR의 "이미지와 무관하게 고정된 object query" 한계를 완화.

<mark style="background: #FFF9D6A6;">두 변형 모두 reference point를 예측 박스에 점점 더 가깝게 정렬시켜, decoder의 sampling location이 실제 객체 위치를 더 정확히 반영하도록 만든다 — deformable attention이 애초에 "reference point 주변만 본다"는 설계이므로, reference point 품질을 반복적으로 높이는 이 두 변형은 deformable attention의 이점을 극대화하는 자연스러운 후속 개선이다.</mark>

# 실험 결과

### 핵심 결과 (COCO 2017 val, 50 epoch 기준 vs DETR 500 epoch)
| 방법 | Epochs | AP | AP_S | 학습 GPU시간 |
|---|---|---|---|---|
| DETR | 500 | 42.0 | 20.5 | 2000 |
| Deformable DETR | 50 | 43.8 | 26.4 | 325 |
| Deformable DETR + iterative refinement | 50 | 45.4 | 26.8 | 325 |
| Deformable DETR ++ two-stage | 50 | 46.2 | 28.8 | 340 |

> [!note]- 세부 결과 및 Ablation
> #### DETR과 전면 비교 (Table 1, COCO 2017 val)
> | 방법 | Epochs | AP | AP50 | AP75 | AP_S | AP_M | AP_L | FLOPs | 추론 FPS |
> |---|---|---|---|---|---|---|---|---|---|
> | Faster R-CNN+FPN | 109 | 42.0 | 62.1 | 45.5 | 26.6 | 45.4 | 53.4 | 180G | 26 |
> | DETR | 500 | 42.0 | 62.4 | 44.2 | 20.5 | 45.8 | 61.1 | 86G | 28 |
> | DETR-DC5 | 500 | 43.3 | 63.1 | 45.9 | 22.5 | 47.3 | 61.1 | 187G | 12 |
> | DETR-DC5 | 50 | 35.3 | 55.7 | 36.8 | 15.2 | 37.5 | 53.6 | 187G | 12 |
> | Deformable DETR | 50 | 43.8 | 62.6 | 47.7 | 26.4 | 47.1 | 58.0 | 173G | 19 |
> - 동일 50 epoch에서 DETR-DC5는 AP 35.3에 그치는 반면 Deformable DETR은 43.8 — 수렴 속도 차이가 극명. AP_S는 DETR-500epoch(20.5) 대비 Deformable DETR-50epoch(26.4)이 이미 앞섬.
> - 추론 속도는 Faster R-CNN+FPN보다 25% 느리지만 DETR-DC5보다 1.6배 빠름 — DETR-DC5의 속도 문제가 Transformer attention의 큰 메모리 접근 때문이라고 분석.
>
> #### Deformable attention ablation (Table 2, COCO 2017 val)
> | MS inputs | MS attention | K | FPN | AP | AP_S |
> |---|---|---|---|---|---|
> | - | - | 1 | w/o | 39.7 | 21.2 |
> | ✓ | - | 1 | w/o | 41.4 | 24.1 |
> | ✓ | - | 4 | w/o | 42.3 | 24.8 |
> | ✓ | ✓ | 4 | w/o | 43.8 | 26.4 |
> | ✓ | ✓ | 4 | FPN 추가 | 43.8 | 26.5 |
> - Multi-scale input 자체가 +1.7 AP(특히 AP_S +2.9), sampling point 수 증가(1→4)가 +0.9 AP, multi-scale attention(레벨 간 정보 교환)이 추가로 +1.5 AP.
> - FPN을 추가해도 성능 향상 없음(43.8→43.8) — multi-scale deformable attention 자체가 이미 레벨 간 정보를 충분히 교환한다는 근거.
>
> #### SOTA 비교 (Table 3, COCO test-dev)
> | 방법 | Backbone | TTA | AP |
> |---|---|---|---|
> | FCOS | ResNeXt-101 | | 44.7 |
> | ATSS | ResNeXt-101+DCN | ✓ | 50.7 |
> | Deformable DETR | ResNet-50 | | 46.9 |
> | Deformable DETR | ResNeXt-101+DCN | | 50.1 |
> | Deformable DETR | ResNeXt-101+DCN | ✓ | **52.3** |
>
> #### 정성 분석 (Appendix A.5~A.6)
> - Gradient norm 시각화: DETR과 유사하게 박스 좌표(x,y,w,h) 예측은 객체의 극단점(extreme point, 경계)에 주로 의존. 다만 카테고리 예측(c)은 DETR과 달리 객체 내부 픽셀에도 의존 — 카테고리 분류에 극단점 외 내부 정보도 필요하다는 차이.
> - Sampling point 시각화(Fig. 6): encoder self-attention은 이미 개별 인스턴스를 분리(DETR과 유사), decoder cross-attention은 객체의 전체 전경(foreground) 영역에 걸쳐 sampling point가 분포 — extreme point뿐 아니라 내부 point도 카테고리 판별에 필요하다는 gradient 분석과 일치.

# Discussion

### 이 아이디어의 잠재적 부작용
- Sampling location이 무작위 접근(unordered memory access)을 유발 → <mark style="background: #FF5582A6;">논문 스스로 "deformable attention이 전통적 convolution보다는 여전히 약간 느리다"고 명시하며, 이 속도 손실의 원인을 unordered memory access로 지목한다 — deformable convolution 계보의 근본적 트레이드오프가 그대로 이어짐을 인정.</mark>
- Reference point 근방만 sampling하므로 reference point 자체가 잘못되면 관련 정보를 원천적으로 놓칠 위험 → <mark style="background: #FF5582A6;">이 위험을 완화하기 위해 iterative bounding box refinement·two-stage 변형을 별도로 제안했다는 것은, 기본 설계만으로는 이 위험이 충분히 낮지 않았음을 방증한다.</mark>

### 한계
- <mark style="background: #FF5582A6;">Two-stage 방식에서 첫 stage는 decoder 없이 encoder-only로 region proposal을 생성하는데, self-attention의 이차 복잡도 때문에 "각 픽셀을 object query로 직접 사용"할 수 없다고 저자가 명시 — 이는 근본적 해결이 아니라 decoder를 제거해 우회한 것.</mark>
- <mark style="background: #FF5582A6;">M(head)=8, K(sampling point)=4 같은 핵심 하이퍼파라미터의 최적값 탐색 범위가 제한적(Table 2도 K=1/4 두 값만 비교) — 더 넓은 탐색이나 데이터셋별 민감도 분석은 없음.</mark>
- ResNeXt-101+DCN(즉 deformable convolution을 백본에도 결합) 조합이 최고 성능(52.3 AP)을 내는데, 이는 attention만으로는 부족해 여전히 별도 deformable convolution 모듈이 백본 단에 필요함을 시사 — 두 메커니즘이 상호 대체가 아니라 보완 관계임을 논문이 암묵적으로 인정.

### 생각할 점
- <mark style="background: #A6E3A1A6;">이 논문은 [[Deformable_Convolutional_Networks]]의 offset 메커니즘을 "CNN의 고정 grid"에서 "attention의 전역 key 집합"으로 옮긴 사례로, [[Deformable_Sampling_Offset]] 개념이 처음 다른 아키텍처(Transformer)로 이식된 것이다 — Appendix A.1에서 `K=1,L=1,W'm=I`일 때 정확히 deformable convolution으로 퇴화함을 수식으로 증명해, 이 계승 관계가 저자들 스스로도 명시적으로 인식하고 있던 설계임을 보여준다.</mark>
- <mark style="background: #A6E3A1A6;">Table 2에서 "multi-scale input 자체"(+1.7 AP)가 "sampling point 수 4배 증가"(+0.9 AP)보다 기여가 크다는 점은, 이 위키의 [[ORFENet]]에서 관찰된 "다중 소스를 쓴다는 것 자체의 기여가 정교화보다 크다"는 패턴과 다시 한번 일치한다 — 서로 다른 아키텍처(FCOS 기반 vs DETR 기반)에서 반복 관찰되는 현상이라는 점에서 근거가 강화된다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">이 논문이 처음 지적한 AP_S 개선(20.5→26.4)은 여전히 절대값 자체는 낮은 편이다 — 이후 처리할 DQ-DETR·Density-Aware DETR 등 "DETR + dynamic query" 계열이 정확히 이 지점(소형/타이니 객체를 위한 query 자체의 품질)을 더 파고드는 것으로 보이며, deformable attention의 "reference point 주변만 본다"는 sparse sampling과 이후 계열의 "query 자체를 동적으로 생성/우선순위화한다"는 접근이 어떻게 다른 층위에서 상호보완적인지 처리하며 비교할 필요가 있다.</mark>
- Two-stage의 "각 픽셀=object query"라는 초기 proposal 생성 방식은, 이 위키의 [[QueryDet]]이 쓰는 "저해상도 예측으로 고해상도 위치를 좁힌다"는 coarse-to-fine query 방식과 목적(밀집 픽셀에서 후보를 어떻게 좁히는가)이 유사해 보인다 — 다만 QueryDet은 CNN 계열, 이 논문은 attention 계열이라는 아키텍처 층위의 차이가 있다.

# 관련 개념
- [[Deformable_Sampling_Offset]] — [[Deformable_Convolutional_Networks]]가 CNN에 도입한 개념을 Transformer attention의 sampling location으로 확장한 첫 사례. "등장 논문"에 이번 논문 추가 갱신함.

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]] — DETR·Deformable Convolutional Networks와 함께 foundational 계열로 별도 취급(비교표 대상 아님). 이후 dynamic query DETR 계열(DQ-DETR 등) 다수의 직접 baseline.

# 읽어볼 만한 논문
- 참고문헌 기반: N. Carion, F. Massa, G. Synnaeve, N. Usunier, A. Kirillov, S. Zagoruyko, "End-to-end object detection with transformers" (DETR, ECCV 2020) — 이미 이 위키의 [[DETR]] 노트로 존재. 이 논문이 직접 개선하는 원조.
- 참고문헌 기반: Z. Teed, J. Deng, "RAFT: Recurrent all-pairs field transforms for optical flow" (ECCV 2020) — iterative bounding box refinement가 아이디어를 차용한 optical flow의 반복적 정제 기법. 반복 정제 설계의 배경 이해에 도움.
- 참고문헌 기반: X. Zhu, H. Hu, S. Lin, J. Dai, "Deformable ConvNets v2: More deformable, better results" (CVPR 2019) — 이 논문의 Table 3에서 최고 성능(52.3 AP)에 쓰인 DCN의 개선판. Deformable convolution 계보의 최신 버전.
- 자유 추천(검증 필요): Deformable attention을 vision task 전반(segmentation, tracking 등)으로 확장한 후속 연구 — 검색 키워드: `deformable attention module extension segmentation tracking 2022 2023`. Deformable DETR의 attention 메커니즘이 detection 외 다른 dense prediction task에 어떻게 재사용됐는지 파악하는 데 유용.

---
**보안 참고**: PDF 전체를 확인했으며, 프롬프트 인젝션이나 지시문처럼 보이는 텍스트는 발견되지 않았다.
