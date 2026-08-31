---
title: "Small Object Detection 연구 계보"
tags: [genealogy, small-object-detection]
created: 2026-08-24
---

# Small Object Detection 연구 계보

> [!tip] 읽는 법
> 각 논문은 **[구조] → [원인이 되는 한계] → [다음 논문의 해결책]** 순서로 이어집니다. 같은 레벨(형제 논문)은 `n-1`, `n-2`처럼 번호를 매겼습니다. <mark style="background:#FF5582A6;">빨강</mark>은 이 위키 안에서 특히 완성도 높은 컨트리뷰션(이론적 증명, 직접 대조실험, 명확한 SOTA 등 근거가 뚜렷한 논문)입니다.

---

## 계보 1 — DETR 계열: end-to-end 탐지의 시작과 query 동적화

### 1. DETR
#### 구조
`##### 원제` End-to-End Object Detection with Transformers · **2020 · ECCV · JCR Q1**
CNN backbone + transformer encoder-decoder + 이분 매칭(Hungarian algorithm) 기반 집합 손실만으로 end-to-end 학습. anchor·NMS 같은 수작업 구성요소를 전부 제거.

#### 왜 이런 구조인가 (원인)
- 기존 detector는 대규모 proposal·anchor에 대한 **대리 회귀·분류 문제**로 집합 예측을 간접 처리
- 중복 예측 제거를 NMS라는 **후처리에 의존** — 예측 간 상호작용을 모델이 직접 배우지 않음

#### 이 구조의 단점
> [!warning] 한계
> - **소형 객체(APS) 성능이 Faster R-CNN 대비 유의미하게 낮음** (동률 AP 모델에서 APS −5.5~−6.1). 저자 스스로 "FPN이 준 것과 유사한 개선이 필요하다"고 인정
> - **극도로 느린 수렴**: COCO 기준 500 epoch 필요
> - Query 슬롯 수(N=100) 고정 → 100개 넘는 밀집 인스턴스에서 성능 급락

**데이터셋**: COCO val · AP 42.0 / APS 20.5 (Faster R-CNN-FPN+: 42.0 / 26.6)
**노벨티 평가**: anchor·NMS 없이 탐지를 재정의한 최초 시도. 다만 이 논문 자체는 소형 객체 문제를 "발견"만 했을 뿐 해결책은 제시하지 않음 — 그래서 후속 26편 대부분의 출발점이 됨.

---

### 2. Deformable-DETR
#### 구조
`##### 원제` Deformable DETR: Deformable Transformers for End-to-End Object Detection · **2021 · ICLR · JCR Q1** <mark style="background:#FF5582A6;">우수 컨트리뷰션</mark>
각 query가 reference point 주변 소수 sampling point만 attend하는 **multi-scale deformable attention**으로 DETR의 attention을 대체.

#### 왜 이런 구조인가 (원인 = DETR의 한계)
- DETR의 encoder self-attention은 픽셀 수에 대해 **이차(quadratic) 복잡도** → 고해상도 feature map을 아예 못 씀 → 소형 객체 열세로 직결
- **원인 → 해결의 다리**: Deformable Convolutional Networks(1917년... 아니 2017, ICCV, 계보 2 참고)의 "학습 가능한 offset으로 샘플링 위치를 입력 조건부로 변형"이라는 아이디어를 attention에 이식

#### 이 구조가 어떻게 해결했나
- Query별 key 후보를 소수로 제한(deformable) + attention weight도 학습 → DCN에는 없던 **요소 간 관계 모델링**을 유지하면서 DETR의 quadratic 비용 문제를 없앰
- FPN 없이 multi-scale feature를 attention 메커니즘 자체로 통합 (ablation에서 FPN 추가해도 성능 향상 없음을 직접 확인)

> [!success] 왜 좋은 논문인가
> **10배 적은 epoch로 DETR을 능가** (COCO AP 43.8 vs 42.0, 50 epoch vs 500 epoch), APS는 20.5→26.4로 대폭 개선. 게다가 K=1, L=1일 때 정확히 deformable convolution으로 퇴화함을 **수식으로 직접 증명**해 두 계보(CNN·Transformer)를 잇는 다리 역할을 스스로 명시. 이후 6편(2-1~2-6, 아래)의 공통 baseline이 됨 — 영향력이 실증됨.

#### 이 구조의 남은 단점
> [!warning] 한계
> - Query 수는 여전히 **고정**(300개) — 이미지마다 인스턴스 수가 1개~2000개 이상 다른데 대응 못 함
> - M=8, K=4 등 하이퍼파라미터 최적값 탐색이 제한적

**데이터셋**: COCO 2017 val · 50 epoch vs DETR 500 epoch · AP 43.8/APS 26.4 (vs DETR 42.0/20.5), 학습 GPU시간 2000→325

---

이제 Deformable-DETR의 "query 수 고정"이라는 한계를, 같은 레벨의 논문 6편이 서로 다른 방식으로 공략합니다. 두 하위 갈래로 나뉩니다.

### 2-A. 하위 갈래 A — 전역 밀도(density map)로 query 수를 정한다

#### 2-A-1. DQ-DETR (원조)
`##### 원제` DQ-DETR: DETR with Dynamic Query for Tiny Object Detection · **2024 · ECCV · JCR Q1**

**구조**: density map을 4단계로 분류 → 그 단계에 따라 query 개수(300/500/900/1500) 선택. 같은 density map으로 encoder feature와 query content·position도 함께 강화.

**왜 (Deformable-DETR의 한계)**: 고정 query 수 → 밀집 이미지에서 미검출(FN) 급증, 희소 이미지에서 오탐(FP) 급증. Query 위치도 이미지 내용과 무관한 학습된 임베딩일 뿐.

**해결**: 회귀 대신 **분류**를 택함 — "회귀 오차가 곧바로 부적절한 query 수로 이어져 학습이 불안정"(ablation에서 회귀 사용 시 AP 25.9→14.9로 급락 확인, 저자 직접 실험).

