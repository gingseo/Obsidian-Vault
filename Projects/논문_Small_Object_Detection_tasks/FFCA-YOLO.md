---
pm-task: true
projectId: "paperwiki-small-object-detection"
parentId:
id: "t-ffca-yolo-o8q40id3q1"
title: "FFCA-YOLO"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-08-20"
due:
progress: 0
assignees: []
tags: []
customFields:
  "7l8l795xmtcjvlf6": 2024
  "1frf59rymtcjvske": "IEEE TGRS"
subtaskIds: []
dependencies: []
year: 2024
venue: "IEEE TGRS"
jcr_quartile: Q1
task: [small-object-detection]
direction: [improvement]
paper_tags: [paper, small-object-detection, remote-sensing, yolo, lightweight, attention-mechanism, feature-fusion]
source: "/Users/GyeongSeo/Workspace/논문_pdf/Small_Object_Detection/2024_TGRS_FFCA-YOLO.pdf"
createdAt: "2026-08-20T00:00:00.000Z"
updatedAt: "2026-08-28T00:00:00.000Z"
---

#paper #small-object-detection #remote-sensing #yolo #lightweight #attention-mechanism #feature-fusion

> [!quote] 원제
> **FFCA-YOLO for Small Object Detection in Remote Sensing Images**
> Yin Zhang, Mu Ye, Guiyi Zhu, Yong Liu, Pengyu Guo, Junhua Yan — Nanjing University of Aeronautics and Astronautics 외, IEEE Transactions on Geoscience and Remote Sensing (TGRS) 2024

# 한 줄 요약
<mark style="background: #FFF3A3A6;">YOLOv5m backbone의 세 스케일 출력마다 지역 문맥을 넓히는 FEM, neck에서 다중 스케일을 채널별 학습 가중치로 재가중 융합하는 FFM, 검출 헤드 직전에서 채널·공간 전역 문맥을 포착하는 SCAM 세 경량 plug-and-play 모듈을 결합해 원격탐사 소형 객체 탐지 정확도를 끌어올리고, backbone을 PConv 기반으로 재구성해 파라미터를 30% 줄인 경량판 L-FFCA-YOLO까지 함께 제시하는 프레임워크.</mark>

> [!info] 내 메모
> 

# 정리

