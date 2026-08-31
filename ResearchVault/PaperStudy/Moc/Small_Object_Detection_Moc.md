---
title: "Small/Tiny Object Detection MOC"
tags: [moc]
task: small-object-detection
created: 2026-08-04
updated: 2026-08-24
---

# 이 분야가 다루는 핵심 질문
- 작은/타이니 객체는 픽셀 수가 적어 feature가 약하고 downsampling 과정에서 정보가 쉽게 사라진다 — 이 약한 feature를 어떻게 보강할 것인가?
- 기존 detector(anchor, label assignment, loss)는 일반 크기 객체를 기준으로 설계되어 있다 — 작은 객체에 맞게 이 파이프라인을 어떻게 다시 조정할 것인가?
- 성능 개선은 대부분 연산량 증가를 동반한다 — 실제 배포(엣지/UAV) 관점에서 정확도와 경량화를 동시에 만족할 수 있는가?

# 지금까지 다룬 흐름
지금까지 읽은 26편 중 17편은 "기존 detector(YOLO, Faster R-CNN, RT-DETR, FCOS, DetectoRS, DINO)에 plug-in 모듈을 추가"하는 방식, 6편은 Deformable DETR을 baseline으로 삼아 object query를 동적으로 조정하는 "dynamic query DETR" 계열, 나머지 3편([[DETR]], [[Deformable_Convolutional_Networks]], [[Deformable-DETR]])은 이후 흐름이 기반하는 순수 foundational 논문이다. 개입 지점에 따라 아래 계열로 나뉜다. 자세한 축별 비교는 [[Small_Object_Detection_Approaches]] 참고.

**Foundational 계열** — 이후 흐름 전체의 출발점이 된 원조 논문.
- [[DETR]] — anchor·NMS 없이 객체 탐지를 직접적인 집합 예측으로 재정의한 최초의 end-to-end transformer 탐지기. [[UAV-DETR]]이 기반하는 RT-DETR의 계보상 원조에 해당하며, "소형 객체 성능이 약하다"는 한계를 스스로 처음 명시한 논문이기도 하다 — 이 위키의 feature 강화 계열 다수가 정확히 이 한계를 다른 아키텍처(YOLO, R-CNN)에서 다루고 있다는 점에서, DETR 계열에도 동일 문제의식이 적용될 여지가 있는지가 흥미로운 교차점.
- [[Deformable_Convolutional_Networks]] — DETR보다 3년 앞서(2017) CNN의 "고정된 샘플링 위치"라는 또 다른 수작업 가정을 없앤 원조 논문. Convolution/RoI pooling의 grid 위치에 입력 조건부 학습 offset을 더해 수용영역을 객체 크기·형태에 맞게 동적으로 변형시킨다([[Deformable_Sampling_Offset]]).
- [[Deformable-DETR]] — DETR의 극도로 느린 수렴(500 epoch)과 소형 객체 성능 열세를, deformable convolution의 offset 아이디어를 attention의 sampling location에 이식한 deformable attention module로 해결. 각 query가 reference point 주변 소수 sampling point만 보게 해 FPN 없이도 멀티스케일 feature를 직접 통합하고, 10배 적은 epoch로 DETR을 능가. DETR 계열과 CNN 계열([[Deformable_Convolutional_Networks]])을 잇는 다리이자, 뒤이어 다룬 DQ-DETR·Density-Aware DETR·IG-DETR·PaQ-DETR·DQA-DETR·DQP-DETR 6편의 공통 baseline.

