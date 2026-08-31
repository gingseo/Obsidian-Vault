# YOLO: You Only Look Once (v1) 정리 노트

- **저자**: Joseph Redmon, Santosh Divvala, Ross Girshick, Ali Farhadi
- **소속**: University of Washington, Allen Institute for AI, Facebook AI Research
- **arXiv**: 1506.02640v5 (2016)
- **한줄 요약**: Object detection을 "분류 후 위치 보정"이 아니라, 이미지 전체에서 바운딩박스 좌표와 클래스 확률을 곧바로 회귀(regression)하는 **단일 CNN 문제**로 재정의한 논문. 속도(45~155 FPS)를 detection 연구의 화두로 올려놓은 원조.

---

## 1. 문제 정의

기존 detection 파이프라인(DPM, R-CNN 계열)의 공통 구조:

1. 이미지에서 후보 영역(region proposal) 또는 sliding window 생성
2. 각 영역에 분류기(classifier) 적용
3. NMS, 박스 refine, rescoring 등 후처리

문제점:

- 파이프라인의 각 단계를 **따로 학습**해야 해서 최적화가 어렵고 느림 (Fast R-CNN도 0.5 FPS 수준, Selective Search만 이미지당 ~2초)
- Sliding window/OverFeat 계열은 지역적 정보만 보므로 배경을 객체로 착각하는 오류(background false positive)가 많음
- 실시간 처리가 거의 불가능

YOLO의 재정의: **detection = 이미지 픽셀 → (bbox 좌표, 클래스 확률)로 가는 단일 회귀 문제**. 파이프라인을 없애고 하나의 CNN이 한 번의 forward pass로 전체를 처리.

---

## 2. 방법 (Unified Detection)

### 2.1 그리드 기반 예측

- 입력 이미지를 **S×S grid**로 분할 (VOC 실험: S=7)
- 객체 중심이 속한 grid cell이 그 객체를 "책임"지고 예측
- 각 cell은 **B개의 bounding box**(x, y, w, h, confidence)와 **C개의 클래스 조건부 확률** Pr(Class_i | Object)을 예측
  - B=2, C=20 (PASCAL VOC 20 클래스) → 출력 텐서 크기 **7×7×30** (= B×5 + C)
- confidence = Pr(Object) × IOU(pred, truth). 객체가 없으면 0에 가깝게 학습됨
- x, y는 grid cell 기준 offset, w, h는 전체 이미지 기준 비율로 정규화 (모두 0~1)
- 테스트 시: box confidence × class 조건부 확률 = **class-specific confidence score** (식 1)

### 2.2 네트워크 구조

- GoogLeNet에서 영감을 받았지만 inception module 대신 **1×1 reduction + 3×3 conv** 조합 사용 (Lin et al., Network in Network 방식)
- **24개 conv layer + 2개 FC layer**, 입력 448×448 → 출력 7×7×30
- ImageNet으로 앞 20개 conv layer를 224×224 해상도로 먼저 pretrain (top-5 88%) → detection용으로 4개 conv + 2개 FC layer 추가, 입력 해상도를 448×448로 2배 키워서 fine-tune
- 활성함수: 마지막 층은 linear, 나머지는 leaky ReLU (x>0이면 x, else 0.1x)
- **Fast YOLO**: conv layer를 24→9개, 필터 수도 줄인 경량 버전. 나머지 학습/추론 설정은 동일

### 2.3 추론 및 후처리

- 이미지 1장당 98개 박스(7×7×2)를 한 번의 network evaluation으로 예측 → R-CNN 계열(Selective Search 기준 ~2000개 proposal)보다 훨씬 적음
- grid 구조 덕분에 한 객체당 보통 박스 1개만 나오지만, 큰 객체나 경계에 걸친 객체는 여러 cell에서 중복 검출될 수 있어 **NMS로 2~3% mAP 개선**

---

## 3. 손실 함수 (핵심 설계)

Sum-squared error 기반 multi-part loss (논문 식 3). 그대로 쓰면 문제가 생겨서 3가지 트릭을 추가:

1. **좌표 손실과 confidence 손실의 가중치 분리**
   - 대부분의 grid cell에는 객체가 없어서, "객체 없음" confidence 손실이 학습을 지배해버려 학습이 불안정해짐
   - λ_coord = 5 (좌표 손실 가중치 ↑), λ_noobj = 0.5 (객체 없는 cell의 confidence 손실 가중치 ↓)
2. **w, h에 제곱근(sqrt) 적용**
   - sum-squared error는 큰 박스와 작은 박스의 오차를 동일하게 취급 → 작은 박스에서는 같은 절대 오차라도 IOU에 미치는 영향이 훨씬 큼
   - √w, √h를 예측하도록 해서 작은 박스의 오차에 상대적으로 더 큰 패널티를 주는 방식으로 "부분적으로만" 보정
   - **→ 이 부분이 완전한 해결책은 아니라고 논문이 직접 인정**(Section 2.4), 이후 연구들의 주요 개선 지점
3. **책임 할당(responsibility assignment)**
   - Cell 안의 B개 predictor 중 ground truth와 IOU가 가장 높은 것만 "책임" predictor로 지정 → 분류 손실은 객체가 있는 cell에서만, 좌표 손실은 책임 predictor에서만 계산
   - 결과적으로 predictor들이 크기/종횡비/클래스별로 자연스럽게 특화(specialization)됨

---

## 4. 학습 설정

- 데이터: PASCAL VOC 2007+2012 train/val (2012 평가 시 2007 test도 학습에 포함)
- 135 epoch, batch size 64, momentum 0.9, weight decay 0.0005
- learning rate: 1e-3→1e-2로 warm-up 후 1e-2(75 epoch) → 1e-3(30 epoch) → 1e-4(30 epoch)
  - 처음부터 높은 lr로 시작하면 gradient 불안정으로 발산