| | 문제 ① — 얕은 layer의 부족한 지역 문맥 | 문제 ② — 다중 스케일 융합의 정보 손실 | 문제 ③ — 배경 혼동 |
|---|---|---|---|
| **문제 정의** | 원격탐사 영상의 소형 객체(32×32픽셀 미만)는 backbone 초반 layer에서 주로 검출되는데, 이 시점 feature는 수용영역(receptive field)이 좁아 주변 문맥을 충분히 못 본다. | 소형 객체 feature는 backbone 출력 단계에서 픽셀 몇 개로만 표현되므로 단일 스케일 feature만으로는 부족하고, 저·고레벨 다중 스케일 feature를 합쳐야 한다. 그런데 기존 융합 방식(BiFPN 등)은 스케일마다 하나의 가중치만 학습해, 스케일 안의 채널별 정보량 차이를 반영하지 못한다. | 원격탐사 영상은 촬영 거리·플랫폼 모션·복잡한 대기 조건 때문에 객체와 배경의 경계가 흐려지고, 소형 객체일수록 배경으로 오인되기 쉽다. |
| **풀고자 하는 문제** | 파라미터를 거의 늘리지 않으면서 backbone 얕은 layer의 지역 인지 능력(수용영역)을 확장 | 채널 단위로 차등 가중해 다중 스케일 feature를 정보 손실 없이 융합 | 채널·공간 두 방향의 전역 관계를 모델링해 객체와 배경을 구별 |
| **선행 연구 접근** | - RFB-s[52]: 여러 branch(표준 conv+atrous conv 조합)로 수용영역 확장 — branch 수가 많아 다소 무거움<br>- TPH-YOLO[22]: transformer encoder block 삽입 — 파라미터 급증<br>- **갭**: 수용영역을 넓히는 기존 방법들은 정확도는 얻지만 경량성을 희생한다. | - PANet[26], NAS-FPN[27], ASFF[28]: FPN 변형으로 다중 스케일 경로 개선 — 스케일 간 연결 구조 개선에 집중, 채널별 재가중은 다루지 않음<br>- BiFPN[29]: 스케일(feature map) 단위로 학습 가능한 가중합 도입 — **갭**: 같은 feature map 안 모든 채널이 동일 가중치를 공유<br>- FE-YOLO[23]: deformable conv로 상하위 레이어 semantic gap 완화 — 연산 비용 증가 | - NLNet[13]: 모든 픽셀 쌍의 pairwise correlation 직접 계산 — `O((HW)²)` 비용<br>- GCNet[14]: 1×1 conv+softmax로 전역 attention을 근사해 비용 절감 — **갭**: 개별 픽셀 정보 손실<br>- SCP[38]: GCNet에 pixel-wise value path 추가 — 개별 픽셀 정보는 보존하지만 여전히 배경 노이즈를 함께 끌어들일 수 있음 |
| **해결 방법** | RFB-s를 경량화한 FEM(잔차 branch 1개 + atrous conv 포함 3-branch)을 backbone 세 스케일 출력에 삽입 | BiFPN 뼈대에 CRC(채널 단위 학습 가중치로 concat 후 재가중)를 적용한 FFM으로 대체 | GAP+GMP로 압축한 전역 정보에 GMP를 추가 결합한 SCAM으로, 채널 문맥과 공간 문맥을 각각 계산 후 결합 |
| **예상되는 문제점** | 여러 branch를 병렬로 두는 구조라 branch 수만큼 파라미터·연산이 늘어난다(단독 추가 시 6.53M→6.70M). | CRC 세 변형(단순 channel attention/단일 재가중/이중 재가중) 중 하나를 실험적으로 골라야 해, 최적 전략이 데이터셋마다 달라질 가능성이 있다. | GAP/GMP로 압축하는 과정에서 세부 공간 정보 일부가 손실되고, QK 기반 연산이 attention 계열 공통의 비용을 수반한다(단독 추가 시 6.53M→6.92M). |

**갭 종합**: <mark style="background: #FFF3A3A6;">기존 방법들은 지역 문맥 확장·다중 스케일 융합·전역 문맥 모델링 중 한두 가지에만 집중했고, 그마저도 경량성을 희생하는 방향(transformer, deformable conv, 정교한 attention)으로 접근했다. FFCA-YOLO의 통찰은 세 방향 모두를 "거의 파라미터를 늘리지 않는" plug-and-play 모듈로 동시에 다루는 것 — 특히 채널 단위 학습 가중치(CRC)로 다중 스케일을 재가중하는 접근은 BiFPN류의 균일 가중 방식에 비해 선행 연구에서 거의 검토되지 않았다.</mark>

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">YOLOv5m의 backbone 세 스케일 출력에 <span style="color:#c0392b; font-weight:bold;">FEM(Feature Enhancement Module)</span>을 적용해 지역 문맥을 넓히고, neck에서 <span style="color:#c0392b; font-weight:bold;">FFM(Feature Fusion Module)</span>이 BiFPN 구조에 <span style="color:#c0392b; font-weight:bold;">CRC(Channel Reweight Concat)</span> 재가중 전략을 결합해 다중 스케일을 융합한 뒤, 세 검출 헤드 직전에 <span style="color:#c0392b; font-weight:bold;">SCAM(Spatial Context Aware Module)</span>을 배치해 채널·공간 전역 문맥을 반영한다. 세 모듈 모두 경량·plug-and-play로 설계해 임의의 detector에 삽입 가능하다.</mark>

## 전체 파이프라인 (Fig. 1 기준)

