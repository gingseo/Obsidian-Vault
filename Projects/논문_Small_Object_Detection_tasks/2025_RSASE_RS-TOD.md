---
pm-task: true
projectId: "paperwiki-small-object-detection"
parentId:
id: "t-rs-tod-gwovwgdzjk"
title: "RS-TOD: Tiny object detection model in Remote Sensing Imagery"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-07-01"
due: ""
progress: 0
assignees: []
tags: []
customFields:
  "7l8l795xmtcjvlf6": 2025
  "1frf59rymtcjvske": "Remote Sensing Applications: Society and Environment (Elsevier)"
subtaskIds: []
dependencies: []
year: 2025
venue: "Remote Sensing Applications: Society and Environment (Elsevier)"
jcr_quartile: Q2
task: [small-object-detection]
direction: [improvement]
paper_tags: [paper, small-object-detection, remote-sensing, yolo, attention-module, detection-head]
source: "Projects/_pdf/Small_Object_Detection/2025_RSASE_RS-TOD.pdf"
source_type: personal
createdAt: "2026-08-18T11:00:00.000Z"
updatedAt: "2026-08-18T11:09:08.271Z"
---

#paper #small-object-detection #remote-sensing #yolo #attention-module #detection-head

> [!quote] 원제
> **RS-TOD: Tiny object detection model in Remote Sensing Imagery**
> Rakhi Nautiyal, Maroti Deshmukh — Computer Science and Engineering Department, National Institute of Technology, Srinagar (Garhwal), Uttarakhand, India, Remote Sensing Applications: Society and Environment (Elsevier) 2025
> https://doi.org/10.1016/j.rsase.2025.101582

# 한 줄 요약
<mark style="background: #FFF3A3A6;">YOLOv8n의 4개 detection head 앞마다 channel+spatial attention 모듈 RSAM을 배치하고, backbone 최상위 해상도 feature(B3)를 neck에 직접 끌어와 160×160 해상도의 tiny-object 전용 detection head를 새로 추가해, 원격탐사 영상(SODA-A/AI-TOD/TinyPerson)에서 tiny object detection 성능을 baseline 대비 큰 폭으로 끌어올린 경량 모델.</mark>

> [!info] 내 메모
> 

# 정리

## 기존 방법의 한계
- **정보 소실**:
  RSI(Remote Sensing Imagery)는 밀집·불규칙 분포한 tiny object가 많고 배경이 장면 대부분을 차지한다. CNN의 연속적 다운샘플링으로 최상위(topmost) feature map에서 객체가 원본 대비 1/16 크기로 축소되어, 이미 몇 픽셀 안 되는 tiny object의 신호가 거의 지워진다.
- **Scale/orientation 다양성 및 복잡한 배경**:
  같은 장면에 배와 항구처럼 스케일 차이가 큰 객체가 공존하고, 차량처럼 객체 방향이 랜덤하게 분포한다. 고정 receptive field는 이런 스케일 편차에 대응하지 못하며, 배경이 장면 대부분을 차지해 객체 영역이 압도(overshadow)되고 밀집·유사 클래스 객체 간 혼동이 발생한다.

## 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Attention/feature 강화 기반**
- FFCA-YOLO(Zhang et al., 2024): class imbalance 대응에 초점, RSAM처럼 spatial+channel을 체계적으로 묶지는 않음.
- CSDP-YOLO, SCDNet/SDSDet, SESA-Net, SEB-YOLO/PLNet-PR, MFCANet/CTAM, KCFS-YOLO/RSI-YOLO 등: attention/feature 강화 기반 다양한 시도 — 대부분 attention 도입이나 feature fusion 한 축에만 집중, 전용 head 설계는 드묾.
- **타겟/해결**: 정보 소실(①)·복잡한 배경(②) 모두를 겨냥하지만, "얕은 feature map을 tiny object 전용 경로로 분리"와 "channel+spatial attention을 매 head 앞에 체계적으로 배치"를 함께 다루는 연구는 드물다.

**갈래 2 — 경량화 및 loss 개선**
- LE-YOLO, STF-YOLO: edge computing 겨냥 경량 설계로 mAP 유지·연산량 감소.
- YOLO-DCTI, MCS-YOLO: Soft-CIOU, IoU-T 등 loss 개선으로 localization 정교화.
- **타겟/해결**: 정보 소실(①) 자체보다는 배포 효율·localization 정교화를 겨냥 — RSAM이 직접 비교하는 SE Block(channel만, spatial 무시)과 CBAM(channel+spatial이지만 2-level pooling·skip connection 없음) 모두 원격탐사 특유의 밀집·복잡 배경을 겨냥해 설계되지 않았다.

