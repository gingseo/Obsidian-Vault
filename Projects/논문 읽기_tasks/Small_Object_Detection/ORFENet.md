---
pm-task: true
projectId: "paperwiki-reading-unified"
parentId:
id: "t-orfenet-fxrkkwmysi"
title: "Tiny Object Detection in Remote Sensing Images Based on Object Reconstruction and Multiple Receptive Field Adaptive Feature Enhancement"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-08-19"
due:
progress: 0
assignees: []
tags: []
subtaskIds: []
dependencies: []
year: 2024
venue: "IEEE Transactions on Geoscience and Remote Sensing (TGRS)"
jcr_quartile: Q1
task: [small-object-detection]
direction: [improvement]
paper_tags: [paper, small-object-detection, remote-sensing, self-supervision, multi-receptive-field, feature-enhancement, fcos]
source: "Projects/논문 읽기_pdf/Small_Object_Detection/2024_TGRS_ORFENet.pdf"
createdAt: "2026-08-19T00:00:00.000Z"
updatedAt: "2026-08-19T00:00:00.000Z"
---

#paper #small-object-detection #remote-sensing #self-supervision #multi-receptive-field #feature-enhancement #fcos

# 한 줄 요약
<mark style="background: #FFF3A3A6;">FCOS(P2) 기반 tiny object detector에, 학습 시에만 존재하며 추론 시 제거 가능한 Object Reconstruction Branch(ORB)로 tiny object 영역의 정보 손실을 억제하고, fine-grained/close-range/distant-context 세 가지 receptive field 특징을 동적 가중치로 결합하는 Multiple Receptive Field Adaptive Feature Enhancement Module(MRFAFEM)을 결합해, 추가 추론 비용 없이 AI-TODv2에서 AP 24.8%를 달성한 원격탐사 tiny object detector ORFENet.</mark>


# 문제 정의

### 기존 방법의 한계
- **딥 네트워크의 정보 손실**:
  Tiny object는 원본에서도 픽셀 수가 적은데, backbone의 반복적 다운샘플링을 거치며 이미 약한 feature 신호가 한층 더 손실된다.
- **다중 receptive field 특징의 정적 활용**:
  Tiny object 판별에는 주변 맥락 정보가 중요하다(예: 저장탱크 하나만 보면 판별이 어렵지만 주변에 비슷한 탱크가 많으면 판별이 쉬워짐). 그런데 어떤 tiny object는 좁은 local receptive field만으로 충분하고 어떤 것은 더 넓은 range의 context가 필요해, 요구되는 receptive field 크기가 상황마다 다르다. 기존 방법은 이 receptive field별 중요도를 동적으로 조정하지 않는다.

### 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Data augmentation**
- Kisantal et al. [38]: tiny object를 여러 번 copy-paste해 학습 샘플 수 증가.
- Chen et al. [39]: context를 고려한 adaptive resampling으로 scale/배경 mismatch 완화.
- 공통 한계: 샘플 수만 늘릴 뿐 feature 자체의 정보 손실 문제는 그대로.

**갈래 2 — Multiscale learning**
- FPN [13], NAS-FPN [40], BiFPN(단, 원문은 명시적 인용 없이 서술) [41,42,43,44], TridentNet [45]: 서로 다른 깊이·receptive field의 feature를 융합하는 구조 개선.
- 공통 한계: feature hierarchy 구조 자체를 재설계할 뿐, 각 receptive field 브랜치의 중요도를 입력에 따라 동적으로 조정하지는 않는다.

**갈래 3 — Feature enhancement**
- Haris et al. [46]: super-resolution과 detection을 end-to-end 공동 학습.
- Li et al. [47], Bai et al. [48]: GAN 기반 generator-discriminator로 고해상도 feature 생성.
- 공통 한계: 생성(super-resolution/GAN) 기반이라 존재하지 않는 디테일을 새로 만들어내는 리스크가 있고, 별도의 고해상도 GT나 복잡한 adversarial 학습이 필요.

**갈래 4 — Improved sample assignment**
- ATSS [49], Xu et al.(dot distance) [14], Xu et al.(NWD-RKA) [16], Xu et al.(receptive-field 기반 RFLA) [17]: positive/negative 샘플 할당 기준 개선.
- 공통 한계: 할당 전략만 다룰 뿐, feature representation 자체의 정보 손실이나 receptive field 활용 문제는 건드리지 않음.

