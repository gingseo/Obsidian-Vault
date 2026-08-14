---
title: "UAV-DETR: Efficient End-to-End Object Detection for Unmanned Aerial Vehicle Imagery (원문 요약)"
tags: [paper-source]
created: 2026-08-04
---

[[UAV-DETR]]의 원문 요약. 분석·평가는 담지 않는다 — 순수 번역/발췌.

# Abstract
무인항공기(UAV) 객체 탐지(UAV-OD)는 다양한 시나리오에서 널리 활용되어 왔다. 그러나 기존 UAV-OD 알고리즘 대부분은 수작업으로 설계된 구성 요소에 의존하며, 이는 광범위한 튜닝을 필요로 한다. 이러한 수작업 설계 구성 요소에 의존하지 않는 end-to-end 모델들은 주로 자연 이미지를 대상으로 설계되어 있어 UAV 영상에는 효과가 떨어진다.

이러한 문제를 해결하기 위해 본 논문은 UAV 영상에 맞춘 효율적인 detection transformer(DETR) 프레임워크, 즉 UAV-DETR을 제안한다. 이 프레임워크는 서로 다른 스케일에서 공간 정보와 주파수 정보를 함께 포착하는 multi-scale feature fusion with frequency enhancement 모듈을 포함한다. 또한 다운샘플링 과정에서 중요한 공간적 디테일을 보존하는 frequency-focused downsampling 모듈을 제시한다. 서로 다른 fusion 경로에서 온 feature를 정렬하고 융합하기 위한 semantic alignment and calibration 모듈도 개발되었다.

실험 결과는 다양한 UAV 영상 데이터셋에 걸쳐 본 접근법의 효과성과 일반화 능력을 입증한다. VisDrone 데이터셋에서 본 방법은 baseline 대비 AP를 3.1%, AP50을 4.2% 개선했다. UAVVaste 데이터셋에서도 유사한 개선이 관찰되었다. 프로젝트 페이지: https://github.com/ValiantDiligent/UAV-DETR

# Introduction

#### UAV-OD의 부상과 end-to-end 필요성
카메라 탑재 UAV는 다양한 분야에 적용되어 왔고, 핵심 기술인 UAV-OD가 주목받아왔다. 그러나 인기 있는 알고리즘들이 NMS·anchor box 등 수작업 튜닝에 의존해 실제 적용이 복잡한 반면, end-to-end 모델은 이 문제에서 자유롭다.

#### 기존 연구 한계 지적 — 실시간성과 도메인 불일치
DETR 계열은 소형 객체 탐지 능력을 개선해왔지만 높은 연산 비용·낮은 실시간성이 문제였고, 이를 RT-DETR[6]이 YOLO를 능가하며 해결했으나 자연 이미지 기준 설계라 UAV 영상엔 그대로 적용하기 어렵다.

#### 개선 방향 제시
UAV 영상은 소형 객체·가림 등 어려움이 크므로(Fig. 1), 세밀한 feature 추출과 객체-주변 관계 활용이 탐지 정확도를 높이는 유효한 방법이다.

#### 제안 방법 개요
본 논문은 UAV 영상을 위한 효율적인 detection transformer, UAV-DETR을 제안한다. 공간·주파수 도메인 정보를 함께 활용해 고주파 성분을 보존하고, 다운샘플링 중 핵심 디테일을 유지하며, 서로 다른 fusion 경로의 feature를 정렬해 semantic 표현력을 강화한다.

#### 기여 요약
저자들은 논문의 주요 기여를 4가지로 요약해 제시한다 (아래 "Main Contribution" 참고).

# Main Contribution
1. UAV 영상을 위한 효율적인 end-to-end detector transformer UAV-DETR을 제안. 우수한 정확도·실시간 성능을 달성하며, 다양한 요구사항에 맞춘 세 가지 크기의 모델을 제시.
2. 소형·가려진 객체 탐지를 향상시키는 multi-scale feature fusion with frequency enhancement 모듈 제시.
3. 이중 도메인(dual-domain) 정보를 보존하는 frequency-focused downsampling 모듈 개발.
4. 서로 다른 fusion 경로의 feature를 정렬해 탐지 성능을 높이는 semantic alignment and calibration 모듈 제안.

# Conclusion
본 논문은 UAV 영상을 위해 특별히 설계된 실시간 end-to-end 객체 탐지기인 UAV-DETR을 제안한다. MSFF-FE 모듈, FD 모듈, SAC 모듈을 개발함으로써 UAV-DETR은 항공 영상에서 소형·가려진 객체를 탐지하는 어려움을 완화하는 데 도움을 준다. 각 모듈은 저마다 중요한 역할을 한다 — MSFF-FE 모듈은 고주파 성분을 보존하면서 멀티스케일 feature fusion을 강화하고, FD 모듈은 다운샘플링 중 공간적 디테일 유지에 집중하며, SAC 모듈은 서로 다른 feature 경로 간의 semantic alignment를 보장한다. VisDrone과 UAVVaste 데이터셋에서의 실험 결과는 본 방법이 유사한 연산 비용을 가진 기존 접근법보다 높은 정확도를 달성하면서도 실시간 추론 속도를 유지함을 보여준다. 향후 연구는 노이즈에 대한 강건성 개선에 초점을 맞출 것이다.