```
입력 이미지 (3, 512, 512) — VEDAI 학습 기준 해상도
       │
       ▼
Backbone: Conv → Conv → 4×CSP → Conv → 4×CSP → Conv → 6×CSP → SPPF   (표준 conv 4회로 다운샘플링)
       │           │                │                │
       │       (스케일 P3: 96ch)  (스케일 P4: 192ch)  (스케일 P5: 384ch, SPPF 통과)
       ▼           ▼                ▼                ▼
① FEM (세 스케일 각각에 적용)     → (P2': 160×160)   (P3': 80×80)   (P4': 40×40, 채널수 유지)
       │
       ▼
② FFM — Neck (BiFPN 뼈대 + CRC 재가중, top-down + bottom-up)
       │      X2'(160×160) ← CRC[Upsample(CSP(X3')), X2]
       │      X3'(80×80)   ← CRC[X3, CSP(X3'), Downsample(X2')]
       │      X4'(40×40)   ← CRC[X4, Downsample(X3'')]
       ▼
③ SCAM (세 스케일 각각, 검출 헤드 직전)  → (160,160,C) / (80,80,C) / (40,40,C)   [채널·공간 전역 문맥 반영]
       │
       ▼
YOLO Head × 3                              → (160,160,18) / (80,80,18) / (40,40,18)
```

> [!info] 내 메모
> 

### ① FEM (Feature Enhancement Module)
- **역할**:
  Backbone 얕은 layer의 feature는 수용영역이 좁아 소형 객체 주변 문맥을 충분히 못 본다. FEM은 이 수용영역을 파라미터를 거의 늘리지 않고 넓혀, 배경과 소형 객체를 구별할 단서를 더 많이 확보하는 모듈이다. RFB-s(Receptive Field Block)에서 착안했지만 branch 수를 줄여 더 가볍다.
- **구현**:
  4-branch 구조(Fig. 2) — 잔차 branch 1개(1×1 conv로 채널만 조정, 정보를 그대로 보존) + 3개의 표준+atrous convolution 조합 branch. 각 branch 입력 앞에 [[1x1_Convolution]]으로 채널을 먼저 줄인 뒤, 표준 conv(1×3, 3×1, 3×3 커널)를 거치고 가운데 두 branch의 마지막 conv는 [[Dilated_Convolution]](dilation rate=5)로 대체해 수용영역을 넓힌다. 세 branch의 출력을 concat한 뒤 잔차 branch와 element-wise 합산.
- **입출력 shape**:
  backbone P3/P4/P5 스케일 각각의 `(C, H, W)` → 같은 `(C, H, W)` (채널·공간 크기 불변, 값만 넓은 문맥을 반영해 갱신됨).

```python
# 논문 Eq.(1)-(4) 기반 의사코드. f_conv^kxk: 표준 convolution, f_diconv^3x3: dilation rate=5인 atrous conv
W1 = conv_3x3(conv_1x1(F))
W2 = diconv_3x3( conv_3x1( conv_1x3( conv_1x1(F) ) ) )   # 가운데 branch: 1x1 -> 1x3 -> 3x1 -> atrous 3x3
W3 = diconv_3x3( conv_1x3( conv_3x1( conv_1x1(F) ) ) )   # 세번째 branch: 1x1 -> 3x1 -> 1x3 -> atrous 3x3
Y  = concat(W1, W2, W3) + conv_1x1(F)                     # 잔차 branch와 element-wise 합산
```

<mark style="background: #FFF9D6A6;">Multibranch 표준+atrous convolution 조합이 파라미터를 거의 늘리지 않으면서 수용영역을 넓혀, "얕은 layer의 좁은 수용영역"이라는 문제 ①을 직접 해소한다 — Table V ablation에서 FEM 단독 추가만으로 precision이 0.900→0.926으로 상승한 것이 이를 뒷받침한다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> Branch 4개를 병렬로 두는 구조라, 단독으로 추가하면 backbone 파라미터가 6.53M→6.70M로 늘어난다(Table V). 개별 모듈은 경량이라도 여러 모듈을 함께 쓰면 누적 비용이 발생하는 구조적 특성이 있다.

> [!info] 내 메모
> 

### ② FFM (Feature Fusion Module)
- **역할**:
  Backbone 세 스케일에서 FEM으로 강화된 feature(고레벨=semantic 풍부·저해상도, 저레벨=공간 디테일 풍부·고해상도)를 하나로 합쳐야 소형 객체를 더 잘 표현할 수 있다. FFM은 BiFPN(top-down+bottom-up 양방향 융합 경로) 구조를 뼈대로 쓰되, 융합 시 **모든 채널에 같은 가중치**를 주는 BiFPN의 한계를 채널별 학습 가중치로 대체한 모듈이다.
