---
title: "Precision and speed: LSOD-YOLO for lightweight small object detection (원문 요약)"
tags: [paper-source]
created: 2026-08-04
---

[[LSOD-YOLO]]의 원문 요약. 분석·평가는 담지 않는다 — 순수 번역/발췌.

# Abstract
소형 객체 검출은 컴퓨터 비전에서 매우 중요하지만, 기존 방법들은 저해상도, 밀집 분포, 복잡한 배경 등의 문제로 인해 미검출(missed detection)과 오검출(false detection)에 취약한 경우가 많다. 본 연구는 이러한 문제를 해결하기 위해 특별히 설계된, YOLOv8 기반의 경량 소형 객체 검출 알고리즘 LSOD-YOLO를 제안한다. 제안하는 lightweight cross-layer output reconstruction(LCOR) 모듈은 cross-layer connection을 통해 얕은 계층과 깊은 계층의 feature 정보 통합을 강화함으로써 소형 객체 검출을 향상시킨다. 또한, large separable convolution과 attention mechanism을 결합한 spatial pyramid pooling layer(SPPFL)가 multi-scale feature를 효과적으로 통합한다.

정규화 기반 attention mechanism에 기반한 C2f-N 모듈은 복잡한 배경에서의 feature 추출을 한층 더 개선하며, 경량 upsampler인 Dysample은 최소한의 연산 비용으로 풍부한 이미지 디테일을 보존한다. VisDrone2019 데이터셋에서의 실험 결과, LSOD-YOLO는 YOLOv8s 대비 정확도(Precision) 3.2%, 재현율(Recall) 1.9%, mAP0.5 2.5%의 향상을 달성하는 동시에 파라미터 수와 모델 크기를 각각 65.5%, 66.2% 줄였다. TinyPerson, LEVIR-Ship, UAVDT 데이터셋에서의 추가 평가는 이 알고리즘의 우수한 일반화 능력과 성능을 확인시켜준다.

# Introduction

#### 응용 분야 소개
인공지능 기술의 발전과 함께 target detection 연구가 활발히 이루어지고 있으며, 소형 객체 검출은 농업 해충 모니터링, 지능형 교통 시스템, 보안 감시, 드론 기반 재난 구조, 원격 감지 위성 영상, 의료 영상 조기 병변 식별, 산업 검사 등 다양한 응용 분야를 갖는다.

#### 문제의 어려움과 접근 흐름
소형 객체 검출은 제한된 픽셀 표현, 밀집 분포, 복잡한 배경 간섭으로 정확한 feature 추출이 어렵고 실시간 처리 요구까지 겹쳐 도전적이다. 전통적 수작업 feature 기반 방법은 이런 문제에 취약한 반면, 딥러닝 기반 방법은 자동 feature 추출로 성능을 크게 향상시켰다.

#### 기존 연구 한계 지적 — 정확도-속도 트레이드오프
2단계 알고리즘(R-CNN 계열)은 정확하지만 느리고, 1단계 알고리즘(YOLO 계열 등)은 빠르지만 정확도가 낮을 수 있다. YOLO 계열에서 TPH-YOLOv5, FE-YOLOv5, YOLO-S, FFNB, STC-YOLO, SO-YOLOv5 등 다양한 개선이 시도되었으나, 정확도를 높인 방법은 파라미터가 늘고 경량화한 방법은 정확도가 떨어지는 트레이드오프가 여전히 남아있다.

#### 제안 방법 개요
이 문제를 해결하기 위해 본 논문은 정확도와 속도의 균형을 맞춘 YOLOv8 기반 경량 소형 객체 검출 알고리즘 LSOD-YOLO를 제안하며, 주요 기여로 LCOR·SPPFL·C2f-N 모듈과 Dysample 업샘플러 네 가지를 제시한다.