**갭**: <mark style="background: #FFF3A3A6;">선행 연구들은 정보 손실 문제(대부분 구조 재설계나 생성 기반 복원으로 간접 대응)와 다중 receptive field 특징의 동적 관리 문제를 각각 따로 다루거나 아예 다루지 않았다. 이 논문은 (1) 생성 없이 pixel-level foreground/background 분류만으로 정보 손실을 억제하는 auxiliary branch와, (2) receptive field별 동적 가중치 결합을 하나의 프레임워크에서 동시에 다룬다.</mark>

### 이 논문이 풀고자 하는 문제
1. 딥 네트워크의 다운샘플링 과정에서 발생하는 tiny object의 정보 손실을 추가 추론 비용 없이 억제하는 것
2. 서로 다른 receptive field의 특징이 tiny object 판별에 기여하는 정도가 상황마다 다르다는 점을 반영해, 이 특징들을 동적으로 결합하는 것

# 제안 방법

<mark style="background: #FFF3A3A6;">FCOS(P2) baseline에 두 모듈을 추가한다 — Object Reconstruction Branch(ORB)는 tiny object 영역에 대한 foreground/background pixel-level 재구성을 학습 목표로 삼아 feature map이 tiny object 정보를 보존하도록 강제하는 auxiliary task이며, 학습 후에는 제거해 추론 비용이 전혀 없다. Multiple Receptive Field Adaptive Feature Enhancement Module(MRFAFEM)은 세 가지 receptive field 브랜치의 출력을 입력 feature에 따라 동적으로 가중합해 tiny object 검출에 유리한 조합을 자동으로 찾는다.</mark>

### ① Object Reconstruction Branch (ORB)
- Head(3×3 conv) → body(N개의 [conv, BatchNorm2d, ReLU, 업샘플링(bilinear)] 블록) → end(3×3 conv, 출력 채널 2) 구조로, FPN의 P2(또는 P3) feature map을 입력받아 원본 이미지 크기의 foreground/background 분류 맵을 예측.
- 학습 target은 GT 박스 위치를 1, 나머지를 0으로 채운 이진 마스크(원본 이미지 크기) — 원본 픽셀을 그대로 복원하는 것이 아니라 tiny object 영역의 위치만 재구성하는 단순한 목표.
- Cross-entropy loss(`Loss_or`)로만 학습되며, 이 branch는 학습이 끝나면 완전히 제거 가능 — 추론 시 파라미터·연산량 변화 없음(Table V, ORB 단독 적용 시 Params/FLOPs 불변).
- N은 입력 feature map 해상도에 따라 결정(P2 입력이면 N=2, P3 입력이면 N=3)해 항상 원본 이미지 해상도로 업샘플링되도록 함.

> [!example]- 구현 디테일
> ```
> Y = end(body(head(X)))
> Loss_or = (1/(H×W)) Σ CE(pred, gt)   # gt: GT 박스 영역=1, 그 외=0인 마스크
> Loss_det = Loss_reg + Loss_cls + Loss_cent   # FCOS 표준 손실(GIoU + Focal + CE)
> Loss = Loss_det + λ·Loss_or   # λ=10(기본값, ablation으로 결정)
> ```
> λ ablation(Table IV): λ=0(ORB 미포함) AP 17.3 → λ=10일 때 AP 18.5(최고) → λ=20에서는 reconstruction loss 비중 과다로 오히려 하락(18.4).

<mark style="background: #FFF9D6A6;">ORB는 tiny object detection head와 "multitask learning" 관계에 있다 — 두 과제 모두 tiny object 영역에 대한 판별을 목표로 하므로 상호 보강 효과가 있고, pixel-level supervision을 중간 feature layer에 추가한다는 점에서 "이종 심층 감독(heterogeneous deep supervision)"으로도 볼 수 있다. 정보 손실 문제 자체를 별도 생성 없이 "이 영역이 tiny object인지" 예측을 강제하는 제약으로 우회하므로, 문제 정의에서 지적한 "생성 기반 방법의 spurious detail" 위험 없이 feature map이 tiny object 정보를 최소한으로 보존하도록 유도한다.</mark>

### ② MRFAFEM (Multiple Receptive Field Adaptive Feature Enhancement Module)
- 입력 feature `F`를 receptive field 크기가 다른 세 브랜치에 병렬로 통과: fine-grained branch(표준 3×3 conv), close-range context branch(7×7 depthwise conv 2회 연속), distant context branch(21×21 depthwise conv 2회 연속).
- 세 브랜치 출력을 채널 방향으로 concat → group conv(3×3) → 채널 합산 → Softmax로 동적 가중치 `W={W1,W2,W3}` 산출.
- 가중합 후 1×1 conv를 거쳐 잔차 연결로 원본 `F`에 더해 최종 출력 `F+`.