**갭**: <mark style="background: #FFF3A3A6;">대부분의 선행 연구는 attention/feature 강화 또는 loss 개선 중 한 축에만 집중하며, "얕은 feature map을 tiny object 전용 경로로 분리"와 "channel+spatial attention을 매 head 앞에 체계적으로 배치"를 함께 다루는 연구는 드물다.</mark>

## 이 논문이 풀고자 하는 문제
1. YOLOv8의 feature representation을 원격탐사 영상의 복잡한 배경·중첩 객체 상황에 맞게 강화하는 attention 모듈을 설계하는 것.
2. Shallow feature map을 활용하는 전용 검출 경로를 마련해, 다운샘플링 과정에서 소실되는 tiny object의 디테일을 detection head 단계까지 보존하는 것.

**갭 종합**: <mark style="background: #FFF3A3A6;">대부분의 선행 연구는 attention/feature 강화 또는 loss 개선 중 한 축에만 집중하며, "얕은 feature map을 tiny object 전용 경로로 분리"와 "channel+spatial attention을 매 head 앞에 체계적으로 배치"를 함께 다루는 연구는 드물다는 것이 RS-TOD의 통찰이다.</mark>

> [!info] 내 메모
> 

# 해결 방법 요약

| | 문제 ① — 정보 소실 | 문제 ② — Scale/orientation 다양성 및 복잡한 배경 |
|---|---|---|
| **해결 방법** | 3-level channel pooling과 skip connection을 갖춘 RSAM을 4개 detection head 각각의 입력 앞에 배치해 feature representation 강화 | Backbone의 얕은 C2f feature(B3)를 neck의 업샘플된 feature와 결합한 160×160 해상도의 tiny-object 전용 detection head 추가 |
| **예상되는 문제점** | RSAM의 3-level pooling에 대한 ablation(레벨 수 변화 비교)이 논문에 없어, 왜 3-level이 최적인지 실험 근거가 약하다. | Head 4개+RSAM 4개로 추론 경로가 길어져, 실측 결과 파라미터·GFLOPs는 오히려 감소했지만 FPS는 60→45로 하락(Table 13) — 이유가 명시적으로 설명되지 않는다. |

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">YOLOv8n을 기반으로 (1) 3-level channel pooling과 skip connection을 갖춘 <span style="color:#c0392b; font-weight:bold;">RSAM(Remote Sensing Attention Module)</span>을 4개 detection head 각각의 입력 앞에 배치해 feature representation을 강화하고, (2) backbone의 얕은 C2f feature를 neck의 업샘플된 feature와 결합한 <span style="color:#c0392b; font-weight:bold;">160×160 해상도의 tiny-object 전용 detection head</span>를 새로 추가한다.</mark>

## 전체 파이프라인 (Fig. 3 기준)

```
입력 (3, 640, 640)
       │
       ▼
Backbone (B1~B9, CSPDarknet53)             → B3: C2f 출력(160,160,128) / B9: SPPF 출력(20,20,1024)
       │
       ▼
Neck (B10~B26, FPN+PANet 스타일 top-down/bottom-up)
       │      B16(C2f 이후) → Upsample → B17: B3와 Concat → B18: C2f로 정제 → (160,160,128)
       │      B19~B26: 기존 3-scale(80×80/40×40/20×20) top-down+bottom-up 융합
       ▼
① RSAM (4개 head 각각, B27/B29/B31/B33)     → 동일 shape (채널·공간 크기 불변, attention reweight)
       │
       ▼
② YOLO Head ×4                              → Detect(20,20,1024) / Detect(40,40,512) / Detect(80,80,256) / Detect(160,160,128) [신규]
```

> [!info] 내 메모
> 

### ① 160×160 Tiny-scale Detection Head

- **역할**:
  YOLOv8의 기존 3개 head(80×80, 40×40, 20×20)는 모두 backbone 하위 레벨(B4 이후) feature만 사용해, 가장 얕은 검출 경로도 80×80에서 멈춘다. 깊은 feature는 의미 정보는 풍부하나 해상도가 낮고, 얕은 feature는 반대다 — 다운샘플링으로 소실된 tiny object 신호를 살릴 경로가 기존 구조에는 없었다.