# Main Contribution
1. Lightweight cross-layer output reconstruction 모듈(LCOR)을 도입, 불필요한 네트워크 계층 제거로 모델 복잡도를 줄이는 동시에 cross-layer feature fusion으로 저해상도 조건의 소형 객체 검출 성능을 향상시켰다.
2. 새로운 spatial pyramid pooling layer(SPPFL)를 구성, large separable convolution과 attention mechanism으로 multi-scale feature를 통합해 소형 객체 민감도와 검출 정확도를 높였다.
3. NAM(Normalized Attention Module) 기반 residual feature learning 구조인 C2f-N을 설계, 복잡한 배경에서 핵심 feature 추출과 모델 강건성을 향상시켰다.
4. 경량 upsampler Dysample을 도입, nearest-neighbor interpolation을 대체해 upsampling 정확도와 디테일 보존을 강화하고 feature distortion을 줄였다.

# Conclusion
본 논문은 기존 YOLOv8을 개선한 경량 소형 객체 검출 알고리즘 LSOD-YOLO를 제시한다. 이 알고리즘은 소형 객체 검출 모델에 내재된 연산 복잡도, 실시간 처리, 검출 정확도라는 과제를 구체적으로 겨냥한다. Lightweight cross-layer output reconstruction 모듈(LCOR), 개선된 spatial pyramid pooling layer(SPPFL), Normalization-based Attention Module(NAM)에 기반한 C2f-N 모듈, 그리고 경량 upsampler인 Dysample을 결합하여 모델의 연산 부담을 효과적으로 줄이고, 처리 속도를 높이며, 검출 정확도를 크게 향상시켰다.

(1) LCOR 모듈: feature fusion 과정을 최적화하고 불필요한 feature map 레벨을 제거함으로써 모델의 파라미터 수를 크게 줄이는 동시에, cross-layer connection을 추가하여 얕은 정보와 깊은 정보의 통합을 강화함으로써 소형 객체에 대한 검출 능력을 향상시킨다.

(2) SPPFL 모듈: large separable kernel attention을 spatial pyramid pooling fusion과 결합하여 구성되며, multi-level feature 간의 상호작용을 강화하고 전역(global) 정보 포착을 최적화한다.

(3) C2f-N 모듈: batch normalization에 기반한 attention mechanism을 C2f 모듈에 도입함으로써, 복잡한 환경에서 소형 객체를 포착하는 모델의 능력이 크게 향상되며, Grad-CAM 시각화 결과는 C2f-N 모듈이 중요하지 않은 feature를 효과적으로 억제하고 핵심 feature를 강조할 수 있음을 보여준다.

(4) Dysample 업샘플러: 샘플링 포인트 위치를 동적으로 조정함으로써 더 많은 이미지 디테일을 보존하고, 소형 객체에 대한 pixel distortion과 정보 손실을 줄이며, upsampling 과정에서의 feature 인식 정확도를 향상시킨다.

실험 평가에서 LSOD-YOLO는 VisDrone2019 데이터셋에서 mAP0.5 37.0%를 달성했으며, 이는 baseline 모델 대비 2.5%의 향상이다. 다른 주류 모델들을 상회하는 성능을 보였고, 매우 복잡한 환경과 multi-scale 타겟 시나리오에서도 우수한 성능을 보였다. 추가 테스트는 다양한 데이터셋에서 이 모델의 우수한 일반화 능력과 뛰어난 검출 결과를 입증한다.

향후 연구는 저조도(poor lighting) 조건 및 불규칙한 타겟 분포 시나리오에서의 모델 검출 성능 향상에 초점을 맞출 계획이다. 이를 위해 극단적인 조명 변화에 대응하기 위한 고급 이미지 향상(image enhancement) 기법과 적응적 학습 전략을 탐구하고, 불규칙한 타겟 분포에 대응하기 위한 동적으로 조정되는 anchor box 전략을 개발할 계획이다. 또한 실시간 응용에서의 적용성을 높이기 위해, 특히 연산 자원이 제한된 모바일 기기에서의 알고리즘 성능을 최적화하고자 model pruning과 quantization 기법을 추가로 탐구할 계획이다.