> [!example]- 구현 디테일
> ```
> F1 = Conv3(F)
> F2 = DW7(DW7(F))
> F3 = DW21(DW21(F))
> F+ = F + Conv1(W1·F1 + W2·F2 + W3·F3)
> W  = Softmax(sum(GC3(cat(F1,F2,F3))))
> ```
> Ablation(Table II): close-range branch kernel 7, distant branch kernel 21이 최적(5/9로 바꾸면 AP 소폭 하락, distant를 17/25로 바꿔도 21이 최적). Dynamic weight 유무 비교(Table III): 동적 가중치 없이 단순 합만 해도 AP 17.3→18.2(+0.9), 동적 가중치를 추가하면 18.4(+0.2 추가) — 즉 "다중 receptive field를 쓴다"는 것 자체의 기여가 크고, "동적으로 가중한다"는 것은 그 위에 소폭의 추가 이득.

<mark style="background: #FFF9D6A6;">Tiny object마다 필요한 context 범위(receptive field)가 다르다는 관찰을, 세 receptive field 브랜치를 항상 동일 비중으로 섞는 대신 입력 feature에 따라 Softmax 가중치로 조정함으로써 직접 반영한다 — "문제 정의"에서 지적한 "receptive field별 중요도가 상황마다 다른데 기존 방법은 이를 반영하지 않는다"는 갭에 정확히 대응하는 설계다.</mark>

# 실험 결과

### 핵심 결과 (AI-TODv2, ResNet-50-FPN, FCOS(P2) baseline)
| 벤치마크 | 지표 | Before(FCOS(P2)) | After(ORFENet) |
|---|---|---|---|
| AI-TODv2 | AP / AP50 | 17.3 / 39.9 | 18.9 / 44.4 (36 epoch 학습 시 24.8 / 55.4) |
| LEVIR-Ship | AP50 | — | 83.3(SOTA, 전 비교 방법 중 최고) |

> [!note]- 세부 결과 및 Ablation
> #### Baseline별 개선 (Table I, 12 epoch 기준)
> | Baseline | ORB | MRFAFEM | AP | AP50 | AP75 | Params(M) | FLOPs(G) |
> |---|---|---|---|---|---|---|---|
> | FCOS | - | - | 13.2 | 32.1 | 8.4 | 31.85 | 123.16 |
> | FCOS | ✓ | - | 14.6 | 36.4 | 9.1 | 31.85 | 123.16 |
> | FCOS | - | ✓ | 13.9 | 33.6 | 9.0 | 32.7 | 131.69 |
> | FCOS | ✓ | ✓ | **15.1** | **37.6** | **9.8** | 32.7 | 131.69 |
> | FCOS(P2) | - | - | 17.3 | 39.9 | 12.2 | 31.92 | 339.24 |
> | FCOS(P2) | ✓ | - | 18.5 | 44.1 | 12.3 | 31.92 | 339.24 |
> | FCOS(P2) | - | ✓ | 18.4 | 43.3 | 12.7 | 32.77 | 373.34 |
> | FCOS(P2) | ✓ | ✓ | **18.9** | **44.4** | **12.7** | 32.77 | 373.34 |
>
> - ORB는 Params/FLOPs 변화 없이(추론 시 제거) AP를 개선(FCOS 기준 +1.4, FCOS(P2) 기준 +1.2) — "추가 연산 비용 없는 개선"이라는 핵심 주장의 근거.
> - MRFAFEM은 Params/FLOPs가 소폭 증가(FCOS(P2) 기준 31.92M→32.77M, 339.24G→373.34G)하며 AP +1.1.
>
> #### SOTA 비교 (AI-TODv2 test, Table V·VI)
> | 방법 | Backbone | AP | AP50 | AP75 | AP_vt | AP_t | AP_s | AP_m |
> |---|---|---|---|---|---|---|---|---|
> | DetectoRS [37] (2-stage 최고) | ResNet-50-FPN | 16.1 | 35.5 | 12.5 | 0.1 | 12.6 | 28.3 | 40.0 |
> | FSANet [61] (1-stage 최고) | ResNet-50-FPN | 17.6 | 45.0 | 10.5 | 5.4 | 15.8 | 27.8 | 33.1 |
> | ORFENet (12 epoch) | ResNet-50-FPN | 18.9 | 44.4 | 12.7 | 6.9 | 18.4 | 23.4 | 30.3 |
> | ORFENet* (36 epoch) | ResNet-50-FPN | **24.8** | **55.4** | **18.2** | **9.7** | **24.4** | **28.7** | 35.1 |
>
> - 클래스별(Table VI): ORFENet*(36 epoch)이 BR/SH/SP/VE/PE/WM 6개 클래스에서 최고 성능. AI(airplane)는 DetectoRS(28.5)가 근소 우세, ST(storage tank)는 Cascade-RCNN w/NWD-RKA(36.9)가 우세.
> - DetectoRS 대비: AP +8.7%p, AP50 +19.9%p, AP75 +5.7%p, AP_vt +9.6%p, AP_t +11.8%p, AP_s +0.4%p.
> - FSANet 대비: AP +7.2%p, AP50 +10.4%p, AP75 +7.7%p, AP_vt +4.3%p, AP_t +8.6%p, AP_s +5.8%p.
>
> #### LEVIR-Ship (Table VII)
> | 방법 | AP50 | 방법 | AP50 |
> |---|---|---|---|
> | RetinaNet | 74.9 | FCOS | 75.5 |
> | Faster R-CNN | 70.8 | DRENet [19] | 82.4 |
> | Efficient-D0/D2 | 71.3 / 80.9 | HSFNet | 73.6 |
> | SSD | 52.9 | CenterNet | 77.7 |
> | ORFENet | **83.3** | | |
>
> #### 정성 결과
> - Fig. 3: ORB가 강조하는 관심 영역(region of interest)이 실제 vehicle/storage tank/ship 위치와 시각적으로 잘 일치함을 확인.
> - Fig. 4: baseline FCOS, FSANet, Cascade-RCNN w/NWD-RKA와 비교 시 ORFENet만 특정 밀집 항만 장면에서 false alarm·missed detection이 거의 없음(빨간 박스로 강조된 영역 비교).
> - Fig. 5(LEVIR-Ship): 잔잔한 바다·짙은 구름 등 다양한 배경 조건에서도 안정적으로 검출.