> [!danger] 이 논문 자체의 한계 (다음 논문이 풀 문제)
> - N>500 구간의 분류 정확도가 **56.6%에 불과** — 이산 분류가 세밀한 밀도 차이를 뭉갬
> - 2단계 학습이 필수 → end-to-end 아님
> - COCO 일반 벤치마크에서는 DINO-DETR보다 낮은 성능(50.2 vs 51.3)

**데이터셋**: AI-TOD-V2 · AP 25.9(DINO)→30.2(+4.3%p), AP_vt 12.7→15.3

---

#### 2-A-2. Density-Aware-DETR
`##### 원제` Density-Aware DETR With Dynamic Query for End-to-End Tiny Object Detection · **2025 · JSTARS · JCR Q1** <mark style="background:#FF5582A6;">우수 컨트리뷰션</mark>

**구조**: crowd counting 기법을 응용한 **연속 density map 회귀**(Density Focal Loss)로 query 수 결정 + static+동적 mix selection + log-ratio anchor L1 손실.

**왜 (2-A-1 DQ-DETR의 한계)**: "이산화 때문에 실제로는 크게 다른 밀도(예: 101개 vs 499개)가 같은 구간으로 뭉뚱그려지고, 2단계 학습이 필수라 end-to-end가 안 된다"고 **DQ-DETR을 직접 겨냥해 비판**.

**해결**: 분류를 연속 회귀로 대체 — DQ-DETR과 **같은 baseline에서 직접 대조 실험**해 회귀 방식의 우위를 실증.

> [!success] 왜 좋은 논문인가
> 같은 조건에서 이전 논문(DQ-DETR)을 직접 재현·대조해 "왜 내 방법이 더 나은가"를 실험으로 증명한 흔치 않은 사례. AI-TOD-v2 mAP 30.2(DQ-DETR)→32.1, End-to-end 학습 가능(2단계 학습 불필요)이라는 구조적 우위도 확보.

> [!warning] 이 논문 자체의 한계
> - Density 추정 정확도 자체는 "여전히 개선 여지가 있다"고 저자가 직접 인정 (고밀도 구간 산포 존재)
> - VisDrone2019에서 AP_vt/AP_t 지표가 불안정 — 원인을 label noise 탓으로만 돌릴 뿐 정량 검증 없음

**데이터셋**: AI-TOD-v2 · mAP 30.2(DQ-DETR)→32.1, AP75 22.3→25.9(+16.1% 상대)

---

#### 2-A-3. IG-DETR
`##### 원제` IG-DETR: Instance-Guided Dynamic Queries for Small Object Detection · **2026 · ICASSP · JCR Q2**

**구조**: DQ-DETR의 4단계보다 세분화된 **6단계 분류**(HIP) + 곱셈 대신 **덧셈 residual**(1+W) 주입(IFE) + salient seed top-K 선택(IGQ).

**왜 (2-A-1의 한계)**: 4단계 분류가 여전히 거칠다는 판단, 그리고 곱셈 마스킹은 attention 값이 0에 가까우면 원본 정보 자체가 소실.

**해결**: 6단계로 세분화, `(1+W)` 형태로 최소 원본 정보(1배)를 항상 보존.

> [!warning] 이 논문 자체의 한계
> ICASSP 4페이지 짧은 논문이라 방법론 설명이 간결, ablation도 AI-TOD-V2 단일 데이터셋. **6단계 구간 경계값 자체가 논문에 명시되지 않아 재현성이 떨어짐**. DQ-DETR 재구현 결과(29.6)가 원 논문 값(30.2)과 다른데 원인 분석 없음.

**데이터셋**: AI-TOD-V2 · AP 25.9(DINO)→30.9(+5.0%p)

---

#### 2-A-4. DQP-DETR (가장 포괄적)
`##### 원제` DQP-DETR: Object-Density-Guided Query Prioritization for Small Object Detection in UAV Imagery · **2026 · SSRN preprint(Elsevier 제출, 동료평가 전) · JCR 해당없음(preprint)**

**구조**: density map을 **encoder memory에 양방향 cross-modulation으로 직접 주입**(BCME) + 토큰 공간 투영 후 classification score와 곱해 우선순위 생성(RCS) + GT 밀도 기반 ranking loss로 그 우선순위 자체를 학습.

**왜 (2-A-1~3의 공통 한계)**: "density를 auxiliary prediction branch나 additional supervision으로만 쓸 뿐, decoder query allocation의 직접적 prior로는 안 쓴다"고 **선행 3편 전체를 겨냥해 비판**. 또한 소형 객체는 classification score 자체가 신뢰 불가능(외형 단서 약함).

**해결**: density 정보를 파이프라인 전역(encoder+decoder 양쪽)에 주입, ranking loss로 "시각적으로 정확한 density map이 반드시 좋은 top-k 선택으로 이어지지 않는다"는 간극을 직접 메움(ablation: RCS 추가 시 AP 26.1→28.2).

> [!warning] 이 논문 자체의 한계
> **SSRN 프리프린트로 동료평가를 거치지 않은 상태** — 정식 게재 여부 불확실. RCS의 하이퍼파라미터(temperature, margin) 민감도 분석 없음. FLOPs·FPS 전혀 미보고. VisDrone·AI-TOD 두 데이터셋만 검증(COCO 미검증).

**데이터셋**: VisDrone val(D-Fine-S) · AP 26.2→28.2, AP_S 18.2→20.8 / AI-TOD AP50 35.6→38.7

---

### 2-B. 하위 갈래 B — 개별 후보의 패턴·품질로 query를 정제한다

#### 2-B-1. PaQ-DETR
`##### 원제` PaQ-DETR: Learning Pattern and Quality-Aware Dynamic Queries for Object Detection · **2025 · arXiv · JCR 해당없음(arXiv)**

**구조**: 이미지 전체 밀도가 아니라 **개별 후보의 공간적 군집 패턴 + 객체다움 신뢰도**를 신호로, 유사도 클러스터링으로 중복 병합 + 품질 임계값 이하 제거.

**왜 (하위 갈래 A 전체의 다른 각도 한계)**: 밀도 수치 하나로는 "객체가 어떻게 뭉쳐 있는가"라는 공간 분포 정보를 못 담음. 게다가 기존 dynamic query 연구는 대부분 **tiny/aerial 특화 데이터셋에서만 검증**되어 일반 객체 탐지에서의 유효성이 불명.

