---
title: "Remote Sensing Attention Module (RSAM)"
tags: [concept, object-detection, attention-mechanism, remote-sensing, channel-attention, spatial-attention]
created: 2026-08-04
updated: 2026-08-04
---

# 정의
RSAM은 channel attention과 spatial attention을 함께 사용하는 경량 attention 모듈로, 원격탐사 영상의 tiny object detection에 맞게 설계되었다. 입력 feature map(h×w×c)에 대해 세 가지 레벨(채널 크기 1, c/4, c/2)의 AvgPool을 병렬로 적용해 pooled feature map들을 얻고, 이를 concat하여 피라미드 형태로 만든 뒤 1×1 convolution + BatchNorm + Sigmoid를 거쳐 attention map(reweight)을 생성한다. 이 attention map을 원본 feature map에 element-wise 곱하여 최종 출력을 얻는다. SE Block(채널 attention만 사용)과 CBAM(채널+공간 attention을 쓰지만 2-level pooling에 그치고 skip connection이 없음)의 한계를 보완하기 위해, RSAM은 3-level channel pooling과 skip connection을 도입해 vanishing gradient 문제를 완화하고 채널 간 상호작용 정보를 더 잘 보존한다.

# 등장 논문
- [[RS-TOD]] — YOLOv8n 기반 RS-TOD 모델의 4개 detection head(20×20, 40×40, 80×80, 160×160) 앞에 각각 RSAM을 배치하여 feature representation을 강화하는 핵심 제안 모듈로 사용. SODA-A/AI-TOD/TinyPerson 세 데이터셋에서 RSAM 적용이 baseline YOLOv8 대비 mAP50 7~11%p 개선에 기여.

# 변형/발전
- 현재는 RS-TOD(2025) 한 논문에서만 제안·사용된 모듈. SE Block, CBAM 등 기존 channel/spatial attention 계열의 설계를 절충·확장한 형태로 소개되었다.
- 저자들이 논문 내에서 스스로 제시한 향후 발전 방향: 3-level보다 더 세분화된 channel pooling, 또는 다른 attention 메커니즘과의 결합을 통한 일반화 성능 개선.

# 관련 개념
- SCAM([[FFCA-YOLO]]) — 원격탐사 detection head 주변에 채널+공간 attention을 배치한다는 목적은 같지만, GCNet/SCP 계보의 전역 QK 기반 문맥 모델링이라 RSAM(다단계 pooling 기반)과 구현 메커니즘이 다르다. 별도 concept으로 유지.
