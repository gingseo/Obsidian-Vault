---
title: "Partial Convolution (PConv)"
tags: [architecture, convolution, lightweight]
created: 2026-08-28
updated: 2026-08-28
---

# 역할
표준 convolution의 파라미터·연산량을 줄이는 경량화 기법. Depthwise convolution(DWConv, MobileNet/ShuffleNet 등이 쓰는 방식)처럼 채널을 전부 따로 처리하지 않고, **입력 채널 중 일부만 표준 convolution으로 처리하고 나머지는 그대로 통과시켜** DWConv보다 실제 속도를 더 개선한다.

# 구조

## 입력/출력
- 입력: feature map `(C, H, W)`
- 출력: `(C, H, W)` (채널 수·공간 크기 모두 유지, 일부 채널만 내용이 갱신됨)

## 내부 동작
1. 입력 채널 `C`개 중 앞쪽 `C_p`개(비율 파라미터로 결정, 예: 1/4)만 표준 convolution(3×3 등)을 적용한다.
2. 나머지 `C - C_p`개 채널은 아무 연산 없이 그대로 다음 단계로 전달(항등 연결)한다.
3. 두 그룹을 다시 합쳐 원래 채널 수 `C`로 복원한다.

> [!example]- 왜 DWConv보다 빠른가 — FLOPs와 실제 속도의 괴리
> DWConv는 각 채널을 독립적으로 처리해 이론적 연산량(FLOPs)은 표준 conv보다 훨씬 적다. 하지만 실제 하드웨어에서는 **메모리 접근(memory access) 횟수**가 속도를 좌우하는데, DWConv는 채널마다 별도로 메모리를 읽고 쓰는 연산이 빈번해 FLOPs 대비 실제 속도 이득이 기대만큼 크지 않다(FasterNet 논문의 핵심 관찰). PConv는 애초에 처리하는 채널 수 자체(`C_p`)를 줄여 메모리 접근 빈도를 낮추므로, 같은 FLOPs 절감 대비 실제 latency 개선이 더 크다.

# 왜 이렇게 되는가
- **왜 채널 일부를 아예 처리하지 않는가**: 표준 conv를 채널 비율만큼만 적용하고 나머지는 그대로 통과시키면, 파라미터·연산량이 처리 비율만큼 줄어드는 것은 물론, DWConv처럼 "채널마다 커널을 따로 두는" 구조가 아니라 "적은 채널에 표준 커널을 통째로 적용"하는 구조라 메모리 접근 패턴이 더 단순하다.
- **트레이드오프**: 처리 비율(`C_p/C`)을 너무 낮추면 정보가 갱신되지 않는 채널이 늘어 정확도가 떨어진다. FasterNet은 이를 표준 convolution 2개를 뒤이어 배치해(모든 채널에 정보가 흐르도록) 보완한다 — PConv를 곧바로 여러 층 쌓지 않고, 반드시 표준 conv와 함께 블록을 구성하는 이유.

# 등장 논문
- [[FFCA-YOLO]] — L-FFCA-YOLO(경량판)에서 backbone/neck의 CSPBlock 안 bottleneck을 PConv 기반 FasterBlock으로 교체(CSPFasterBlock). 채널 재가중 비율 M=3/4(1×1 conv 채널) 설정, PConv 뒤에 표준 conv 2개를 이어 붙여 정보 흐름을 보완. 파라미터 30% 감소, 정확도 손실은 거의 없음(mAP50 0.909→0.907).