**해결**: 클러스터링 결과 자체에 공간 분포가 암묵적으로 인코딩됨. **COCO 일반 탐지에서 검증한 유일한 사례**.

> [!warning] 이 논문 자체의 한계
> 클러스터링이 인접한 두 객체를 하나로 잘못 병합할 위험(under-segmentation)에 대한 정량 검증 부족. AI-TOD류 tiny 특화 벤치마크 미검증이라 다른 5편과 직접 비교 어려움. 수치 표가 이미지로만 렌더링되어 정확한 AP 값 확인이 어려운 상태(문서 자체 한계).

**데이터셋**: COCO val2017 (DINO 대비 일관된 개선, 정확한 수치는 원문 표 확인 필요)

---

#### 2-B-2. DQA-DETR
`##### 원제` DQA-DETR: Dynamic Query Aggregation for Oriented Object Detection in Remote Sensing Images · **2026 · JSTARS · JCR Q1** <mark style="background:#FF5582A6;">우수 컨트리뷰션</mark>

**구조**: Rotated-DINO 기반 **oriented(회전) object detection**. 유사한 고품질 query를 제거가 아니라 **병합**(aggregate)하는 QA 모듈.

**왜 (2-B-1의 한계 + one-to-one matching 고유 문제)**: One-to-one matching에서 "중복된 고품질 negative"가 gradient를 왜곡한다는 것을 **focal loss gradient를 직접 유도해(수식) 이론적으로 증명** — p>0.5 구간에서 gradient ratio가 음수로 전환됨을 Fig.3에서 시각화.

**해결**: 제거가 아니라 병합(ablation: QA 제거 시 AP50 78.29→76.89로 하락 확인).

> [!success] 왜 좋은 논문인가
> 이 위키 26편 중 **유일하게 oriented detection**을 다루고, **유일하게 문제의 원인을 손실 함수의 gradient 수식으로 직접 증명**한 사례 — 대부분의 논문이 "관찰"에 그치는 반면 이 논문은 "왜 문제인지"를 수학적으로 보임. DOTA-v1.0 mAP 75.59→78.29, DOTA-v1.5 68.79→71.53.

> [!warning] 이 논문 자체의 한계
> Rotated-NMS는 미분 불가능한 연산이라 대표 선별 자체는 학습되지 않고 고정 규칙에 의존. 밀집 중심 개수 예측이 전역 feature 통계 의존이라 극도로 밀집된 이미지에서 일반화 제한 가능성을 저자가 직접 인정. 최고 성능의 2-stage 방법(ReDet, DCFL)에는 여전히 못 미침.

**데이터셋**: DOTA-v1.0 · mAP 75.59→78.29(+2.70%p) / DOTA-v1.5 · 68.79→71.53(+2.74%p)

---

> [!info] 계보 1 종합 — Dynamic Query DETR 6편이 공통으로 보여주는 패턴
> "다중 신호를 쓴다는 것 자체"의 기여가 "그 신호를 얼마나 정교화하는가"보다 일관되게 크다(DQP-DETR의 RCS, Density-Aware-DETR의 IDE&QA 등에서 반복 확인). DQA-DETR의 query 900→2400 확장 실험(baseline mAP 붕괴)은 "query를 무작정 늘리는 것 자체가 위험하다"는 이 계열 전체의 문제의식을 가장 극적으로 보여줌.

---

## 계보 2 — CNN의 고정 샘플링 문제 (Deformable-DETR의 또 다른 뿌리)

### Deformable_Convolutional_Networks
`##### 원제` Deformable Convolutional Networks · **2017 · ICCV · JCR Q1**

**구조**: convolution/RoI pooling의 고정 정사각 grid 위치에 **학습 가능한 2D offset**을 더해, 입력 내용에 따라 수용영역이 동적으로 변형.

**왜 (기존 CNN의 한계)**: convolution은 항상 고정 위치에서 샘플링 → 동일 레이어 내 모든 위치가 동일 수용영역 → **비강체(non-rigid) 객체에 근본적으로 최적이 아님**. STN은 비싼 feature warping이 필요, Active Convolution은 offset이 정적(static) 파라미터라 위치별 적응 불가.

**해결**: 별도 conv 레이어가 위치마다 다른 offset을 예측(bilinear interpolation) → 수용영역이 객체 크기·형태에 맞춰 조정.

> [!warning] 이 논문 자체의 한계
> DeepLab과 다른 과제의 최적 deformable layer 수가 서로 다름 — 과제별 재튜닝 필요. Offset이 학습 데이터 통계에 과적합될 위험에 대한 정량 검증 없음. 배경 영역에도 큰 effective dilation이 학습되는 현상(Table 2)의 위험성이 논의 안 됨.

**데이터셋**: COCO test-dev · mAP@[0.5:0.95] Faster R-CNN 29.4→33.1(+13%), R-FCN 30.8→34.5(+12%)

**→ 이 논문의 offset 아이디어가 Deformable-DETR의 deformable attention으로 이식됨** (계보 1로 연결).

---

## 계보 3 — "학습 시에만 존재하는 auxiliary branch" — 재구성(reconstruction)으로 정보 손실을 우회하는 4대

### 3-1. SR-TOD (원조)
`##### 원제` Visible and Clear: Finding Tiny Objects in Difference Map · **2024 · ECCV · JCR Q1** <mark style="background:#FF5582A6;">우수 컨트리뷰션</mark>

**구조**: neck의 저수준 feature(P2)에서 원본 이미지를 재구성하는 self-reconstruction head를 붙이고, **원본-재구성 이미지 차이(difference map)**를 tiny object 위치의 prior로 사용.

**왜 (기존 방법의 한계)**: backbone의 downsampling은 tiny object 신호를 필연적으로 지움(AI-TOD 2~8px 객체는 feature map에서 신호가 거의 소실). GAN 기반 super-resolution은 **존재하지 않는 texture와 artifact를 만들어내(spurious)** 오히려 성능을 떨어뜨림.

