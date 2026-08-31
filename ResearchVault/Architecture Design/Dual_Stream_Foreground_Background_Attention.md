---
title: "Dual-Stream Foreground/Background Attention"
tags: [architecture, attention]
created: 2026-08-28
updated: 2026-08-28
---

# 역할
고레벨(semantic이 풍부한) feature로부터 "여기가 전경(객체)인지"와 "여기가 배경인지"를 **동시에, 상호보완적으로** 나타내는 attention map 두 장을 만들어 저레벨(공간 디테일이 풍부한) feature에 각각 적용하는 기법. 전경만 강조하는 단일 attention과 달리, 배경을 명시적으로 억제하는 신호까지 별도로 만든다.

# 구조

## 입력/출력
- 입력: 고레벨 feature `P_high (C_high, H_high, W_high)`, 저레벨 feature `P_low (C_low, H_low, W_low)`
- 출력: 전경 강조 feature와 배경 강조 feature(둘 다 `P_low`와 같은 공간 해상도), 이후 이 둘을 합쳐 최종 강화 feature를 만든다.

## 내부 동작
1. `P_high`에 1×1 conv(→[[1x1_Convolution]]) + upsampling + sigmoid를 적용해 Foreground Priority Attention Map(FPAM) `A^F`(값 범위 0~1)를 만든다.
2. 전체 원소가 1인 행렬 `E`에서 `A^F`를 빼서 Background Priority Attention Map(BPAM) `A^B = E - A^F`를 얻는다 — **별도 파라미터·학습 없이** FPAM으로부터 상보적으로 유도.
3. 저레벨 `P_low`에 `A^F`, `A^B`를 각각 element-wise 곱해 전경 강조 feature와 배경 강조 feature를 얻는다.
4. 두 feature를 각각(또는 병렬 브랜치로) 추가 처리한 뒤 concat해 최종 강화 feature를 만든다.

> [!example]- 수식
> ```
> A^F = σ(Upsample(Conv1x1(P_high)))
> A^B = E - A^F
> fg_feature = P_low ⊙ A^F
> bg_feature = P_low ⊙ A^B
> ```

# 왜 이렇게 되는가
- **왜 배경 attention을 "여집합"으로 만드는가(별도 학습 안 함)**: 전경일 확률과 배경일 확률은 정의상 서로 배타적·상보적이므로(전경이 아니면 배경), 이미 계산한 FPAM에서 1을 빼는 것만으로 별도 파라미터 없이 유효한 배경 attention을 얻을 수 있다 — 계산·파라미터 비용을 늘리지 않고 상보적 신호 두 개를 얻는 경량 트릭.
- **왜 고레벨 feature로 attention을 만들고 저레벨 feature에 적용하는가**: 고레벨 feature는 여러 레이어의 pooling/downsampling을 거쳐 "이게 객체인지 배경인지"를 판단하는 semantic 정보가 풍부하지만 공간 해상도가 낮다. 저레벨 feature는 반대로 공간 디테일은 풍부하지만 semantic 판단력이 약하다. 고레벨에서 만든 "어디가 전경/배경인지" 지도를 저레벨의 세밀한 공간 정보에 씌우면, 두 특성을 모두 살릴 수 있다.
- **한계**: 배경 attention이 전경 attention의 단순 여집합이라, 전경 attention 자체가 부정확하면 배경 attention도 함께 부정확해지는 구조적 종속성이 있다 — 두 attention이 독립적으로 검증되지 않는다.

# 등장 논문
- [[BAFNet]] — Dual-Stream Attention Module(DSAM)의 핵심 기여. `P4`(최고레벨)로 FPAM/BPAM을 만들고 `P0`(최저레벨)에 적용, 이후 [[Dilated_Convolution]] 브랜치로 각각 처리해 전경·배경 문맥을 모두 반영한 강화 feature를 생성.