- 과적합 방지: 첫 FC layer 뒤 dropout(0.5), 랜덤 스케일/이동(최대 20%), HSV 색공간에서 exposure/saturation 랜덤 조정(최대 1.5배)

---

## 5. 실험 결과

### 5.1 실시간 detector 비교 (VOC 2007, Table 1)
- Fast YOLO: **52.7% mAP, 155 FPS** — 당시 가장 빠른 detector, 기존 실시간 detector 대비 mAP 2배 이상
- YOLO: **63.4% mAP, 45 FPS**
- 30Hz DPM: 26.1% mAP / Fast R-CNN: 70.0% mAP, 0.5 FPS(실시간 아님) / Faster R-CNN(VGG-16): 73.2% mAP, 7 FPS

→ YOLO는 "정확도 1위"가 아니라 "실시간 영역에서 정확도 압도"라는 포지셔닝.

### 5.2 오류 분석 (Fig. 4) — 여기가 중요
Fast R-CNN과 비교했을 때 오류 유형이 완전히 다름:
- **YOLO**: Localization error(정답 클래스지만 IOU 0.1~0.5) 비중이 압도적으로 큼(19.0%) → **위치를 정확히 못 맞춤**
- **Fast R-CNN**: Background error(IOU<0.1, 즉 배경을 객체로 착각) 비중이 큼(13.6%, YOLO의 약 3배)

→ 서로 다른 종류의 실수를 하므로 **YOLO로 Fast R-CNN 결과를 rescore하면 상호보완**: Fast R-CNN 단독 71.8% → 결합 시 75.0% mAP (+3.2%, Table 2)

### 5.3 VOC 2012 결과
- YOLO 57.9% mAP로 당시 SOTA보다 낮음, R-CNN(VGG-16) 수준
- **논문이 직접 명시**: "Our system struggles with small objects compared to its closest competitors." bottle, sheep, tv/monitor 등에서 R-CNN보다 8~10%p 낮음

### 5.4 일반화 성능 (Picasso, People-Art artwork 데이터셋)
- R-CNN은 자연 이미지→아트워크로 갈 때 AP가 크게 하락(Selective Search가 자연 이미지에 튜닝되어 있음)
- YOLO는 DPM처럼 객체의 크기/모양/맥락을 명시적으로 모델링하기 때문에 도메인 변화에 상대적으로 강함

---

## 6. 한계 (Section 2.4) — 후속 연구로 이어지는 지점

논문이 스스로 정리한 한계 4가지:

1. **Grid당 박스 수(B) 제한 + cell당 클래스 1개**: 강한 공간적 제약 → 같은 grid cell에 여러 객체가 몰리면(특히 **작은 객체들이 무리지어 있는 경우**, 예: 새 떼) 검출 실패
2. **드문 종횡비/구성에 대한 일반화 약함**: bbox를 데이터로부터 학습하므로 학습 데이터에 없던 형태에는 취약
3. **coarse feature 사용**: 여러 번의 downsampling을 거친 feature map에서 bbox를 예측 → **공간 해상도 손실이 작은 객체 검출에 직접적으로 불리**
4. **Loss function이 박스 크기를 충분히 반영 못함**: sum-squared error는 큰 박스/작은 박스의 절대 오차를 동일하게 취급 (sqrt trick으로 부분 완화했지만 근본 해결 아님) → 작은 박스의 작은 오차가 IOU에 훨씬 크게 영향

→ 이 4가지가 정확히 **YOLOv2의 anchor box, YOLOv3의 multi-scale prediction(FPN 스타일), 이후 세대의 다양한 loss(GIoU/CIoU 등) 도입 배경**.

---

## 7. 내 연구 방향과의 연결 (small object detection)

지금 관심사인 "모델 구조/학습 기법으로 small object detection 성능 개선"의 출발점으로 보면:

- **구조적 원인**: YOLOv1은 7×7이라는 매우 coarse한 grid + 단일 최종 feature map만 사용 → 작은 객체가 차지하는 픽셀 수가 애초에 grid resolution보다 작아지면 원천적으로 놓침. 이후 FPN, multi-scale head, high-resolution branch 등이 이 문제를 정면으로 다룸
- **손실 함수 관점**: sqrt trick은 "크기에 따른 오차 가중치 문제"에 대한 최초의(하지만 미완의) 시도. IOU 기반 loss(GIoU, DIoU, CIoU), 혹은 크기별 가중치를 명시적으로 주는 방식들의 시작점으로 읽으면 도움이 될 것
- **grid의 spatial constraint**: cell당 객체 1개 책임 원칙은 밀집/소형 객체에서 근본적 한계 → anchor-free/dense prediction, 혹은 query 기반(DETR류) 접근이 이 제약을 어떻게 풀었는지 비교해볼 만함
- 참고로 이 논문 자체가 증강(augmentation)에 대해서는 스케일/이동/HSV 정도의 기본적인 것만 쓰고 있어서, 지금까지 해온 augmentation 연구와는 결이 다른, "구조/학습기법" 계열 논문을 고를 때 대조점으로 쓰기 좋음

---

## 8. 더 볼 만한 후속 논문 (제안)

- YOLOv2/YOLO9000 — anchor box, batch norm, multi-scale training으로 이 논문의 한계를 어떻게 메웠는지
- SSD — multi-scale feature map에서 직접 예측하는 방식과 YOLOv1의 single feature map 방식 비교
- Feature Pyramid Networks (FPN) — coarse feature 문제에 대한 구조적 해법
- Focal Loss (RetinaNet) — 작은/드문 객체의 class imbalance를 loss로 다루는 접근