**해결**: 이미지 재구성은 픽셀 단위 변화에 극도로 민감한 low-level task라, 정보가 심하게 지워진 영역일수록 복원이 어려움 → 그 실패(difference) 자체가 "여기 tiny object가 있다"는 신호. **없는 디테일을 새로 만들지 않으므로** GAN류의 부작용이 원천적으로 없음.

> [!success] 왜 좋은 논문인가
> "재구성의 실패를 신호로 쓴다"는 발상의 전환이 명확하고, 생성 모델 없이 reconstruction의 부산물만 활용하는 **가장 가벼운 방식**. 이후 3편(FFSSTDNet, ORFENet, CoLR-Det, 그리고 유사 계보의 BAFNet)이 이 원리를 각자 다른 방식으로 확장 — 계보의 시작점이 된 영향력이 확인됨.

> [!warning] 이 논문 자체의 한계
> "향후 더 정확한 difference map 구성법을 탐색하겠다"고 저자가 직접 명시. High-frequency difference map이 일부 작은 드론 객체를 오히려 흐릿하게 만드는 부작용. Transformer 계열(DINO) 개선폭(+0.2 AP)이 CNN 계열(+2.1)보다 훨씬 작음 — 아키텍처 의존성.

**데이터셋**: DroneSwarms(평균 물체 크기 7.9px) · RFLA AP 36.9→39.0(+2.1, 최고) / AI-TOD · DetectoRS AP 14.6→24.0(+9.4)

---

### 3-2. FFSSTDNet
`##### 원제` From Fuzzy Global to Clear Local: A Focus and Super-Resolution-Guided Tiny Target Detection Method for Full-Scene Images · **2026 · IEEE TGRS · JCR Q1**

**구조**: 전체 장면(full-scene) 위성 이미지를 패치로 나눠 **CFD**(경량 query로 배경 패치 필터링)로 연산 절감 + **FSR**(학습 시에만 존재하는 SR 보조 브랜치)로 backbone이 고해상도 특징을 학습하도록 유도.

**왜 (3-1의 한계 + full-scene 고유 문제)**: SR-TOD와 유사하게 auxiliary branch를 쓰지만, full-scene 이미지는 배경(RONI) 비중이 훨씬 커서 **연산 비용 문제가 SR-TOD보다 심각** — 이 논문은 그래서 SR-TOD가 안 다룬 "연산 가속" 축까지 함께 다룸.

**해결**: FSR은 재구성 오차 자체를 attention prior로 안 쓰고 backbone feature 품질을 간접적으로 끌어올리는 **정규화 역할**만 함(SR-TOD와의 핵심 차이). CFD는 grid 단위 이진 분류로 배경 패치를 걸러 RONI 연산 비용을 줄임.

> [!warning] 이 논문 자체의 한계
> CFD가 **대규모 라벨링된 데이터셋에 강하게 의존** — 사전 라벨된 관심 영역이 필요하다고 저자 명시. FSR의 업샘플링 전략이 밀집·저대비 영역에서 재구성 아티팩트를 낼 수 있음(저자 인정). CFD가 실제 타겟 패치를 잘못 배제할 위험의 정량적 recall 손실 분석 없음.

**데이터셋**: FAIR1M · mAP 43.88→46.25 / DOTA-v2.0 · mAP 53.65→56.75

---

### 3-3. ORFENet
`##### 원제` Tiny Object Detection in Remote Sensing Images Based on Object Reconstruction and Multiple Receptive Field Adaptive Feature Enhancement · **2024 · IEEE TGRS · JCR Q1**

**구조**: FCOS(P2) 기반. 학습 시에만 존재하는 **ORB**(이진 foreground/background 마스크 재구성) + fine-grained/close-range/distant-context 세 receptive field를 동적 가중합하는 **MRFAFEM**.

**왜 (3-1, 3-2의 한계)**: SR-TOD의 difference map, FFSSTDNet의 FSR 모두 재구성 target이 원본 이미지(고차원)라 학습이 무거움. 또 receptive field 크기의 최적값이 상황마다 다른데 기존 방법은 이를 동적으로 조정하지 않음.

**해결**: 재구성 target을 **이진 마스크로 단순화** — "이 영역이 tiny object인지"만 예측하도록 제약을 낮춤(SR-TOD의 difference map보다 훨씬 단순한 self-supervision). MRFAFEM은 3개 receptive field를 Softmax 가중치로 동적 결합.

> [!warning] 이 논문 자체의 한계
> "제안 방법의 복잡도가 상대적으로 높다"고 저자가 결론에서 직접 인정. 일부 클래스(AI, ST)에서 다른 SOTA가 더 우세하나 원인 분석 없음. 재구성 target이 이진 마스크뿐이라 "정보 손실 정도"가 아니라 "존재 여부"만 다룬다는 표현력 제한을 논문이 스스로 논의하지 않음.

**데이터셋**: AI-TODv2 · AP 17.3→24.8(36 epoch) / LEVIR-Ship · AP50 83.3(SOTA)

---

### 3-3b. BAFNet (같은 계보의 변형 — 경계에 적용)
`##### 원제` Boundary-Aware Feature Fusion With Dual-Stream Attention for Remote Sensing Small Object Detection · **2025 · IEEE TGRS · JCR Q1**

**구조**: 최고레벨 semantic feature에서 전경(FPAM)·배경(BPAM) attention을 **동시에** 생성하는 DSAM + Laplacian pyramid GT 경계 맵으로 supervision하는 **Boundary-Aware Branch**(학습시에만 관여, 세 번째 auxiliary branch 변형).

**왜 (기존 attention 계열 + 3-1~3-2 계보의 다른 각도 한계)**: 기존 attention 방법은 전경 강조에만 집중하고 배경을 명시적으로 모델링 안 함. Cross-scale fusion 중 boundary-blurring이나 over-erosion 발생.

**해결**: 배경 attention을 전경의 여집합으로 유도(별도 파라미터 없이 경량). 경계를 별도 auxiliary supervision으로 명시화(ablation: boundary branch 추가 시 AP_s 21.0→22.4).

