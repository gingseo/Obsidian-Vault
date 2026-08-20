---
title: "Small/Tiny Object Detection MOC"
tags: [moc]
task: small-object-detection
created: 2026-08-04
updated: 2026-08-19
---

# 이 분야가 다루는 핵심 질문
- 작은/타이니 객체는 픽셀 수가 적어 feature가 약하고 downsampling 과정에서 정보가 쉽게 사라진다 — 이 약한 feature를 어떻게 보강할 것인가?
- 기존 detector(anchor, label assignment, loss)는 일반 크기 객체를 기준으로 설계되어 있다 — 작은 객체에 맞게 이 파이프라인을 어떻게 다시 조정할 것인가?
- 성능 개선은 대부분 연산량 증가를 동반한다 — 실제 배포(엣지/UAV) 관점에서 정확도와 경량화를 동시에 만족할 수 있는가?

# 지금까지 다룬 흐름
지금까지 읽은 13편 중 12편은 "기존 detector(YOLO, Faster R-CNN, RT-DETR, FCOS)에 plug-in 모듈을 추가"하는 방식이고, 나머지 1편([[DETR]])은 이 위키에서 처음 다루는 순수 foundational 논문이다. 개입 지점에 따라 아래 계열로 나뉜다. 자세한 축별 비교는 [[Small_Object_Detection_Approaches]] 참고.

**Foundational 계열** — 이후 흐름 전체의 출발점이 된 원조 논문.
- [[DETR]] — anchor·NMS 없이 객체 탐지를 직접적인 집합 예측으로 재정의한 최초의 end-to-end transformer 탐지기. [[UAV-DETR]]이 기반하는 RT-DETR의 계보상 원조에 해당하며, "소형 객체 성능이 약하다"는 한계를 스스로 처음 명시한 논문이기도 하다 — 이 위키의 feature 강화 계열 다수가 정확히 이 한계를 다른 아키텍처(YOLO, R-CNN)에서 다루고 있다는 점에서, DETR 계열에도 동일 문제의식이 적용될 여지가 있는지가 흥미로운 교차점.

**Feature 강화 계열** — 약한 feature 자체를 보강. 신호를 얻는 소스가 서로 다르다.
- [[SR-TOD]] — self-reconstruction의 difference map을 신호로 사용. 생성 모델 없이 reconstruction의 부산물만 활용하는 가장 가벼운 방식.
- [[Feature_Info_Driven_Gaussian]] — 정보이론 관점의 information map + 위치 기반 Gaussian Mixture map을 동시에 사용. sr-tod와 달리 "어디를 강조할지"를 명시적으로 모델링.
- [[FANet]], [[UAV-DETR]] — 둘 다 주파수(DFT/FFT) 도메인 정보를 쓴다는 공통점이 있다. 원격탐사(FANet)와 드론뷰(UAV-DETR) 각각의 항공 이미지 특성(반복적 텍스처)과 관련 있을 가능성.
- [[RS-TOD]] — 공간 attention(RSAM) + 전용 고해상도 헤드 추가. FANet/SR-TOD처럼 기존 헤드를 보강하는 대신 헤드 자체를 새로 만든다는 점에서 다른 접근.
- [[Detection_Oriented_Rectification]] — 다른 feature 강화 논문들과 달리 "복원 목표"가 pixel fidelity가 아니라 탐지 지향적(task-oriented)이라는 점이 핵심 차별점. 열화 패턴을 명시적으로 모델링하는 첫 시도.
- [[FFSSTDNet]] — sr-tod와 유사하게 학습 시에만 존재하는 auxiliary reconstruction branch(FSR)를 쓰지만, 재구성 오차 자체를 attention prior로 쓰지 않고 backbone feature 품질을 간접적으로 끌어올리는 정규화 역할만 한다는 점에서 구별됨. Full-scene 위성 이미지 특유의 RONI(배경) 연산 비용 문제를 CFD 모듈로 별도 해결.
- [[ORFENet]] — FFSSTDNet과 마찬가지로 학습 시에만 존재하고 추론 시 완전히 제거되는 auxiliary reconstruction branch(ORB)를 쓰지만, 재구성 target이 원본 이미지가 아니라 GT 박스 기반 이진 foreground/background 마스크라는 점에서 SR-TOD의 difference map보다 훨씬 단순한 self-supervision. 여기에 fine-grained/close-range/distant-context 세 receptive field를 동적 가중합하는 MRFAFEM을 더해, "정보 손실 억제"와 "다중 receptive field 활용"을 한 프레임워크에서 함께 다룬 첫 사례.

**연산 가속(sparse computation) 계열** — 이번 처리에서 새로 생긴 갈래. "어디를 계산할지/어디를 양성 샘플로 볼지"를 좁혀 연산을 줄인다는 공통점이 있다.
- [[QueryDet]] — 저해상도 feature 예측으로 고해상도 sparse convolution 위치를 좁히는 Cascade Sparse Query. 정확도 손실 없이 고해상도 feature 연산 비용을 74%→1%로 절감. FPN 레벨 간 coarse-to-fine의 원조 격.
- [[FFSSTDNet]]의 CFD 모듈도 이 갈래와 문제의식이 겹친다 — patch 단위로 배경을 걸러낸다는 점에서 QueryDet과 구현 층위만 다를 뿐 동일한 coarse-to-fine 사상을 공유.
- [[CDATOD-Diff]] — "어디를 계산할지"가 아니라 "어디를 양성 샘플로 볼지"를 diffusion으로 정제한다는 점에서 연산 가속 계열과는 다른 축이지만, QueryDet과 결합 가능성이 있는 직교적 설계로 함께 묶어 둠.

