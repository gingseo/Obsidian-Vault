---
title: "Lightweight Cross-Layer output Reconstruction (LCOR)"
tags: [concept, object-detection, small-object-detection, model-lightweighting, yolo, feature-fusion]
created: 2026-08-04
updated: 2026-08-04
---

# 정의
YOLO 계열 검출기에서 소형 객체 검출 성능과 모델 경량화를 동시에 달성하기 위한 검출 헤드 재구성 기법. 두 가지 조작을 결합한다: (1) semantic 기여는 크지만 소형 객체 민감도가 낮은 저해상도·고레벨 검출층(YOLOv8 기준 P5)을 제거해 파라미터·연산량을 줄이고, (2) 대신 고해상도 저레벨(P2)에 검출 헤드를 추가하되, 얕은 layer(디테일 정보 풍부)와 깊은 layer(semantic 정보 풍부) 사이에 cross-layer connection(skip connection)을 두어 두 정보를 함께 활용한다. 즉 "고해상도 헤드를 단순 추가"할 때 따라오는 연산량 증가를 "저기여도 헤드 제거"로 상쇄하면서, cross-layer connection으로 정보 손실을 최소화하는 것이 핵심 트레이드오프 설계다.

# 등장 논문
- [[LSOD-YOLO]] — YOLOv8s에 처음 이 기법을 도입. P5 검출 헤드를 제거하고 P2 헤드를 추가하며 P3-P2 간 cross-layer connection을 결합. Ablation에서 LCOR 제거 시 파라미터가 3.8M→12.3M로 급증하고 mAP0.5가 37.0%→35.8%로 하락해, 경량화 효과의 대부분이 이 모듈에서 나옴을 확인. cross-layer connection만 제거한 변형(파라미터 3.7M, mAP0.5 36.5%)과 비교해 connection 자체의 기여(+0.5%p)도 별도로 검증됨.

# 변형/발전
- 이 개념이 나오게 된 배경: 소형 객체 검출을 위해 P2 레벨에 검출 헤드를 추가하는 것은 YOLOv5/v8 계열에서 흔히 쓰이는 방법(논문 내 YOLOv8s-P2 비교군)이나, 단순 추가는 검출 헤드가 3개→4개로 늘어 연산량이 크게 증가하는 부작용이 있음. LSOD-YOLO는 "헤드를 늘리는 대신 하나는 빼고 하나는 넣는" 방식으로 이 문제를 해결.
- 현재로선 [[LSOD-YOLO]] 한 편에서만 제안된 기법이며, 다른 논문에서의 재사용 사례는 아직 확인되지 않음.

# 관련 개념
-