**Feature 강화 계열** — 약한 feature 자체를 보강. 신호를 얻는 소스가 서로 다르다.
- [[SR-TOD]] — self-reconstruction의 difference map을 신호로 사용. 생성 모델 없이 reconstruction의 부산물만 활용하는 가장 가벼운 방식.
- [[Feature_Info_Driven_Gaussian]] — 정보이론 관점의 information map + 위치 기반 Gaussian Mixture map을 동시에 사용. sr-tod와 달리 "어디를 강조할지"를 명시적으로 모델링.
- [[FANet]], [[UAV-DETR]] — 둘 다 주파수(DFT/FFT) 도메인 정보를 쓴다는 공통점이 있다. 원격탐사(FANet)와 드론뷰(UAV-DETR) 각각의 항공 이미지 특성(반복적 텍스처)과 관련 있을 가능성.
- [[RS-TOD]] — 공간 attention(RSAM) + 전용 고해상도 헤드 추가. FANet/SR-TOD처럼 기존 헤드를 보강하는 대신 헤드 자체를 새로 만든다는 점에서 다른 접근.
- [[Detection_Oriented_Rectification]] — 다른 feature 강화 논문들과 달리 "복원 목표"가 pixel fidelity가 아니라 탐지 지향적(task-oriented)이라는 점이 핵심 차별점. 열화 패턴을 명시적으로 모델링하는 첫 시도.
- [[FFSSTDNet]] — sr-tod와 유사하게 학습 시에만 존재하는 auxiliary reconstruction branch(FSR)를 쓰지만, 재구성 오차 자체를 attention prior로 쓰지 않고 backbone feature 품질을 간접적으로 끌어올리는 정규화 역할만 한다는 점에서 구별됨. Full-scene 위성 이미지 특유의 RONI(배경) 연산 비용 문제를 CFD 모듈로 별도 해결.
- [[ORFENet]] — FFSSTDNet과 마찬가지로 학습 시에만 존재하고 추론 시 완전히 제거되는 auxiliary reconstruction branch(ORB)를 쓰지만, 재구성 target이 원본 이미지가 아니라 GT 박스 기반 이진 foreground/background 마스크라는 점에서 SR-TOD의 difference map보다 훨씬 단순한 self-supervision. 여기에 fine-grained/close-range/distant-context 세 receptive field를 동적 가중합하는 MRFAFEM을 더해, "정보 손실 억제"와 "다중 receptive field 활용"을 한 프레임워크에서 함께 다룬 첫 사례.
- [[FFCA-YOLO]] — reconstruction이나 정보이론이 아니라 순수 attention/융합 설계로 feature를 강화하는 계열. 지역 문맥 확장(FEM), 채널별 학습 가중 다중 스케일 융합(FFM), GCNet/SCP 계보의 전역 문맥 attention(SCAM) 세 모듈을 동시에 배치한 첫 사례 — 다른 논문들이 대체로 한두 지점만 건드리는 것과 달리 세 지점(지역/스케일간/전역)을 한 프레임워크에서 함께 다룬다.
- [[BAFNet]] — 전경 attention과 그 여집합인 배경 attention을 동시에 생성하는 이중 스트림(DSAM)으로 배경 억제를 명시적으로 모델링한 첫 사례. 여기에 Laplacian pyramid 기반 경계 GT로 supervision하는 Boundary-Aware Branch를 더해, [[ORFENet]]의 ORB·[[FFSSTDNet]]의 FSR과 같은 "학습시에만 관여하는 auxiliary branch" 계보의 세 번째 변형(대상이 영역/이미지가 아니라 경계)을 제시.
- [[RTP-Net]] — "수용영역 확장과 texture 보존이 근본적으로 상충한다"는 문제의식을 backbone 소스 단계부터 대·소 커널 병렬 브랜치(GLEM)로 정면 돌파. FFT 없이 spatial domain의 down/up-sampling residual만으로 고/저주파를 근사 분리(CRM)한다는 점이 [[FANet]]·[[UAV-DETR]]의 FFT 기반 접근과 대비된다. 유일하게 정확도 개선과 GFLOPs·파라미터·FPS 동시 개선을 모두 달성.
- [[CoLR-Det]] — SR을 명시적 이미지 복원이 아니라 학습 전용 latent 정규화로 재정의(추론 시 SR 브랜치 완전 제거). [[ORFENet]]의 ORB·[[FFSSTDNet]]의 FSR·[[BAFNet]]의 Boundary-Aware Branch와 같은 "학습시에만 존재하는 auxiliary branch" 계보의 네 번째 변형이지만, 이 원리를 "복원이 탐지를 지배해서는 안 된다"는 명제로 가장 명시적으로 정식화하고 saliency 기반 non-destructive token routing으로 배경 텍스처 오염을 능동적으로 차단한다는 점에서 한 단계 더 나아간다. 저해상도(2× downsampling) 원격탐사 특화.

**연산 가속(sparse computation) / 서브영역 국소화 계열** — "어디를 계산할지/어디를 양성 샘플로 볼지/어디를 잘라낼지"를 좁혀 연산을 줄이거나 배경 간섭을 억제한다는 공통점이 있다.
- [[QueryDet]] — 저해상도 feature 예측으로 고해상도 sparse convolution 위치를 좁히는 Cascade Sparse Query. 정확도 손실 없이 고해상도 feature 연산 비용을 74%→1%로 절감. FPN 레벨 간 coarse-to-fine의 원조 격.
- [[FFSSTDNet]]의 CFD 모듈도 이 갈래와 문제의식이 겹친다 — patch 단위로 배경을 걸러낸다는 점에서 QueryDet과 구현 층위만 다를 뿐 동일한 coarse-to-fine 사상을 공유.
- [[CDATOD-Diff]] — "어디를 계산할지"가 아니라 "어디를 양성 샘플로 볼지"를 diffusion으로 정제한다는 점에서 연산 가속 계열과는 다른 축이지만, QueryDet과 결합 가능성이 있는 직교적 설계로 함께 묶어 둠.
- [[YOFOR]] — Coarse detection 결과를 클러스터링해 이미지/픽셀 공간에서 직접 객체 밀집 서브영역을 크롭(ALSM)하는, 세 논문과 층위는 다르지만(feature/patch가 아닌 이미지 레벨) 동일한 coarse-to-fine 사상. 여기에 recursive Gaussian filter로 배경을 흐리게 하는 FEM, tail class를 공간 semantic 보존하며 복제하는 CBM(이 위키에서 long-tailed 문제를 다룬 유일한 사례)을 결합.