> [!warning] 이 논문 자체의 한계
> AI-TOD AP50(59.8)이 경쟁 SOTA(63.7)보다 낮음 — 엄격한 지표(AP75, AP_vt)에서만 우위. BPAM이 FPAM의 단순 여집합이라 별도 학습 없이 전적으로 종속. AP_l(대형 객체)이 결합 시 오히려 하락(44.5→43.2)했는데 논의 없음.

**데이터셋**: AI-TOD · AP 30.2→30.5, AP_vt 12.8→16.6(+3.8%p) / VisDrone · AP_s 17.5→22.4(+4.9%p)

---

### 3-4. CoLR-Det (네 번째 변형, 가장 정식화됨)
`##### 원제` CoLR-Det: Collaborative Latent Restoration for Small Object Detection in Low-Resolution Remote Sensing Images · **2026 · arXiv · JCR 해당없음(arXiv)** <mark style="background:#FF5582A6;">우수 컨트리뷰션</mark>

**구조**: SR을 이미지 복원이 아니라 **학습 시에만 관여하는 latent 정규화**로 재정의(추론 시 SR 브랜치 완전 제거) + saliency 기반 **비파괴적(non-destructive) token routing**.

**왜 (3-1~3-3의 공통 원리를 가장 명시적으로 정식화)**: "restoration-first 패러다임"은 SR이 조밀한 텍스처·edge fidelity를 우선하는 반면 탐지는 sparse instance-level semantic에 의존한다는 **근본적 목표 불일치**를 가정 — 이 위키 4편 중 이 문제를 가장 명확한 명제("복원이 탐지를 지배해서는 안 된다")로 정리.

**해결**: SR 브랜치를 "출력"이 아니라 "학습 시 encoder에 가하는 보조 제약"으로 재정의(ablation: SR 브랜치 추가만으로 AP 0.766→0.816). Saliency 라우팅은 저saliency 토큰도 **제거가 아니라 우회**시켜 정보 흐름을 유지 — 오탐지된 실제 소형 객체 토큰이 완전히 사라지지 않음(ablation: saliency+SR 결합 시 AP_s 0.552→0.649로 대폭 개선).

> [!success] 왜 좋은 논문인가
> 3-1~3-3이 각자 다른 방식으로 "우회"했던 원리를 **하나의 명제로 정식화**하고, "제거가 아니라 우회"라는 설계 철학을 raw token 단위까지 일관되게 적용. Ablation 표(Table IV, VI)에서 각 구성요소의 기여를 명확히 분리해 보임.

> [!warning] 이 논문 자체의 한계
> 아직 arXiv 프리프린트(정식 게재 여부 불명). 학습된 saliency가 초기 학습 단계에서 신뢰하기 어려울 수 있다고 저자가 인정하나 정량 분석 없음. 고정된 2× bicubic downsampling에서만 검증 — 다양한 열화 정도 미검증. 저자 스스로 "간극을 좁혔을 뿐 완전히 해소 못함"이라고 결론에서 인정.

**데이터셋**: NWPU VHR-10-Split(2× downsampling) · AP 0.766→0.836(+7.0%p), AP_s 0.487→0.649(+16.2%p)

---

> [!info] 계보 3 종합
> SR-TOD(재구성 실패=신호) → FFSSTDNet(연산 가속 결합) → ORFENet(재구성 target 단순화) → BAFNet(경계로 확장) → CoLR-Det(원리의 명시적 정식화)로 이어지는 흐름은, "학습에만 관여하고 추론에선 사라지는 auxiliary branch"라는 하나의 설계 패턴이 4~5번 반복해서 재발견·정교화되는 과정을 보여줍니다. 다음 연구 방향으로는 이 패턴을 **detection 이외의 task(segmentation 경계, anomaly 정상/비정상 경계)**로 확장하거나, auxiliary branch의 target 자체를 학습 가능하게(현재는 전부 고정 target) 만드는 시도가 비어 있습니다.

---

## 계보 4 — 주파수 도메인으로 정보 손실을 우회하는 갈래

### 4-1. FANet
`##### 원제` FANet: Frequency-Aware Attention-Based Tiny-Object Detection in Remote Sensing Images · **2025 · Remote Sensing(MDPI) · JCR 미확인**

**구조**: Faster R-CNN 기반. FPN(P2)과 RoI head에 2D-DFT/2D-DCT 기반 attention 모듈(MSFFEM, CAREM) + 다중 방향 flip 증강(SAS).

**왜 (spatial-domain 방법의 공통 한계)**: FPN·PANet·DetectoRS 등은 spatial-domain 정보에만 의존 — tiny object의 근본적으로 약한 feature 문제를 해결 못함. 심한 class imbalance(vehicle 88.22% vs windmill 0.1% 미만)도 병존.

**해결**: tiny object는 patch 단위 FFT 스펙트럼에서 다른 영역과 뚜렷이 구분되는 고유 주파수 응답을 보인다는 관찰 → MSFFEM으로 이를 attention화, CAREM으로 RoI 내 배경/저주파 성분을 한 번 더 필터링.

> [!warning] 이 논문 자체의 한계
> Very tiny object는 주변 문맥 의존 자체가 필수적인데, 주파수 강화는 경계를 부각시킬 뿐 없던 정보를 만들어낼 수 없음(구조적 한계). 학습된 주파수 가중치가 데이터셋 통계에 최적화되어 도메인 전이 시 부적합 위험을 저자도 인정. CAREM 도입으로 FPS 9~19% 저하.

**데이터셋**: AI-TOD · AP 20.6→24.8, AP50 50.4→58.1(SAS 포함)

**→ 이후 UAV-DETR이 유사하게 주파수 도메인(FFT/IFFT)을 다르게(필터링 방식) 활용** (계보 6 참고).

---

## 계보 5 — 정보이론으로 "어디를 강조할지"를 직접 계산

### Feature_Info_Driven_Gaussian
`##### 원제` Feature Information Driven Position Gaussian Distribution Estimation for Tiny Object Detection · **2025 · CVPR · JCR Q1**

**구조**: 픽셀 단위 **정보량(Shannon entropy 기반 encoding cost)**을 비지도로 계산하는 information map(PFIM) + 객체 위치·크기 기반 Gaussian Mixture(PGDP)를 동시 사용.