- **구현**:
  Neck의 B16(C2f 이후)에 업샘플링을 추가하고, backbone 최상위 해상도 첫 C2f(B3, 160×160×128)와 concat(B17)한 뒤 추가 C2f(B18)로 정제한다. 정제된 feature는 RSAM(B33)을 거쳐 새 head(B34, 160×160)로 입력된다. B18까지 과도한 다운샘플링을 거치지 않아 tiny object 디테일이 보존된다.
- **입출력 shape**: `B16 출력` + `B3(160,160,128)` → concat(B17) → C2f(B18) → `(160, 160, 128)` → RSAM → Detect head.

<mark style="background: #FFF9D6A6;">160×160 head는 backbone 최상위 해상도 feature(B3)를 직접 끌어와, 기존 YOLOv8이 원천적으로 가지지 못했던 얕은 검출 경로를 확보함으로써 다운샘플링에 의한 정보 소실(문제 ①)을 구조적으로 우회한다. 저자들은 학습 시 IoU 상승 → 정확한 bbox 생성에 기여한다고 설명한다.</mark>

> [!info] 내 메모
> 

### ② RSAM (Remote Sensing Attention Module)

- **역할**:
  단순 channel attention(SE Block)만으로는 "어디를 봐야 하는지"를 특정하지 못하고, 2-level pooling만 쓰는 CBAM은 세밀한 채널 상호작용을 놓친다. RSAM은 3-level pooling으로 촘촘한 채널 정보를 유지하면서 spatial attention까지 적용해 배경 대비 객체 영역에 가중치를 집중시키는, 원격탐사 특화 channel+spatial attention 모듈이다.
- **구현**:
  입력 feature map `I(h,w,c)`에 채널 크기 1, c/4, c/2 세 레벨로 AvgPool을 병렬 적용해 pyramid pooled feature 획득. 이를 concat 후 1×1 conv로 채널 1로 축소해 attention map 생성, BatchNorm+Sigmoid로 reweight map을 얻어 원본 `I`에 element-wise 곱. 4개 detection head 각각의 입력 앞(B27/B29/B31/B33)에 배치.
- **입출력 shape**: `I(h,w,c)` → 세 레벨 AvgPool `(h,w,1)/(h,w,c/4)/(h,w,c/2)` → concat → `(h,w,c_out)` → 1×1 conv+BN+Sigmoid → reweight `(h,w,1)` → 원본과 element-wise 곱 → `(h,w,c)`.

```python
# 논문 Eq.(1)-(2) 기반 의사코드
X1 = AvgPool_c1(I)     # (h, w, 1)
X2 = AvgPool_c_quarter(I)  # (h, w, c/4)
X3 = AvgPool_c_half(I)     # (h, w, c/2)
M = Sigmoid(BatchNorm(Conv1x1(concat(X1, X2, X3))))   # attention map, (h, w, 1)
output = I * M                                          # element-wise 곱 (reweight)
```

<mark style="background: #FFF9D6A6;">3-level pooling으로 촘촘한 채널 정보를 유지하면서 spatial attention까지 적용해 배경 대비 객체 영역에 가중치를 집중시키고, skip connection으로 gradient 흐름을 유지한다 — "배경에 압도되어 객체 영역이 무시된다"는 문제 ②의 원인에 직접 대응한다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> RSAM을 4개 head 모두에 배치하는 구조라 추론 경로가 길어진다 — 실측 결과 파라미터(3.81M→2.93M)와 GFLOPs(8.7B→7.8B)는 오히려 감소했지만, FPS는 60→45로 25% 하락한다(Table 13). 논문은 GFLOPs 감소와 FPS 하락이 동시 발생하는 이유를 설명하지 않는다.

> [!info] 내 메모
> 

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| Backbone | (3,640,640) | B3(160,160,128) ~ B9(20,20,1024) | 이미지 → 다중 스케일 feature | CSPDarknet53(Conv+C2f+SPPF) |
| Neck | B3~B9 | 4개 스케일(160/80/40/20) | 다중 스케일 융합, tiny 경로 신설 | FPN+PANet + B16→B18 신규 upsample/concat/C2f |
| ① 160×160 Head 경로 | B16 출력 + B3(160,160,128) | (160,160,128) | tiny object 디테일 보존 경로 신설 | Upsample+Concat(B17)+C2f(B18) |
| ② RSAM ×4 | 각 head 입력 (h,w,c) | 동일 (h,w,c) | 배경 억제, 객체 영역 가중 | 3-level AvgPool + 1×1 conv + BN + Sigmoid |
| YOLO Head ×4 | (160,160,128)/(80,80,256)/(40,40,512)/(20,20,1024) | 클래스+박스 예측 | 최종 검출 | YOLOv8 표준 Detect head |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table 6(SODA-A), Table 7(AI-TOD)
**보는 법**: 두 벤치마크 모두 baseline YOLOv8 대비 RS-TOD의 mAP50 개선폭을 확인하면 된다.