- **구현**:
  세 검출 헤드에 맞춰 조정한 BiFPN 경로(top-down 1회 + bottom-up 1회) 위에서, 두 feature map을 합칠 때마다 표준 concat 대신 [[Channel_Reweight_Concat]](CRC)을 적용한다. 세 가지 CRC 변형(SENet류 channel attention 기반, 단순 concat 후 균일 채널 가중치, feature map 내부까지 재가중하는 이중 재가중) 중 두 번째(CRC_2)를 채택 — 이중 재가중(CRC_3)과 성능 차이가 거의 없으면서(mAP50:95 0.003 차이) 더 단순하기 때문.
- **입출력 shape**:
  FEM 출력 3개 스케일 `X2(160,160), X3(80,80), X4(40,40)` → 융합된 3개 스케일 `X2'(160,160), X3''(80,80), X4'(40,40)` (채널 수는 CSPBlock 통과 후 유지).

```python
# 논문 Eq.(5)-(7) 기반 의사코드. CBS = 3x3 conv+BN+SiLU, CSP = CSPBlock
X4_prime  = CSP(X4)                                              # 최고레벨부터 시작
X3_prime  = CSP( CRC[ upsample_2x(CSP(X4_prime)), X3 ] )         # top-down: 위→아래
X2_prime  = CSP( CRC[ upsample_2x(CSP(X3_prime)), X2 ] )         # top-down 계속
X3_pprime = CSP( CRC[ CBS(X3_prime), X3, CBS(X2_prime, stride=2) ] )  # bottom-up: 아래→위
X4_pprime = CSP( CRC[ X4_prime, CBS(X3_pprime, stride=2) ] )     # bottom-up 계속
```

<mark style="background: #FFF9D6A6;">BiFPN의 "모든 채널이 동일 가중치를 공유한다"는 한계를 채널 단위 학습 가중치로 대체해, 정보량이 다른 스케일별 feature를 손실 없이 결합한다 — "문제 ②"인 다중 스케일 융합의 정보 손실을 직접 겨냥하며, Table V에서 FFM 단독 추가 시 recall이 0.826→0.837로 오른 것이 이를 뒷받침한다.</mark>

> [!info] 내 메모
> 

### ③ SCAM (Spatial Context Aware Module)
- **역할**:
  FEM·FFM이 지역·다중 스케일 정보를 다뤘다면, SCAM은 이미지 전체에서 픽셀 간 전역 관계를 명시적으로 모델링해 소형 객체와 배경을 전역 문맥에서 구별하도록 만드는 모듈이다. GCNet·SCP(Non-local network 계보)를 계승하되, 정보 집약 단계에 GMP(global max pooling)를 추가 결합해 채널 선택 정보를 보강했다.
- **구현**:
  세 branch로 구성(Fig. 4, [[Global_Context_Modeling_GAP_GMP]] 참고) — (1) GAP+GMP로 전역 정보를 집약해 공간 문맥 가중치 `a`를 만드는 branch, (2) [[1x1_Convolution]]으로 value를 생성하는 branch, (3) 1×1 conv로 query·key(QK)를 생성하는 branch. (1)과 (3)을 행렬곱해 **채널 방향** context를, (1)과 (2)를 결합해 **공간 방향** context를 각각 계산한 뒤, 두 context를 broadcast Hadamard product(원소별 곱)로 결합한다.
- **입출력 shape**:
  FFM 출력 3개 스케일 `(C, H, W)` 각각 → 같은 `(C, H, W)` (검출 헤드 직전, 채널·공간 크기 불변).