**왜 (SR-TOD류 계보의 한계 + attention 계열의 한계)**: SR-TOD조차 "복원 이미지 품질에 의존하고, 복원 과정의 다운샘플링이 difference map 정보를 다시 훼손"한다고 지적. 기존 attention은 heuristic한 map 생성에 의존.

**해결**: 별도 복원 네트워크나 heuristic 없이, **정보이론적으로 정의된 양(encoding cost)을 직접 최소화**해서 σ를 추정 — "왜 이 영역을 강조하는지"에 수학적 근거를 부여.

> [!warning] 이 논문 자체의 한계
> RFLA와 결합 시 다른 baseline 대비 gain 폭이 뚜렷이 작음(AI-TOD +0.9 vs DetectoRS +9.7) — 방법의 일관성에 의문. σ·PGDP 계산에 따른 추가 연산 오버헤드 비교가 전혀 없음. 지상 시점(SODA-D류) 검증 없음.

**데이터셋**: AI-TOD · AP 14.6(DetectoRS)→24.3(+9.7, 이 위키 전체에서 손꼽히는 gain)

---

## 계보 6 — 연산 가속 / 서브영역 국소화: "어디를 볼지 좁힌다"

### 6-1. QueryDet (원조)
`##### 원제` QueryDet: Cascaded Sparse Query for Accelerating High-Resolution Small Object Detection · **2022 · CVPR · JCR Q1** <mark style="background:#FF5582A6;">우수 컨트리뷰션</mark>

**구조**: 저해상도 feature에서 tiny object 위치를 먼저 예측(query) → 그 위치 주변에만 고해상도 feature에서 sparse convolution 적용(Cascade Sparse Query).

**왜 (dense 연산의 근본 문제)**: FPN에 고해상도 레벨(P2) 추가 시 연산량 300% 증가, 추론 속도 13.6→4.85 FPS. 소형 객체는 이미지의 극히 일부에만 존재 → dense 연산 대부분이 배경에 낭비. AutoFocus는 image pyramid라 backbone 중복 연산 발생, PointRend는 point-wise라 context 부족.

**해결**: 연산 비용을 소형 객체가 실제 존재하는 sparse 영역에만 비례하게 만듦(cascade 구조로 레벨마다 key 수 폭증 방지).

> [!success] 왜 좋은 논문인가
> "정확도 개선"이 아니라 **"동일 정확도를 더 빠르게"**라는 이 위키에서 유일한 목표 설정이 명확하고, 실제로 정확도 손실 거의 없이 연산 비용을 74%→1%로 줄임(FPS 4.85→14.88). Feature 강화 계열과 완전히 직교적이라 결합 여지가 뚜렷.

> [!warning] 이 논문 자체의 한계
> Query head가 위치를 맞춰도 detection head 자체가 국소화에 실패하는 case 존재. 하이퍼파라미터(query 시작 레벨, threshold, context 크기)에 성능-속도 트레이드오프가 민감.

**데이터셋**: COCO(RetinaNet) · AP 38.53→38.36(정확도 유지), FPS 4.85→14.88 / VisDrone FPS 1.16→2.75

**→ 같은 사상(coarse-to-fine)이 FFSSTDNet의 CFD(패치 단위), CDATOD-Diff(양성 샘플 단위, 계보 7), YOFOR(이미지 레벨, 아래)로 층위를 바꿔가며 반복됨.**

---

### 6-2. YOFOR
`##### 원제` YOFOR: You only focus on object regions for tiny object detection in aerial images · **2026 · Neural Networks(Elsevier) · JCR Q1**

**구조**: coarse detection 결과를 클러스터링해 객체 밀집 서브영역을 크롭하는 **ALSM**(비지도) + recursive Gaussian filter로 서브영역 배경을 흐리는 **FEM** + tail class를 공간 semantic 보존하며 복제하는 **CBM**.

**왜 (6-1과 다른 층위의 같은 문제 + long-tailed 문제)**: 항공 이미지는 객체가 전체 면적의 25%에만 밀집(75%는 배경). 균일 crop/tiling은 배경까지 함께 처리해 간섭을 늘림. 게다가 이 위키에서 유일하게 **long-tailed 클래스 불균형**을 다룸.

**해결**: QueryDet(feature pyramid 레벨)·FFSSTDNet(patch 레벨)과 달리 **이미지/픽셀 공간 레벨**에서 클러스터링·크롭. FEM은 배경을 "선명하게"가 아니라 의도적으로 "흐리게" 만들어 주의를 객체로 유도(SR 계열과 정반대 발상). CBM은 "자동차를 바다 위에 복제"하는 semantic 모순을 피하며 tail class 복제.

> [!warning] 이 논문 자체의 한계
> ALSM의 coarse detection이 애초에 실패하면(recall 실패) 이후 파이프라인 전체가 기회조차 얻지 못함 — 이 영향을 정량화하지 않음. CBM의 tail class 임계값(7%)이 고정값이라 데이터셋마다 재분석 필요(저자도 인정). 일부 표에서 수치 정합성 문제(노트 자체가 원문 재확인 필요라고 명시).

**데이터셋**: VisDrone val · AP 19.1(YOLOv8s)→52.8(+33.7%p) / AI-TOD val · AP 21.8→70.2(+48.4%p, 이 위키 전체 최대 폭)

---

## 계보 7 — Label assignment: "어떤 걸 양성 샘플로 볼지"를 동적으로

### 7-1. Unc-SOD
`##### 원제` Unc-SOD: An Uncertainty Learning Framework for Small Object Detection · **2026 · IEEE TIP · JCR 미확인**

**구조**: RPN에 uncertainty branch 추가 — 예측 박스를 Gaussian으로 모델링해 instance-level uncertainty를 IoU 대신 동적 positive 기준으로 사용 + 두 pyramid level feature 융합(Perception-and-Interaction).