| 벤치마크 | 지표 | Before(YOLOv8) | After(RS-TOD) |
|---|---|---|---|
| SODA-A | mAP50 | 52.81% | 60.10% (+7.29%p) |
| AI-TOD | mAP50 | 48.50% | 59.84% (+11.34%p) |

> [!note]- 세부 결과 및 Ablation
> #### Baseline 대비 개선 전체 (Table 6, 7, 10)
> | 벤치마크 | 지표 | Before(YOLOv8) | After(RS-TOD) | 비고 |
> |---|---|---|---|---|
> | SODA-A | mAP50 | 52.81% | 60.10% | +7.29%p (baseline: YOLOv8n) |
> | SODA-A | mAP50-95 | 14.30% | 20.30% | +6.00%p |
> | AI-TOD | mAP50 | 48.50% | 59.84% | +11.34%p (baseline: YOLOv8s) |
> | AI-TOD | mAP50-95 | 22.37% | 28.40% | +6.03%p |
> | TinyPerson | mAP50 | 40.52% | 47.60% | +7.08%p (baseline: YOLOv8s) |
> | TinyPerson | mAP50-95 | 13.40% | 16.80% | +3.40%p |
>
> 설정: SODA-A(2,513장, 6class, 872,069 instance), AI-TOD(28,036장, 8class, 700,621 instance), TinyPerson(1,610장, 1class person, 72,651 instance). 비교 대상은 YOLOv5n/v6n/NAS/v9t/v10n/11n, baseline YOLOv8(n 또는 s), 데이터셋별 최신 문헌(TBNet, EL-YOLO, DetectoRS w/ SR-TOD, Faster RCNN-FPN 등). 아래 수치는 별도 표기 없으면 1280×1280 기준.
>
> #### SODA-A 클래스별 성능 (mAP50, Table 6)
> | 클래스 | YOLOv8n baseline | RS-TOD | 비고 |
> |---|---|---|---|
> | Small Vehicle | 20.67 | **49.20** | |
> | Large Vehicle | **42.69** | 39.50 | 유일하게 baseline이 우세 (mAP50-95도 30.16 vs 25.84) |
> | Ship | 20.40 | **42.70** | |
> | Storage Tank | 35.21 | **48.80** | |
> | Container | 29.30 | **69.52** | |
> | Swimming Pool | 72.30 | **77.40** | |
> | ALL | 52.81 | **60.10** | |
>
> #### AI-TOD 클래스별 성능 (mAP50, Table 7)
> | 클래스 | YOLOv8s baseline | RS-TOD | 비고 |
> |---|---|---|---|
> | Airplane | 24.20 | **30.00** | |
> | Bridge | 27.68 | **32.00** | |
> | Storage Tank | 39.00 | **41.34** | |
> | Ship | 40.84 | **58.96** | |
> | Swimming Pool | 38.13 | **42.00** | |
> | Vehicle | **27.66** | 26.70 | baseline 근소 우세 — 형태/스케일 다양성, Storage Tank와 class 유사성 |
> | Person | 11.27 | **14.18** | |
> | Windmill | 9.60 | 13.40 | YOLO-NAS가 최고(16.34) — 극심한 class imbalance(train 176 / test 67) |
> | ALL | 48.50 | **59.84** | |
>
> #### 기존 문헌 대비 비교 (Table 8, 11)
> | 벤치마크 | 비교 모델 | 지표 | 비교 모델 | RS-TOD | 비고 |
> |---|---|---|---|---|---|
> | AI-TOD | TBNet | mAP50 | 59.00 | **59.84** | +0.84%p |
> | AI-TOD | EL-YOLO-s | mAP50-95 | 26.30 | **28.40** | +2.10%p |
> | AI-TOD | DetectoRS w/ SR-TOD | mAP50 / mAP50-95 | 54.60 / 24.00 | **59.84 / 28.40** | RS-TOD가 mAP50-95에서도 상회 |
> | TinyPerson | Faster RCNN-FPN | mAP50 | 43.55 | **47.60** | +4.05%p |
> | TinyPerson | Faster RCNN-FPN | mAP50-95 | 5.35 | **16.80** | +11.45%p |
>
> #### 해상도 비교 (Table 12)
> | 데이터셋 | 지표 | 640×640 | 1280×1280 |
> |---|---|---|---|
> | SODA-A | mAP50 | 53.78 | **60.10** |
> | AI-TOD | mAP50 | 50.24 | **59.84** |
> | TinyPerson | mAP50 | 41.89 | **47.60** |
>
> #### 연산 비용 (Table 13)
> | 모델 | #Param(M) | FPS | GFLOPs(B) | 모델 크기(MB) |
> |---|---|---|---|---|
> | Baseline YOLOv8 | 3.81 | **60** | 8.7 | **12.2** |
> | RS-TOD | **2.93** | 45 | **7.8** | 12.9 |
>
> 파라미터·GFLOPs는 감소했지만(레이어 수 225→168) FPS는 60→45로 25% 하락, 모델 크기는 0.7MB 증가 — attention 모듈이 경량(1×1 conv 위주)이라 파라미터 총량은 줄었으나, FPS 하락은 head 개수 증가로 인한 추론 단계 연산 분기 증가로 추정(논문이 이 괴리를 직접 설명하지는 않음).
>
> #### 정성 결과 (Fig. 11~14)
> Fig. 11(SODA-A)·Fig. 12(AI-TOD)·Fig. 13(TinyPerson): RS-TOD가 YOLOv8 대비 더 많은 객체를 더 높은 confidence로 검출. Fig. 14: 극도로 작은(15×15픽셀 미만) person 인스턴스에서 YOLOv8은 놓치지만(missed) RS-TOD는 성공적으로 검출하는 failure case 비교.