**Label assignment/sampling 계열**
- [[Unc-SOD]] — 유일하게 feature가 아니라 "어떤 prior를 positive로 볼지" 자체를 동적으로 바꾼다. Feature 강화 계열과 직교적이라 결합 가능성이 있음(논문 자체도 Cascade R-CNN 등에 이식해 효과 유지 확인).
- [[CDATOD-Diff]] — RFLA의 Gaussian receptive field 매칭을 계층적으로 확장하고, 여기에 CLIP의 크로스모달 의미 정보를 diffusion denoising 조건으로 결합해 "의미적으로 타당한" 양성 샘플을 생성. 이 위키에서 VLM(CLIP)을 label assignment에 결합한 첫 사례.

**아키텍처 경량화 계열**
- [[LSOD-YOLO]] — 유일하게 "성능 개선"보다 "성능 유지하며 경량화"가 목표. 파라미터 65.5% 감소.

**End-to-end 구조 계열**
- [[UAV-DETR]] — 유일한 DETR(anchor-free, NMS-free) 계열. 위 주파수 도메인 활용과 별개로, 구조 자체가 다른 논문들(대부분 YOLO/R-CNN/FCOS 기반)과 궤를 달리한다.

# 이 분야를 관통하는 개념
- [[Self_Reconstruction_Difference_Map]] — sr-tod의 핵심 기여. reconstruction 기반 접근의 시작점.
- [[Frequency_Domain_Feature_Enhancement]] / [[Frequency_Domain_Feature_Attention]] — 주파수 도메인 신호를 쓰는 두 논문(uav-detr, fanet)에서 각각 등장. 아직 하나로 통합할지는 미결정 — 두 논문의 메커니즘이 미묘하게 다름(전자는 FFT/IFFT 기반 필터링, 후자는 DFT/DCT 기반 attention).
- [[Gaussian_Box_Uncertainty_Modeling]] / [[Position_Gaussian_Saliency_Map]] — 둘 다 Gaussian 분포로 뭔가를 모델링하지만 대상이 다르다(박스 좌표 불확실성 vs 위치 saliency). 혼동하지 않도록 주의.
- [[Perception_And_Interaction]] — unc-sod의 핵심 기여, 두 pyramid level 간 feature 융합.
- [[Degradation_Aware_Rectification]] — detection-oriented-rectification의 핵심 기여.
- [[Remote_Sensing_Attention_Module]] — rs-tod의 핵심 기여.
- [[Lightweight_Cross_Layer_Output_Reconstruction]] — lsod-yolo의 핵심 기여.
- [[Cascade_Sparse_Query]] — querydet의 핵심 기여. 연산 가속 계열의 시작점.

# 비교 문서
- [[Small_Object_Detection_Approaches]]

# 아직 못 채운 빈틈
- unc-sod, sr-tod, cdatod-diff, feature-info-driven-gaussian, orfenet 등 다수 논문에서 반복적으로 비교 대상·관련 연구로 언급된 RFLA(ECCV 2022) 논문 자체가 아직 위키에 없음 — 여러 논문이 baseline으로 직접 확장하는 핵심 선행 연구라 우선순위가 매우 높음. CFINet 논문도 마찬가지.
- DETR은 들어왔지만 YOLO, Faster R-CNN, RT-DETR, FCOS 등 이 위키의 다른 12편이 실제로 baseline으로 삼는 원조 아키텍처 자체는 아직 하나도 없음(모두 #pending 마커나 텍스트 인용으로만 존재) — RT-DETR은 [[UAV-DETR]]에 `#pending:rt-detr`로 마킹되어 있어 우선순위가 높다.
- 원격탐사/드론뷰(FANet, RS-TOD, UAV-DETR, FFSSTD-Net)와 SAR(CDATOD-Diff), 지상 시나리오(SODA-D 등)를 모두 다루는 unc-sod 외에, 여러 센서 도메인(광학/SAR/적외선)을 직접 비교하는 논문은 없음.
- 연산 가속 계열(querydet)과 feature 강화 계열을 실제로 결합한 논문은 아직 없음 — MOC 상에서만 결합 가능성을 언급한 상태.
- DETR이 스스로 인정한 "소형 객체 성능 열세"를 이 위키의 feature 강화 기법(주파수 도메인, reconstruction 등)으로 보완하려는 시도가 DETR 계열 자체에는 아직 없음 — UAV-DETR이 유사한 방향이지만 RT-DETR 기반이라 원조 DETR과는 한 단계 떨어져 있다.

# 관련 MOC
- [[000-Home]]
- [[Instance_Segmentation_Moc]] — [[Self_Reconstruction_Difference_Map]] 개념을 공유하는 인접 분야. [[Reconstruction_Error_Guided_Instance_Segmentation]]이 sr-tod의 원리를 detection에서 segmentation으로, 가시광에서 적외선 도메인으로 확장했다.
