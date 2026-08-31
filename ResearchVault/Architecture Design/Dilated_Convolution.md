---
title: "Dilated Convolution"
tags: [architecture, convolution]
created: 2026-08-28
updated: 2026-08-28
---

# 역할
커널 원소 사이에 빈 간격(dilation)을 두어, **파라미터 수·계산량을 늘리지 않고도 수용영역(receptive field)만 넓히는** convolution. "구멍 뚫린(atrous) convolution"이라고도 부른다.

# 구조

## 입력/출력
- 입력: feature map `(C, H, W)`
- 출력: `(C', H, W)` (padding을 dilation에 맞춰 주면 공간 크기 유지 가능) — 표준 conv와 입출력 shape 형태는 동일하고, **커널이 실제로 입력을 읽는 간격**만 다르다.

## 내부 동작
- 표준 3×3 conv는 인접한 9개 픽셀을 본다.
- Dilation rate `d`를 적용한 3×3 conv는 커널 원소 사이를 `d`칸씩 띄워서 샘플링한다 — 예를 들어 `d=3`이면 커널이 실제로 덮는 영역은 3×3이 아니라 7×7 크기가 되지만, 학습되는 가중치 개수는 여전히 9개(3×3)뿐이다.
- 즉 "실질 수용영역은 커지지만 파라미터는 표준 3×3 conv와 동일"하다는 게 핵심.

> [!example]- 수식/그림 이해
> dilation rate `d`인 커널의 실질 크기(한 변) = `k + (k-1)(d-1)` (k=원래 커널 크기). 예: k=3, d=3 → 3+2×2=7, d=5 → 3+2×4=11, d=7 → 3+2×6=15.

# 왜 이렇게 되는가
- **왜 파라미터를 안 늘리고 수용영역을 넓히나**: 수용영역을 넓히는 다른 방법(더 큰 커널, 더 많은 레이어, pooling으로 다운샘플링)은 각각 파라미터 증가, 연산량 증가, 공간 해상도 손실이라는 비용이 따른다. Dilated convolution은 커널 크기·파라미터 수를 그대로 두고 "커널이 보는 범위"만 넓혀서 이 비용 없이 넓은 문맥을 본다.
- **왜 여러 dilation rate를 병렬로 쓰는 경우가 많은가**: 물체마다 크기가 다르므로, 작은 dilation(좁은 수용영역)과 큰 dilation(넓은 수용영역)을 병렬 브랜치로 두면 서로 다른 크기의 문맥 정보를 동시에 포착할 수 있다 — 이 특성 때문에 multi-scale feature 강화 모듈에서 자주 여러 rate를 병렬로 결합한다.

# 등장 논문
- [[BAFNet]] — DSAM(Dual-Stream Attention Module)에서 FPAM/BPAM으로 강조된 저레벨 feature를 rate 3/5/7의 병렬 dilated convolution 4개 브랜치(1×1 conv 포함)로 처리해, 서로 다른 크기의 전경·배경 문맥을 동시에 포착.