> [!info] 내 메모
> 

# Discussion

### 이 아이디어의 잠재적 부작용
- 추론 경로가 길어질 위험(head 4개+RSAM 4개 추가) → 실측 결과 파라미터·GFLOPs는 오히려 감소했지만 <mark style="background: #FF5582A6;">FPS는 60→45로 25% 하락, 논문은 GFLOPs 감소와 FPS 하락이 동시 발생하는 이유를 설명하지 않는다.</mark>
- RSAM이 원격탐사 특유 조건(복잡 배경, 중첩 객체)을 겨냥해 설계됨 → <mark style="background: #FF5582A6;">원격탐사 세 데이터셋에만 검증, COCO 등 일반 자연 영상 일반화 실험은 없다. 저자도 "다른 attention과 결합해 일반화 개선 가능"이라 언급해 한계를 사실상 인정한다.</mark>

### 한계
- <mark style="background: #FF5582A6;">AI-TOD Windmill 클래스(train 176/test 67, 극심한 class imbalance)에서 RS-TOD(13.40)가 YOLO-NAS(16.34)보다 낮다</mark> — oversampling으로 개선 여지 언급만 하고 실제 적용은 안 했다.
- <mark style="background: #FF5582A6;">AI-TOD Vehicle, SODA-A Large Vehicle에서 baseline이 근소 우세</mark> — 형태/스케일 다양성과 유사 클래스 혼동이 원인으로 지목되나 RSAM/추가 head가 이를 해결하도록 설계되지는 않았다.
- <mark style="background: #FF5582A6;">저자 명시 한계: 성능에 가장 큰 영향을 미치는 요인은 데이터셋 자체(class imbalance, 유사성, 라벨링 품질, 중첩, 노이즈)이며 전처리로 일부 완화 가능하나 근본 해결은 안 된다.</mark>
- RSAM의 3-level pooling에 대한 ablation(레벨 수 변화 비교)이 논문에 없음 — Discussion에서 제안만 하고 실험 근거는 없다.