```python
# 논문 Eq.(11)-(12) 기반 의사코드. avg/max: GAP/GMP, w_qk/w_v: 1x1 conv(선형변환)
a  = softmax( conv_1x1( concat(avg(P), max(P)) ) )     # 전역 정보 집약 → 공간 문맥 가중치
qk = conv_1x1(P)                                        # query-key
v  = conv_1x1(P)                                        # value
channel_context = matmul(a, qk)                         # 채널 방향 context
spatial_context = matmul(a, v)                          # 공간 방향 context
Q = P + channel_context * spatial_context                # broadcast Hadamard product로 결합
```

<mark style="background: #FFF9D6A6;">픽셀 간 전역 관계를 채널·공간 두 방향에서 동시에 모델링해, "문제 ③"인 배경 혼동을 직접 겨냥한다 — Table VII에서 NLBlock(0.905)·SCP(0.902)·GCBlock(0.907)보다 SCAM(0.909)이 앞선 것은 GMP 추가가 실제로 효과 있었음을 보여준다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> QK 기반 연산이 attention 계열 공통의 비용을 수반해, SCAM 단독 추가 시 파라미터가 6.53M→6.92M로 늘어난다(Table V). 논문은 이 증가분이 실시간성(FPS)에 미치는 영향을 USOD 실험에서 별도로 보고하지 않는다.

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| Backbone | (3, 512, 512) | P3(96,160,160) / P4(192,80,80) / P5(384,40,40) | 이미지 → 다중 스케일 feature | Conv×4 다운샘플 + CSPBlock×3 + SPPF |
| ① FEM | 각 스케일 (C, H, W) | 동일 (C, H, W) | 지역 수용영역 확장 | 4-branch (잔차 + 표준·atrous conv 3-branch), [[1x1_Convolution]] + [[Dilated_Convolution]] |
| ② FFM | 3개 스케일 (C, H, W) | 3개 스케일 (C, H, W) | 다중 스케일 채널별 재가중 융합 | BiFPN 뼈대 + [[Channel_Reweight_Concat]] |
| ③ SCAM | 각 스케일 (C, H, W) | 동일 (C, H, W) | 채널·공간 전역 문맥, 배경 억제 | [[Global_Context_Modeling_GAP_GMP]] (GAP+GMP+QK) |
| YOLO Head ×3 | (160,160,C)/(80,80,C)/(40,40,C) | (160,160,18)/(80,80,18)/(40,40,18) | 최종 박스·클래스 예측 | YOLOv5 표준 검출 헤드 |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table II (VEDAI), Table III (AI-TOD), Table IV (USOD)
**보는 법**: 세 벤치마크 모두 행이 모델, mAP50/mAP50:95/mAPs(소형 객체 전용 mAP) 열로 성능을 비교한다 — YOLOv5m(베이스라인) 대비 FFCA-YOLO의 개선폭을 확인하면 된다.

| 벤치마크 | 지표 | YOLOv5m (baseline) | FFCA-YOLO |
|---|---|---|---|
| VEDAI (Table II) | mAP50 / mAP50:95 / mAPs | 0.723 / 0.410 / 0.399 | 0.748 / 0.448 / 0.446 |
| AI-TOD (Table III, vs HANet SOTA) | mAP50 / mAP50:95 | HANet 0.537 / 0.221 | 0.617 / 0.277 |
| USOD (Table IV) | precision / recall / mAP50 / mAPs | 0.892 / 0.821 / 0.873 / 0.313 | 0.929 / 0.855 / 0.909 / 0.340 |