# Discussion

### 이 아이디어의 잠재적 부작용
- ORB가 학습 시 추가 loss·연산을 요구해 학습 시간/자원이 늘어날 위험 → <mark style="background: #FF5582A6;">논문은 학습 시간 비교(wall-clock time)를 전혀 보고하지 않는다 — "추가 추론 비용 없음"만 강조할 뿐 학습 비용 증가는 다루지 않음.</mark>
- MRFAFEM의 21×21 depthwise conv 두 번 연속(distant context branch)은 큰 receptive field를 저비용으로 얻지만, receptive field가 실제 이론적 크기만큼 유효하게 작동하는지(effective receptive field)는 검증되지 않음 → <mark style="background: #FF5582A6;">논문은 커널 크기 ablation(Table II)만 제시할 뿐 effective receptive field 시각화나 정량 분석은 없다.</mark>

### 한계
- <mark style="background: #FF5582A6;">저자가 결론에서 직접 명시: "제안 방법의 복잡도가 상대적으로 높다"— 향후 과제로 tiny object detection 알고리즘의 복잡도 감소를 명시적으로 제시.</mark>
- <mark style="background: #FF5582A6;">AI(airplane), ST(storage tank) 클래스에서는 다른 SOTA(DetectoRS, Cascade-RCNN w/NWD-RKA)가 더 우세 — 저자가 원인을 별도로 분석하지 않음.</mark>
- ORB의 재구성 target이 원본 이미지 픽셀이 아니라 단순 이진 마스크이므로, SR-TOD의 difference map처럼 "어디서 얼마나 정보가 손실됐는지"에 대한 세밀한 정량 신호는 얻지 못하고 "tiny object 영역인지 아닌지"라는 이진 신호만 얻는다 — 정보 손실의 정도(degree)가 아니라 존재(location) 여부만 다룬다는 점에서 표현력이 제한적일 수 있으나 논문은 이를 명시적으로 논의하지 않음.
- 36 epoch(ORFENet*)와 12 epoch(ORFENet) 결과 차이가 커서(AP 18.9→24.8) 학습 schedule에 대한 민감도가 있어 보이나, 왜 이렇게 큰 차이가 나는지에 대한 분석은 없음.