**왜**: 고정 IoU 임계값(≥0.7)으로 나누면 작은 객체는 기준 만족 prior가 극히 적음(IoU 0.8 이상 prior조차 학습 진행될수록 오히려 타겟에서 멀어지는 역설 관찰). 부분 가림·모션 블러로 GT 라벨링 자체가 모호(data-level uncertainty). P2에서만 RoI Align하지만 실제 proposal 95%는 P3~P5에서 나옴(hierarchy-level uncertainty).

**해결**: 확신도 높은 인스턴스는 기준 완화, 애매한 인스턴스는 엄격화 — "모두에게 동일 기준"이라는 근본 원인 자체를 제거.

> [!warning] 이 논문 자체의 한계
> Aleatoric uncertainty만 다룸, epistemic uncertainty는 향후 과제로 남김(저자 명시). 밀집 가림 상황에서 NMS 이후에도 중복 예측 남음.

**데이터셋**: SODA-D AP 28.9→31.0 / SODA-A AP 32.5→34.8(SOTA)

---

### 7-2. CDATOD-Diff
`##### 원제` Vision-Language Guided Semantic Diffusion Sampling for Small Object Detection in Remote Sensing Imagery · **2025 · Remote Sensing(MDPI) · JCR Q2**

**구조**: CLIP의 이미지-텍스트 의미 정보를 조건으로 diffusion denoising 과정을 통해 GT 박스 주변 anchor 샘플링 포인트를 생성 + corner distance·IoU를 크기 적응적으로 가중합한 BC-IoU loss.

**왜**: 고정 grid 형태의 균일 샘플링이 소형 객체와 유효한 대응을 못 만듦. IoU 기반 손실은 미세한 위치 오차에 극도로 민감한 반면, 중심점 거리 손실은 박스 크기와 무관하게 0이 되어버림.

**해결**: RFLA의 Gaussian 수용영역 매칭을 계층적으로 확장하고, 여기에 **CLIP의 크로스모달 의미 정보를 diffusion 조건으로 결합** — 이 위키에서 VLM을 label assignment에 결합한 첫 사례. 순수 기하학적 거리(WDS)만으로 구분 안 되는 "의미적으로 타당한" 위치를 반영.

> [!warning] 이 논문 자체의 한계
> 추론 속도·파라미터 수 등 연산 비용 지표가 전혀 보고되지 않음. SAR 영상은 CLIP의 원 학습 분포(자연 이미지)와 크게 달라 프롬프트-이미지 정렬 품질 검증이 없음.

**데이터셋**: AI-TOD · AP 16.3(RFLA)→19.4 / MSAR-1.0 · AP 63.4→64.1

---

## 계보 8 — 아키텍처 경량화: 정확도와 속도를 함께

### 8-1. LSOD-YOLO
`##### 원제` Precision and speed: LSOD-YOLO for lightweight small object detection · **2025 · Expert Systems With Applications · JCR 미확인**

**구조**: YOLOv8s의 **P5 헤드 제거 + P2 헤드 추가**(cross-layer connection, LCOR) + SPPFL·C2f-N·Dysample.

**왜**: P2 헤드를 단순 추가하면 헤드 3→4개로 늘어 연산량 증가. 기존 정확도 개선 기법 대부분(TPH-YOLOv5 등)은 파라미터를 늘리는 방향으로만 접근. 경량 backbone(YOLO-S 등)은 정확도가 떨어짐.

**해결**: "헤드 하나를 빼고 하나를 더하는" 상쇄 구조 — 연산량 증가를 원천 제거하고, P5가 담당하던 semantic 정보 손실은 cross-layer connection으로 보완.

> [!warning] 이 논문 자체의 한계
> P5 제거가 대형 객체 검출을 희생시킬 가능성을 직접 검증하지 않음. VisDrone 특화 설계가 다른 도메인(COCO류)에 전이될지 불확실.

**데이터셋**: VisDrone2019 · mAP0.5 34.5%(YOLOv8s)→37.0%, Params 11.0M→3.8M(−65.5%)

---

### 8-2. FFCA-YOLO
`##### 원제` FFCA-YOLO for Small Object Detection in Remote Sensing Images · **2024 · IEEE TGRS · JCR Q1**

**구조**: FEM(atrous multibranch) + FFM(BiFPN + 채널별 학습 가중 CRC) + SCAM(GCNet/SCP 계열 전역 attention), PConv 기반 경량판(L-FFCA-YOLO).

**왜**: 얕은 layer의 불충분한 feature 표현, 배경 혼동, 정확도-속도 트레이드오프.

**해결**: 세 모듈이 각각 지역 문맥, 스케일 간 융합, 전역 문맥을 담당 — "다른 논문들이 대체로 한두 지점만 건드리는 것과 달리 세 지점을 한 프레임워크에서" 다룸. 경량화는 LSOD-YOLO(구조 재배치)와 달리 **PConv로 연산 층위에서** 접근.

> [!warning] 이 논문 자체의 한계
> Gaussian noise·stripe noise에 강건성이 매우 취약(저자 명시). 지상/항공 기반 데이터셋에서만 검증, 우주 기반 원격탐사는 미검증.

**데이터셋**: VEDAI · mAP50 0.723→0.748 / AI-TOD · mAP50 0.537(HANet)→0.617 / USOD · mAP50 0.873→0.909

---

### 8-3. RTP-Net
`##### 원제` Collaborative Optimization of Receptive Field and Texture Preservation for Remote Sensing Small Object Detection · **2026 · JSTARS · JCR Q1** <mark style="background:#FF5582A6;">우수 컨트리뷰션</mark>

**구조**: 대·소 커널 병렬 depthwise convolution(GLEM) + 다중 pooling 기반 texture 복원(AWEM) + 채널·공간 attention(MSAF).

**왜**: "수용영역 확장과 texture 보존이 근본적으로 상충한다"는 관찰 — large-kernel이나 깊은 다운샘플링은 필연적으로 공간 스무딩(고주파 texture 응답 약화)을 동반. Frequency/wavelet 접근(FANet류)은 필터 설계가 까다롭고 GSD 일반화가 어려움.

**해결**: 대커널 브랜치가 전역 문맥을, 소커널 브랜치가 texture를 각자 전담 — **단일 수용영역이 감당하던 trade-off 자체를 설계 단계에서 제거**.