> [!note]- 세부 결과 및 Ablation
> #### Table II — VEDAI 상세
> **보는 법**: 클래스별(Car/Pickup/... ) AP와 mAP50/mAP50:95/mAPs를 비교. CMAFF(RGB) mAP50 0.743, TPH-YOLO 0.584 대비 FFCA-YOLO 0.748로 최고. L-FFCA-YOLO도 0.913으로 일부 클래스(Car 0.913)에서 오히려 원본보다 소폭 상회.
>
> #### Table III — AI-TOD 상세
> **보는 법**: mAPvt/mAPt/mAPs가 각각 객체 크기 구간(8×8 미만 / 8~16 / 16~32)별 mAP다. FFCA-YOLO는 mAP50 0.617로 현재 최고 모델 HANet(0.537)보다 0.08 높고, mAP50:95·mAPvt·mAPt·mAPs 전 지표에서 HANet을 상회(0.277/0.126/0.249/0.318 vs 0.221/0.109/0.222/0.273).
>
> #### Table IV — USOD 상세
> **보는 법**: Para 열(파라미터 수)까지 함께 봐야 "정확도 대비 경량성"을 판단할 수 있다. TPH-YOLOv5(mAP50 0.895, 45.36M) 대비 FFCA-YOLO는 mAP50 0.909이면서 파라미터는 7.12M로 훨씬 작다. L-FFCA-YOLO는 파라미터 7.12M→5.04M(−30%)이면서 mAP50 0.907로 거의 손실 없음.
>
> #### Table V — 모듈별 기여 Ablation (USOD)
> **보는 법**: 체크(√) 조합마다 성능·파라미터가 어떻게 바뀌는지 행별로 비교.
>
> | FEM | FFM | SCAM | precision | recall | mAP50 | mAP50:95 | mAPs | Para |
> |---|---|---|---|---|---|---|---|---|
> | × | × | × | 0.900 | 0.826 | 0.868 | 0.310 | 0.303 | 6.53M |
> | √ | × | × | 0.926 | 0.839 | 0.899 | 0.343 | 0.335 | 6.70M |
> | × | √ | × | 0.908 | 0.837 | 0.876 | 0.314 | 0.306 | 6.54M |
> | × | × | √ | 0.916 | 0.828 | 0.885 | 0.330 | 0.321 | 6.92M |
> | √ | √ | × | 0.928 | 0.845 | 0.903 | 0.345 | 0.334 | 6.74M |
> | √ | × | √ | 0.925 | 0.842 | 0.901 | 0.342 | 0.334 | 7.09M |
> | × | √ | √ | 0.923 | 0.851 | 0.898 | 0.335 | 0.324 | 6.93M |
> | √ | √ | √ | **0.929** | **0.855** | **0.909** | **0.350** | **0.340** | 7.12M |
>
> FEM 단독 기여가 가장 큼(precision 0.900→0.926, mAPs 0.303→0.335). 세 모듈 모두 충돌 없이 누적 개선.
>
> #### Table VI — FFM 재가중 전략 비교 (USOD)
> **보는 법**: BiFPN 원본(without CRC) 대비 CRC 변형들의 mAP50/mAP50:95 개선폭.
> BiFPN(without CRC) 0.900 < CRC_1(SENet 유사) 0.903 ≈ CRC_1(ECANet 유사) 0.897 < **CRC_2(채택) 0.909** ≈ CRC_3 0.908(mAP50:95는 CRC_2 0.350 > CRC_3 0.347, 차이 0.003).
>
> #### Table VII — SCAM vs 기존 전역 문맥 모듈 (USOD)
> **보는 법**: 같은 자리에 NLBlock/SCP/GCBlock을 각각 넣었을 때와 SCAM을 넣었을 때 성능 비교.
> NLBlock 0.905, SCP 0.902, GCBlock 0.907 < **SCAM 0.909** — 세 대안 모두 상회, 파라미터는 넷 다 7.12M~7.52M 수준으로 큰 차이 없음.
>
> #### Table VIII — Robustness, 이미지 열화 실험 (USOD)
> **보는 법**: PSNR(값이 낮을수록 열화가 심함)이 낮아질수록 mAP50이 얼마나 떨어지는지, 괄호 안 퍼센트가 하락폭.
> Blurring·Fog에는 YOLOv5m 대비 FFCA-YOLO가 상대적으로 강건(예: fog A=0.5에서 FFCA-YOLO −48.6% vs YOLOv5m −55.9%). 그러나 Gaussian noise(σ²=0.01)·stripe noise(r=0.2)에는 FFCA-YOLO·YOLOv5m 둘 다 mAP50이 최대 99.7%까지 급락. 열화 조건을 데이터 증강에 넣어 재학습(retrained)하면 크게 개선되지만(예: stripe noise r=0.2 재학습 후 FFCA-YOLO −18.2%) 완전히 해소되지는 않음.
>
> #### Fig. 11 — FEM·SCAM의 feature map 시각화
> **보는 법**: 원본 이미지 → 원본 feature map → FEM 통과 후 → FEM+SCAM 통과 후 순서로, 밝을수록 모델이 그 위치에 주목한다는 뜻. FEM 이후 소형 객체 위치가 밝아지고, SCAM까지 거치면 배경(도로·건물 등)의 밝기가 더 억제됨을 시각적으로 확인 가능.
>
> #### Table IX — 경량화 비교, L-FFCA-YOLO (USOD)
> **보는 법**: CSPBlock을 여러 경량 블록으로 교체했을 때 mAP50/Para/GFLOPs/FPS 비교. GhostBlock·ShuffleBlock은 파라미터 수 자체는 더 작지만(3.53M/4.13M), GFLOPs 대비 실제 속도(FPS)가 CSPFasterBlock보다 낮다는 점이 핵심 — FLOPs가 낮다고 실제로 빠른 건 아니라는 [[Partial_Convolution]]의 논리와 직결된다.
> CSPFasterBlock(ratio=2): mAP50 0.907, Para 5.04M, GFLOPs 37.1, FPS 191 — GhostBlock(mAP50 0.889, Para 3.53M, GFLOPs 27.3, FPS 204)·ShuffleBlock(mAP50 0.832, Para 4.13M, GFLOPs 32.9, FPS 161)보다 mAP50이 확실히 높고, 파라미터·GFLOPs는 더 크지만 "연산 대비 실제 속도" 효율은 더 낫다는 것이 채택 근거. ratio를 낮출수록(2→1→0.5) 파라미터·GFLOPs·mAP50이 함께 줄어든다(0.907→0.899→0.897, Para 5.04M→4.27M→3.89M).

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- SCAM의 전역 문맥 모델링이 QK 기반 연산을 포함해 attention 계열 공통의 연산 비용을 수반 → <mark style="background: #FF5582A6;">논문은 SCAM 단독 추가 시 파라미터가 6.53M→6.92M로 늘어남을 명시하지만, 이 증가분이 실시간성에 미치는 영향(FPS)은 USOD 실험에서 별도로 보고하지 않는다.</mark>
- 세 모듈을 모두 결합하면 파라미터가 6.53M→7.12M로 누적 증가 → <mark style="background: #FF5582A6;">개별 모듈은 경량이라도 합산 비용은 논문이 강조하는 "lightweight" 프레임과 다소 긴장 관계이며, 이 때문에 별도로 L-FFCA-YOLO가 필요해진 것으로 보인다.</mark>

