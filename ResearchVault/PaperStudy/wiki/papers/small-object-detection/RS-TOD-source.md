---
title: "RS-TOD: Tiny object detection model in Remote Sensing Imagery (원문 요약)"
tags: [paper-source]
created: 2026-08-04
---

[[RS-TOD]]의 원문 요약. 분석·평가는 담지 않는다 — 순수 번역/발췌.

# Abstract
원격탐사 영상(RSI)은 촘촘하게 밀집된 tiny 표적 객체와 복잡한 배경이 존재한다는 점에서 자연 영상과 다르다. 이를 위한 수많은 모델이 제안되었지만, 환경 조건, 영상 내 객체의 중첩·혼동, 영상 선명도, tiny·multi-class 객체를 탐지·인식하는 데 따르는 어려움 등 다양한 문제로 인해 원격탐사 영상에서 tiny object를 탐지하는 성능은 여전히 부족하다.

본 논문에서는 YOLOv8을 개선한 버전인 Remote Sensing Tiny Object Detector(RS-TOD) 모델을 제안하며, 이는 RSI에서 직면하는 문제들을 극복하도록 설계되었다. 제안하는 RS-TOD는 feature extraction을 개선하기 위한 Remote Sensing Attention Module(RSAM)을 포함하고, tiny object 탐지에 특화된 detection head를 추가로 도입한다.

SODA-A, AI-TOD, TinyPerson 데이터셋에 대한 실험 결과는 제안된 RS-TOD의 효과를 입증한다. 이 모델은 SODA-A 데이터셋에서 mAP50 60.10, mAP50-95 20.30을 달성했으며, 이는 각각 7.29%, 6%의 개선을 반영한다. AI-TOD 데이터셋에서는 mAP50 59.84, mAP50-95 28.40을 달성해 11.34%, 6.03%의 증가를 나타냈다. TinyPerson 데이터셋에서는 mAP50 47.60, mAP50-95 16.80을 달성해 YOLOv8 모델 대비 각각 7.08%, 3.40%의 개선을 보였다. 이러한 결과는 원격탐사 영상에서 tiny object detection을 향상시키는 데 있어 제안된 RS-TOD 모델의 효과를 검증한다.

# Introduction

#### 배경 — RSI tiny object detection의 중요성과 어려움
고고도 촬영이 제공하는 넓은 범위와 독특한 시점 때문에 RSI에서의 tiny object detection이 중요한 관심 분야로 떠올랐으며, 밀집·불규칙 분포한 tiny object는 CNN 기반 feature extraction에서 간과되기 쉽다.

#### 문제의 구체화 — tiny object가 어려운 이유
32×32 픽셀보다 작은 tiny object는 배경 잡음, 큰 스케일 편차, 연속적 다운샘플링에 따른 1/16 크기 축소, 고정 receptive field와의 불일치 등으로 탐지가 특히 어렵다(Fig. 1).

#### 관련 접근 개관 — 탐지 패러다임 소개
Two-stage(R-CNN 계열)는 정밀하지만 느리고, one-stage(YOLO·SSD)는 빠르지만 정확도가 낮은 경향이 있으며, CenterNet 같은 keypoint 기반 방식은 anchor-free로 과정을 단순화한다.

#### 기존 연구 한계 지적 — YOLO 계열의 tiny object 취약성
최근 연구 다수가 평균 크기 객체 탐지에 집중해 RSI tiny object detection은 여전히 큰 도전 과제로 남아 있고, YOLOv8도 개선되고 있으나 tiny object에는 여전히 취약하며, attention mechanism 통합이 개선 방향으로 제시된다.

#### 제안 방법 개요 및 논문 구성
이러한 한계를 해결하기 위해 RSAM을 갖춘 Remote Sensing Tiny Object Detector(RS-TOD)를 제안하며, 이어서 3가지 핵심 기여와 논문 구성(관련 연구, 제안 모델, 실험, 결론)을 소개한다.

# Main Contribution
1. tiny object 탐지 정확도 향상을 위해 Remote Sensing Attention Module(RSAM)을 제안, detection head 앞에 배치해 전반적 탐지 능력을 강화했다.
2. tiny object 전용 160×160 detection head를 추가해 핵심 feature 집중과 localization 개선으로 탐지·분류 정확도를 높였다.
3. RS-TOD는 다양한 데이터셋에서 baseline YOLOv8n을 능가해 detection head와 RSAM의 효과를 입증했으며, 복잡한 환경에서도 신뢰할 수 있는 성능을 보인다.

# Conclusion
본 논문은 YOLOv8을 개선한 RS-TOD 모델을 제안했다. 이 네트워크는 경량 detection head를 통합하여 tiny object의 탐지를 크게 향상시키고, 네트워크 파라미터를 최적화하며, 추론 속도를 높인다. 또한 RSAM 모듈을 각 detection head 앞에 통합하여 feature extraction을 강화하고 네트워크의 전반적인 정확도를 향상시켰다. 실험 결과, 제안된 RS-TOD 모델은 모든 원격탐사 데이터셋에서 baseline을 능가하는 성능을 보였다. YOLOv8과 비교했을 때, SODA-A에서 mAP50 60.10%(7.29% 개선), AI-TOD에서 59.84%(11.34% 증가), TinyPerson에서 47.60%(7.08% 개선)를 달성했다. 이 연구는 RSAM 모듈을 prediction head에 통합하는 것이 tiny object detection 성능을 향상시킨다는 점을 보여준다. 향후에는 다양한 응용 분야에 걸쳐 tiny object detection의 탐지 정확도와 일반화 성능을 더욱 향상시키기 위해 RS-TOD 모델의 기능을 확장하는 것을 목표로 한다. 이러한 응용 분야는 육상, 하늘, 해상 표면과 관련된 다양한 환경 조건을 포괄한다.