**Label assignment/sampling 계열**
- [[Unc-SOD]] — 유일하게 feature가 아니라 "어떤 prior를 positive로 볼지" 자체를 동적으로 바꾼다. Feature 강화 계열과 직교적이라 결합 가능성이 있음(논문 자체도 Cascade R-CNN 등에 이식해 효과 유지 확인).
- [[CDATOD-Diff]] — RFLA의 Gaussian receptive field 매칭을 계층적으로 확장하고, 여기에 CLIP의 크로스모달 의미 정보를 diffusion denoising 조건으로 결합해 "의미적으로 타당한" 양성 샘플을 생성. 이 위키에서 VLM(CLIP)을 label assignment에 결합한 첫 사례.

**아키텍처 경량화 계열**
- [[LSOD-YOLO]] — "성능 유지하며 경량화"가 목표. 저기여도 검출 헤드(P5) 제거 + cross-layer connection으로 구조 자체를 재배치, 파라미터 65.5% 감소.
- [[FFCA-YOLO]] — L-FFCA-YOLO(경량판)는 PConv로 backbone convolution 연산을 재구성해 정확도 손실 거의 없이 파라미터 30% 감소. LSOD-YOLO와 목표는 같지만 구조 재배치가 아니라 연산 층위의 경량화라는 점에서 접근이 다르다.
- [[RTP-Net]] — 별도 경량판을 만들지 않고 기본 설계(depthwise separable 병렬 커널) 자체로 정확도 개선과 GFLOPs 27.2% 감소를 동시에 달성 — LSOD-YOLO·FFCA-YOLO가 "경량화 vs 정확도"의 균형을 관리하는 데 그친 것과 달리 둘 다 개선한 예외적 사례.

**End-to-end 구조 계열**
- [[UAV-DETR]] — 유일한 DETR(anchor-free, NMS-free) 계열. 위 주파수 도메인 활용과 별개로, 구조 자체가 다른 논문들(대부분 YOLO/R-CNN/FCOS 기반)과 궤를 달리한다.

**Dynamic Query DETR 계열** — Deformable DETR을 공통 baseline으로 삼아, object query의 개수·구성을 이미지 내용에 따라 동적으로 조정하는 6편. 두 하위 갈래로 나뉜다(자세한 비교는 [[Small_Object_Detection_Approaches]]의 전용 절 참고).
- *하위 갈래 A(전역 밀도 기반, [[Density_Guided_Dynamic_Query]])*: [[DQ-DETR]](density map 4단계 분류로 query 수 결정, 원조) → [[Density-Aware-DETR]](분류를 연속 회귀로 대체, DQ-DETR과 직접 대조 실험) → [[IG-DETR]](6단계 세분화 + 덧셈 residual feature 강화) → [[DQP-DETR]](density를 encoder memory 주입·토큰 순위 학습까지 확장, 가장 포괄적).
- *하위 갈래 B(인스턴스 레벨 패턴/품질 기반, [[Pattern_Quality_Aware_Query_Refinement]])*: [[PaQ-DETR]](클러스터링 병합+품질 pruning, 유일하게 COCO 일반 탐지 검증) → [[DQA-DETR]](유일한 oriented detection, one-to-one matching의 gradient 왜곡을 이론적으로 증명하고 병합으로만 해결).
- 6편 모두에서 반복되는 패턴: "다중 신호를 쓴다는 것 자체"의 기여가 "그 신호를 얼마나 정교화하는가"보다 일관되게 크다(DQP-DETR의 RCS, Density-Aware DETR의 IDE&QA 등). DQA-DETR의 query 900→2400 확장 실험(baseline mAP 붕괴)은 "query를 늘리는 것 자체는 위험하다"는 이 계열 전체의 문제의식을 가장 극적으로 보여준다.