### 한계
- <mark style="background: #FF5582A6;">Gaussian noise·stripe noise에 대한 강건성이 매우 취약 — 저자가 직접 명시하며, 향후 image denoising·nonuniformity correction 등 전처리가 필요하다고 인정.</mark>
- <mark style="background: #FF5582A6;">지상/항공 기반(VEDAI, AI-TOD, USOD) 데이터셋에서만 검증됨 — 우주 기반(space-based) 원격탐사 영상은 해상도가 낮고 열화가 더 심해, 저자 스스로 "효과성이 아직 검증되지 않았다"고 명시.</mark>
- 저자가 명시적으로 "단일 모달 데이터 소스만으로는 한계가 있다"며 multiplatform/multiband 융합을 향후 방향으로 제시 — 이 논문 자체는 RGB 단일 모달만 다룸.

### 생각할 점
- <mark style="background: #A6E3A1A6;">FFM의 CRC(채널 단위 학습 가중치)는 BiFPN 계열을 쓰는 다른 논문([[RS-TOD]] 등)에도 손쉽게 이식 가능해 보이는 범용적 개선으로, "학습 가능한 재가중"이라는 아이디어 자체가 이 위키의 다른 feature 강화 계열과 결합할 여지가 크다.</mark>
- <mark style="background: #A6E3A1A6;">SCAM이 GCNet/SCP 대비 GMP를 추가한 것만으로 일관되게 앞선다는 점(Table VII)은, "정보 집약 방식을 다양화하는 것"이 attention 설계에서 저비용 고효율 개선 지점이 될 수 있음을 시사한다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">[[LSOD-YOLO]]는 "저기여 헤드 제거로 경량화"를, FFCA-YOLO는 "PConv로 backbone을 재구성해 경량화"를 택했다 — 두 경량화 전략이 상호 배타적이지 않아 보이므로, LCOR(P5 제거)와 PConv 기반 L-FFCA-YOLO backbone을 함께 적용하면 추가 경량화 여지가 있는지 검토할 가치가 있다.</mark>
- <mark style="background: #A6E3A1A6;">[[Small_Object_Detection_Approaches]]의 feature 강화 계열 중 [[SR-TOD]]·[[ORFENet]]은 reconstruction 기반 신호를, FFCA-YOLO는 순수 attention/융합 기반 신호를 쓴다 — SCAM의 전역 문맥 정보를 SR-TOD류의 difference map과 결합해 "어디를 강조할지"를 두 신호로 교차 검증하는 방향도 가능해 보인다.</mark>