### 생각할 점
- <mark style="background: #A6E3A1A6;">RSAM의 attention map 계산 자체(3-level AvgPool + 1×1 conv + Sigmoid)는 도메인 특정적이지 않다 — "복잡한 배경·밀집 객체" 조건은 드론뷰·감시 카메라·의료 영상에도 있어, 원격탐사 특화인지 일반적 attention 개선인지는 다른 도메인 실험 없이 판단하기 어렵다.</mark>
- <mark style="background: #A6E3A1A6;">160×160 head가 backbone 얕은 feature(B3)를 그대로 끌어오는 방식은 [[2024_ECCV_SR-TOD|SR-TOD]]가 difference map으로 "정보 손실 큰 영역"을 간접적으로 찾는 방식과 대조적이다 — RS-TOD는 위치를 특정하지 않고 얕은 해상도 경로를 통째로 추가하는 반면, SR-TOD는 reconstruction 난이도 신호로 위치를 특정한 뒤 강화를 가한다. 결합 가능성이 있다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">RS-TOD의 RSAM(feature 강화)과 [[2026_TIP_Unc-SOD|Unc-SOD]]의 label assignment 축(instance-level uncertainty 기반 동적 sampling)은 서로 다른 지점에 개입하므로 직교적이다 — RSAM으로 강화된 feature 위에서 uncertainty 기반 sampling을 적용하면 개선이 누적될 가능성이 있다.</mark>
- <mark style="background: #A6E3A1A6;">RS-TOD Table 8에서 직접 비교되는 DetectoRS w/ SR-TOD([[2024_ECCV_SR-TOD|SR-TOD]])는 mAP50-95(24.00)에서 RS-TOD(28.40)에 뒤진다 — 같은 AI-TOD 벤치마크에서 이미 간접 비교되고 있으므로, difference map 기반 강화와 RSAM+전용 head를 같은 백본에서 직접 결합하는 실험이 자연스러운 다음 단계다.</mark>

> [!info] 내 메모
> 

# 관련 개념
- [[Remote_Sensing_Attention_Module]] — 이 논문이 새로 제안하는 channel+spatial attention 모듈(RSAM). 각 detection head 앞에 배치되어 feature representation을 강화하는 핵심 기여.

# 관련 문서
- 주의(혼동 방지): 이름이 유사한 [[2024_ECCV_SR-TOD|SR-TOD]](difference-map 기반 tiny object detection, 2024 ECCV)와는 저자·방법론이 전혀 무관한 별개 논문이다 — RS-TOD는 YOLOv8 기반 attention+head 확장, SR-TOD는 self-reconstruction difference map 기반 feature 강화. 다만 RS-TOD의 Table 8에서 "DetectoRS w/ SR-TOD"가 AI-TOD 비교 대상 중 하나로 실제로 인용되므로, 두 논문은 이름만 비슷한 게 아니라 같은 벤치마크(AI-TOD)에서 실제로 비교되는 관계이기도 하다.
- 비교: [[Small_Object_Detection_Approaches]] — feature 강화(attention) + 헤드 추가 축, 원격탐사 특화 계열로 분류

# 읽어볼 만한 논문
- 참고문헌 기반: Z. Li, Y. Wang, D. Xu, Y. Gao, T. Zhao, "TBNet: A texture and boundary-aware network for small weak object detection in remote-sensing imagery" (Pattern Recognition, 2025) — RS-TOD가 AI-TOD에서 mAP50 기준 직접 비교하는 가장 근접한 경쟁 모델(59.00 vs 59.84). texture/boundary를 명시적으로 다루는 접근이라 RSAM의 spatial attention과 대비하며 읽을 만하다.
- 참고문헌 기반: B. Cao, H. Yao, P. Zhu, Q. Hu, "Visible and Clear: Finding Tiny Objects in Difference Map" (ECCV, 2025 — 논문 내 인용 표기는 Cao et al., 2025) — 이미 위키에 [[2024_ECCV_SR-TOD|SR-TOD]]로 등록된 논문. RS-TOD의 Table 8에서 AI-TOD 비교 대상("DetectoRS w/ SR-TOD")으로 직접 인용되어, RS-TOD와 실측 비교가 가능한 가장 가까운 대안적 feature 강화 접근.
- 참고문헌 기반: C. Xu, J. Wang, W. Yang, H. Yu, L. Yu, G.-S. Xia, "Detecting tiny objects in aerial images: A normalized Wasserstein distance and a new benchmark" (ISPRS J. Photogramm. Remote Sens., 2022) — RS-TOD가 Table 8에서 "RetinaNet w/ NWD-RKA"로 비교하는 label-assignment 계열 대표 논문. RSAM 같은 feature 강화 축과는 다른 축(assignment metric)에서 tiny object 문제를 다뤄, [[2026_TIP_Unc-SOD|Unc-SOD]] 노트에서 이미 추천된 RFLA와 함께 label-assignment 계열 배경 이해에 도움.
- 자유 추천(검증 필요): CBAM(Convolutional Block Attention Module)의 원 논문 — 검색 키워드: `CBAM convolutional block attention module ECCV 2018`. RS-TOD 본문이 RSAM을 SE Block, CBAM과 직접 비교하며 설계 근거로 삼고 있어, RSAM의 3-level pooling·skip connection이 실제로 CBAM 대비 어떤 구조적 차이인지 원 논문으로 확인할 가치가 있다.