# 이 분야를 관통하는 개념
- [[Self_Reconstruction_Difference_Map]] — sr-tod의 핵심 기여. reconstruction 기반 접근의 시작점.
- [[Frequency_Domain_Feature_Enhancement]] / [[Frequency_Domain_Feature_Attention]] — 주파수 도메인 신호를 쓰는 두 논문(uav-detr, fanet)에서 각각 등장. 아직 하나로 통합할지는 미결정 — 두 논문의 메커니즘이 미묘하게 다름(전자는 FFT/IFFT 기반 필터링, 후자는 DFT/DCT 기반 attention).
- [[Gaussian_Box_Uncertainty_Modeling]] / [[Position_Gaussian_Saliency_Map]] — 둘 다 Gaussian 분포로 뭔가를 모델링하지만 대상이 다르다(박스 좌표 불확실성 vs 위치 saliency). 혼동하지 않도록 주의.
- [[Perception_And_Interaction]] — unc-sod의 핵심 기여, 두 pyramid level 간 feature 융합.
- [[Degradation_Aware_Rectification]] — detection-oriented-rectification의 핵심 기여.
- [[Remote_Sensing_Attention_Module]] — rs-tod의 핵심 기여.
- [[Lightweight_Cross_Layer_Output_Reconstruction]] — lsod-yolo의 핵심 기여.
- [[Cascade_Sparse_Query]] — querydet의 핵심 기여. 연산 가속 계열의 시작점.
- [[Deformable_Sampling_Offset]] — Deformable Convolutional Networks의 핵심 기여. 이후 DETR 계열에서 attention sampling location을 변형하는 데 재사용되는 원조 메커니즘.
- [[Density_Guided_Dynamic_Query]] — DQ-DETR의 핵심 기여. Dynamic query DETR 계열 하위 갈래 A(전역 밀도 기반)의 시작점. DQ-DETR·Density-Aware DETR·IG-DETR·DQP-DETR 4편이 공유.
- [[Pattern_Quality_Aware_Query_Refinement]] — PaQ-DETR의 핵심 기여. Dynamic query DETR 계열 하위 갈래 B(인스턴스 레벨 패턴/품질 기반)의 시작점. PaQ-DETR·DQA-DETR 2편이 공유.
- [[Class_Balanced_Spatial_Copy_Paste]] — YOFOR의 CBM 핵심 기여. 이 위키에서 long-tailed detection을 다룬 유일한 개념.
- [[Dual_Stream_Foreground_Background_Attention]] — BAFNet의 DSAM 핵심 기여. 전경 attention의 여집합으로 배경 attention을 별도 파라미터 없이 유도하는 경량 설계.
- [[Collaborative_Receptive_Field_Texture_Optimization]] — RTP-Net의 핵심 기여. 수용영역 확장과 texture 보존의 trade-off를 backbone 소스 단계 병렬 브랜치로 해소.
- [[Latent_Restoration_Regularization]] — CoLR-Det의 핵심 기여. SR을 명시적 복원이 아니라 학습 전용 latent 정규화로 재정의, "학습시에만 존재하는 auxiliary branch" 계보의 네 번째 변형.

# 비교 문서
- [[Small_Object_Detection_Approaches]]

# 아직 못 채운 빈틈
- unc-sod, sr-tod, cdatod-diff, feature-info-driven-gaussian, orfenet 등 다수 논문에서 반복적으로 비교 대상·관련 연구로 언급된 RFLA(ECCV 2022) 논문 자체가 아직 위키에 없음 — 여러 논문이 baseline으로 직접 확장하는 핵심 선행 연구라 우선순위가 매우 높음. CFINet 논문도 마찬가지.
- DETR·Deformable Convolutional Networks·Deformable DETR은 들어왔지만 YOLO, Faster R-CNN, RT-DETR, FCOS 등 이 위키의 다른 논문들이 실제로 baseline으로 삼는 원조 아키텍처 자체는 아직 하나도 없음(모두 #pending 마커나 텍스트 인용으로만 존재) — RT-DETR은 [[UAV-DETR]]에 `#pending:rt-detr`로 마킹되어 있어 우선순위가 높다.
- 원격탐사/드론뷰(FANet, RS-TOD, UAV-DETR, FFSSTD-Net)와 SAR(CDATOD-Diff), 지상 시나리오(SODA-D 등)를 모두 다루는 unc-sod 외에, 여러 센서 도메인(광학/SAR/적외선)을 직접 비교하는 논문은 없음.
- 연산 가속 계열(querydet)과 feature 강화 계열을 실제로 결합한 논문은 아직 없음 — MOC 상에서만 결합 가능성을 언급한 상태.
- DETR이 스스로 인정한 "소형 객체 성능 열세"를 이 위키의 feature 강화 기법(주파수 도메인, reconstruction 등)으로 보완하려는 시도가 DETR 계열 자체에는 아직 없음 — UAV-DETR이 유사한 방향이지만 RT-DETR 기반이라 원조 DETR과는 한 단계 떨어져 있다.

# 관련 MOC
- [[000-Home]]
- [[Instance_Segmentation_Moc]] — [[Self_Reconstruction_Difference_Map]] 개념을 공유하는 인접 분야. [[Reconstruction_Error_Guided_Instance_Segmentation]]이 sr-tod의 원리를 detection에서 segmentation으로, 가시광에서 적외선 도메인으로 확장했다.