> [!info] 내 메모
> 

# 관련 개념
- [[1x1_Convolution]] — FEM 각 branch의 채널 조정, SCAM의 value/QK 생성에 사용.
- [[Dilated_Convolution]] — FEM의 atrous convolution branch(dilation rate=5)에 사용, 파라미터 증가 없이 수용영역 확장.
- [[Channel_Reweight_Concat]] — FFM의 핵심 재가중 전략(CRC). 이 논문에서 최초 제시.
- [[Global_Context_Modeling_GAP_GMP]] — SCAM의 GAP+GMP 기반 전역 문맥 모델링. GCNet/SCP 계보를 이 논문이 GMP 추가로 확장.
- [[Partial_Convolution]] — L-FFCA-YOLO의 CSPFasterBlock에 쓰인 경량 convolution.
- [[Remote_Sensing_Attention_Module]] — RS-TOD의 채널+공간 attention과 개념적으로 유사(둘 다 detection head 주변에 attention 삽입). SCAM은 GCNet/SCP 계보의 전역 문맥 모델링이라는 점에서 구현 방식은 다르나, "원격탐사 특화 attention으로 배경 억제"라는 목적은 공유. RS-TOD 노트의 "등장 논문"에 이번 논문 추가 갱신함.

# 관련 문서
- 비교: [[Small_Object_Detection_Approaches]] — feature 강화(FEM/FFM/SCAM) + 아키텍처 경량화(L-FFCA-YOLO) 이중 축으로 분류, [[LSOD-YOLO]]와 경량화 축에서 직접 대응.

# 읽어볼 만한 논문
- 참고문헌 기반: Y. Cao, J. Xu, S. Lin, F. Wei, H. Hu, "GCNet: Non-local networks meet squeeze-excitation networks and beyond" (ICCV Workshop 2019) [14] — SCAM이 계승한 전역 문맥 모델링의 원조. SCAM의 GAP/GMP 확장을 이해하려면 먼저 읽을 필요가 있음.
- 참고문헌 기반: J. Chen et al., "Run, don't walk: Chasing higher FLOPS for faster neural networks" (CVPR 2023) [51] — L-FFCA-YOLO가 채택한 PConv(FasterNet)의 원조 논문. DWConv의 낮은 FLOPS가 실제로는 빈번한 메모리 접근 때문이라는 분석이 L-FFCA-YOLO 설계의 근거.
- 참고문헌 기반: M. Tan, R. Pang, Q. V. Le, "EfficientDet: Scalable and efficient object detection" (CVPR 2020) [29] — FFM이 뼈대로 삼은 BiFPN의 원조 논문. CRC와의 차이(균일 가중 vs 채널별 학습 가중)를 비교하려면 필요.
- 자유 추천(검증 필요): PConv와 LCOR([[LSOD-YOLO]])류 헤드 재배치 경량화 기법을 함께 적용한 하이브리드 경량 원격탐사 detector 연구 — 검색 키워드: `lightweight remote sensing object detection partial convolution head reduction combined`. 두 경량화 전략이 이 위키에서 아직 결합된 사례가 없어 검증 필요.
