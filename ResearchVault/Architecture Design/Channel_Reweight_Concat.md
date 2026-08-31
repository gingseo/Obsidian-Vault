---
title: "Channel Reweight Concat (CRC)"
tags: [architecture, feature-fusion]
created: 2026-08-28
updated: 2026-08-28
---

# 역할
여러 스케일의 feature map을 합칠 때, 단순히 이어붙이거나(concat) 모든 채널에 같은 가중치를 주는 대신, **채널마다 학습된 가중치를 곱해서 합치는** 다중 스케일 feature 융합 방법. BiFPN이 "가중합(weighted sum)"을 스케일(feature map) 단위로만 학습하는 것과 달리, 이 방법은 concat한 뒤 **채널 하나하나에** 별도 가중치를 학습시킨다.

# 구조

## 입력/출력
- 입력: 서로 다른 스케일의 feature map 여러 장 `(H, W, C_1), (H, W, C_2), ..., (H, W, C_k)` (같은 공간 해상도로 이미 맞춰진 상태)
- 출력: `(H, W, N)` (`N = C_1 + C_2 + ... + C_k`, 채널별로 재가중된 뒤 결합된 feature map)

## 내부 동작
1. 입력 feature map들을 채널 축으로 그대로 concat: `(H, W, C_1)` + ... + `(H, W, C_k)` → `(H, W, N)`.
2. 학습 가능한 가중치 벡터 `[ω_1; ω_2; ...; ω_N]` (concat 후 채널 개수 N과 동일한 길이)를 정규화(합이 1이 되도록, `ε`로 0 나눗셈 방지)한다.
3. 정규화된 가중치를 concat된 feature map에 채널별로 dot product — 즉 각 채널에 자기 몫의 가중치를 곱한다.

> [!example]- 수식과 다른 재가중 전략과의 비교
> ```
> Output = Σ_j [ ω_j / (ε + Σ_m ω_m) ] · x_j     (CRC, 채널마다 학습된 가중치 하나씩)
> ```
> 비교 대상:
> - BiFPN 원조: 스케일(feature map) 단위로만 가중합 — 같은 feature map 안의 모든 채널이 같은 가중치를 공유.
> - SENet/ECANet류 channel attention: 입력에 따라 동적으로 가중치를 계산(추가 attention 서브네트워크 필요) — 파라미터·연산 비용이 CRC보다 큼.
> - "이중 재가중"(먼저 각 feature map 내부 채널을 재가중한 뒤, feature map 간에도 재가중): 표현력은 더 크지만 실험적으로 단일 재가중과 성능 차이가 거의 없어 굳이 쓸 필요가 없는 경우가 많음.

# 왜 이렇게 되는가
- **왜 스케일 단위가 아니라 채널 단위로 가중치를 주는가**: 서로 다른 스케일의 feature map이라도 그 안의 개별 채널이 담는 정보량·중요도는 균일하지 않다. 채널 단위로 가중치를 따로 학습하면, "이 스케일의 이 채널만 특히 중요하다"는 세밀한 조정이 가능해져 정보 손실 없이 융합할 수 있다.
- **왜 SENet류 channel attention 대신 고정 학습 가중치를 쓰는가**: Channel attention은 입력마다 가중치를 동적으로 계산하므로 표현력은 크지만 추가 conv·FC 연산이 필요해 비용이 늘어난다. 학습된 (입력에 무관한) 고정 가중치는 훨씬 적은 파라미터로 비슷한 효과를 낼 수 있어 경량성이 중요한 상황에서 유리한 트레이드오프다.

# 등장 논문
- [[FFCA-YOLO]] — FFM(Feature Fusion Module)의 핵심 재가중 전략. BiFPN 뼈대에 CRC를 적용해(3가지 변형 중 "먼저 concat 후 균일 채널별 가중치" 방식을 채택) 다중 스케일 feature를 손실 없이 결합.