### 생각할 점
- <mark style="background: #A6E3A1A6;">ORB는 pixel-level foreground/background 마스크만 재구성 target으로 삼는다는 점에서, [[SR-TOD]]의 difference map(원본 이미지 전체를 재구성해 정보 손실 정도를 연속값으로 얻음)보다 훨씬 단순한 self-supervision이다 — "정보 손실 정도"가 아니라 "위치"만 알려주므로 표현력은 낮지만 구현이 단순하고, GT 박스만 있으면 바로 만들 수 있는 마스크라 추가 라벨 비용이 전혀 없다는 장점이 있다.</mark>
- <mark style="background: #A6E3A1A6;">MRFAFEM의 "여러 receptive field를 동적 가중합"이라는 설계는 [[FANet]]의 multi-patch-size FFEM 분기 병렬 융합과 구조적으로 유사한 사상(서로 다른 스케일/범위의 정보를 병렬로 뽑아 학습된 가중치로 합친다)을 공유한다 — 하나는 공간(receptive field), 하나는 주파수(patch size) 축에서 같은 아이디어를 반복한다는 점이 흥미롭다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">Table III에서 "동적 가중치 없이 단순 합"만으로도 AP가 17.3→18.2로 대부분의 이득(+0.9/+1.1)을 차지하고, 동적 가중치 추가는 +0.2뿐이다 — 이는 [[Small_Object_Detection_Approaches]]의 "feature 강화 계열"에서 반복적으로 관찰되는 패턴(다중 소스/스케일 정보를 "쓴다"는 것 자체가 가장 큰 이득이고, "동적으로 조정한다"는 정교화는 부가적)과 일치한다. RS-TOD/FANet 등과 함께 이 패턴을 정량적으로 재확인하는 근거로 쓸 수 있다.</mark>
- <mark style="background: #A6E3A1A6;">ORB(단순 이진 마스크 재구성)와 [[SR-TOD]]의 difference map(연속값 정보 손실 신호)을 같은 백본에서 직접 비교하면, "위치만 아는 것"과 "정보 손실 정도까지 아는 것" 중 어느 신호가 tiny object feature 강화에 더 유효한지 실증할 수 있는 좋은 대조군이 된다.</mark>

# 관련 개념
- [[Self_Reconstruction_Difference_Map]] — SR-TOD가 제안한, 재구성 오차를 정보 손실 신호로 쓰는 개념과 목적이 유사하나, ORFENet의 ORB는 원본 이미지가 아니라 GT 박스 기반 이진 마스크를 재구성 target으로 삼는다는 점에서 더 단순한 변형이다. 다만 실제 메커니즘(재구성 대상, loss 형태, 산출물 활용 방식)이 상당히 다르므로 이 개념 문서의 "등장 논문"에는 추가하지 않고, 이 노트의 "생각할 점"에서만 대조했다. MRFAFEM은 이 논문 안에서만 의미 있는 구현(receptive field 커널 크기 조합 등 세부 튜닝 성격이 강함)으로 판단해 별도 concept 문서로 분리하지 않았다.

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]]

# 읽어볼 만한 논문
- 참고문헌 기반: C. Xu, J. Wang, W. Yang, H. Yu, L. Yu, G.-S. Xia, "RFLA: Gaussian receptive field based label assignment for tiny object detection" (ECCV 2022) [17] — 이 위키에서 여러 논문(CDATOD-Diff, FANet, SR-TOD, Feature_Info_Driven_Gaussian)이 반복적으로 baseline·비교 대상으로 삼는 핵심 선행 연구. ORFENet도 관련 연구에서 label assignment 갈래의 대표작으로 인용하며, 원 논문 자체가 아직 위키에 없어 우선순위가 매우 높다.
- 참고문헌 기반: J. Wu, Z. Pan, B. Lei, Y. Hu, "FSANet: Feature-and-spatial-aligned network for tiny object detection in remote sensing images" (IEEE Trans. Geosci. Remote Sens., 2022) [61] — ORFENet이 1-stage SOTA로 직접 비교하는 대상(Table V/VI). 원격탐사 tiny object 분야의 또 다른 대표 1-stage 접근이라 비교하며 읽을 가치가 있다.
- 참고문헌 기반: Z. Cai, N. Vasconcelos, "Cascade R-CNN: Delving into high quality object detection" (CVPR 2018) [35] — ORFENet의 시각적 비교(Fig. 4)에서 NWD-RKA와 결합해 가장 강력한 2-stage 경쟁자로 등장하는 baseline 아키텍처. Cascade 구조 자체의 배경 이해에 도움.
- 자유 추천(검증 필요): auxiliary reconstruction task를 이용한 self-supervised feature regularization 관련 연구(detection 외 도메인, 예: segmentation/depth estimation에서 auxiliary head를 학습에만 쓰고 추론 시 제거하는 기법) — 검색 키워드: `auxiliary reconstruction task train-time only inference-free regularization object detection`. ORB처럼 "학습에만 관여하고 추론 비용이 0인 auxiliary branch"라는 설계 패턴이 다른 dense prediction task에서 어떻게 쓰이는지 비교하면 이 설계의 일반성을 판단하는 데 도움될 것으로 예상.

Project: [[논문 읽기|논문 읽기]]