> [!success] 왜 좋은 논문인가
> LSOD-YOLO·FFCA-YOLO가 "경량화 vs 정확도"의 균형 관리에 그친 것과 달리, **정확도 개선과 GFLOPs 27.2% 감소를 동시에** 달성한 이 위키의 유일한 사례. DIOR mAP 83.3(YOLOv8n)→84.1, NWPU VHR-10 94.57(GLSANet)→95.3.

> [!warning] 이 논문 자체의 한계
> NWPU VHR-10 Bridge 클래스에서 GLSANet(97.51%)에 뚜렷이 못 미침(86.1%), 원인 분석 없음. 대·소 커널 조합(5×5/7×7)이 고정값이라 근거가 약함. 순수 CNN 기반, 단일 모달리티 국한(저자가 향후 과제로 명시).

**데이터셋**: DIOR mAP 83.3→84.1 / AI-TOD mAP50:95 0.148(QueryDet)→0.166 / GFLOPs −27.2%

---

## 계보 9 — End-to-end 구조: RT-DETR을 원격탐사로

### UAV-DETR
`##### 원제` UAV-DETR: Efficient End-to-End Object Detection for Unmanned Aerial Vehicle Imagery · **2025 · arXiv · JCR 해당없음(arXiv)**

**구조**: RT-DETR 기반. 공간+주파수 도메인을 함께 쓰는 3모듈(MSFF-FE, FD, SAC) + Inner-SIoU loss.

**왜**: DETR 계열은 자연 이미지 중심 설계라 UAV 특유의 극소형 객체·가림에 그대로 적용 어려움. 전통적 fusion·다운샘플링은 고주파 성분을 쉽게 소실. 단순 합/concat 융합은 레벨 간 semantic gap으로 공간적 misalignment 발생.

**해결**: MSFF-FE(FFT→정제→IFFT)로 fusion 경로에 주파수 정보 직접 삽입. SAC는 학습된 offset으로 grid sampling을 미분 가능하게 워핑해 정렬.

> [!warning] 이 논문 자체의 한계
> GFLOPs 증가(R18: 60→77), FPS 실측 저하 뚜렷(R18 183→124, 약 32% 감소) — 논문은 이를 "실시간성을 대체로 유지"라고 완곡 표현. 모델이 노이즈 있는 무관 영역에 잘못 집중하는 현상을 향후 과제로 남김. 엣지/임베디드 배포 실험 없음.
> 
> ⚠️ 이 논문 PDF에서 프롬프트 인젝션 시도(리뷰어에게 긍정 평가를 요구하는 은닉 지시문)가 발견되어 무시하고 처리했다는 기록이 위키 노트에 있습니다.

**데이터셋**: VisDrone(RT-DETR-R18) · AP 26.7→29.8(+3.1%p), AP50 44.6→48.8(+4.2%p)

---

## 데이터셋 색인 (자주 등장하는 벤치마크)

| 데이터셋 | 성격 | 등장 논문 |
|---|---|---|
| **AI-TOD / AI-TOD-v2** | 극소형 객체(2~32px) 전용, 8클래스 | SR-TOD, ORFENet, DQ-DETR, Density-Aware-DETR, IG-DETR, DQP-DETR, FANet, Feature_Info_Driven_Gaussian, CDATOD-Diff, RTP-Net, BAFNet, YOFOR |
| **VisDrone2019** | 드론뷰, 10클래스 | LSOD-YOLO, UAV-DETR, DQP-DETR, YOFOR, BAFNet |
| **SODA-D / SODA-A** | 지상/항공 소형객체 | Unc-SOD, Detection_Oriented_Rectification |
| **DOTA-v1.0/v1.5/v2.0** | 원격탐사 회전 객체 | DQA-DETR, YOFOR, FFSSTDNet |
| **DroneSwarms** | anti-UAV, 자체 제안(SR-TOD), 평균 7.9px | SR-TOD |
| **COCO** | 일반 객체 탐지 | DETR, Deformable-DETR, DQ-DETR(비교용), PaQ-DETR |
| **VEDAI / USOD** | 차량 탐지(항공) | FFCA-YOLO |
| **NWPU VHR-10** | 원격탐사 10클래스 | RTP-Net, CoLR-Det |
| **MSAR-1.0** | SAR 영상 | CDATOD-Diff |
| **FAIR1M** | full-scene 위성, 대형 이미지 | FFSSTDNet |
| **DIOR** | 원격탐사 20클래스 | RTP-Net |
| **LEVIR-Ship** | 선박 탐지 | ORFENet |

---

## 종합 — 새 연구 방향을 떠올리기 위한 빈틈

> [!question] 아직 비어 있는 조합
> 1. **계보 3(auxiliary reconstruction branch) × 계보 6(연산 가속)**: FFSSTDNet만 두 축을 동시에 다뤘을 뿐, ORFENet·BAFNet·CoLR-Det는 연산 가속을 고려하지 않음 — "학습에만 존재하는 branch"를 QueryDet류 sparse computation과 결합하면?
> 2. **계보 3의 원리를 detection 밖으로**: "재구성 실패가 곧 신호"라는 아이디어는 segmentation 경계, anomaly의 정상/비정상 경계처럼 "라벨 자체가 애매한" 다른 task에도 이식 가능해 보이나 아직 시도 없음.
> 3. **Dynamic Query 하위 갈래 A(밀도)와 B(패턴/품질)의 결합**: 지금까지는 밀도 아니면 패턴/품질 중 하나만 신호로 씀 — 둘을 동시에 쓰는 시도가 없음.
> 4. **RFLA(ECCV 2022)**: SR-TOD, Unc-SOD, CDATOD-Diff, Feature_Info_Driven_Gaussian, ORFENet, RS-TOD, BAFNet 등 절반 가까운 논문이 baseline/비교 대상으로 직접 인용하지만, 이 위키에 아직 노트가 없는 핵심 선행 연구 — 최우선 추가 후보.
> 5. **주파수 도메인(계보 4·9) × 재구성 계보(계보 3)의 결합**: FANet/UAV-DETR의 FFT 필터링과 SR-TOD류의 재구성 실패 신호를 함께 쓰는 논문 없음.

