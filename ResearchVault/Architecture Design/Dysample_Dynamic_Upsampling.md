---
title: "Dysample (Point-based Dynamic Upsampling)"
tags: [architecture, upsampling, dynamic-sampling]
created: 2026-08-31
updated: 2026-08-31
---

# 역할
Nearest-neighbor 업샘플링(픽셀의 공간 위치만으로 커널을 결정, content 무시)과 CARAFE류 content-aware 업샘플링(dynamic convolution·별도 sub-network 필요, 연산 비용 큼) 사이의 절충으로, **"linear layer + pixel shuffle"만으로 샘플링 위치 offset을 동적으로 생성**해 저비용 content-aware 업샘플링을 구현하는 기법. 소형 객체에서 특히 두드러지는 pixel distortion·정보 손실을 줄이는 것이 목적이다.

# 구조
## 입력/출력
- 입력: feature map `x (c, h, w)`
- 출력: 업샘플된 feature map `x' (c, sh, sw)` — `s`는 업샘플링 배율.

## 내부 동작
1. **Offset 생성**:
   입력 feature `x`를 linear layer(출력 채널 `2s²`)에 통과시켜, 각 원본 픽셀 위치에 대한 `2s²`개의 값을 만든다.
2. **Pixel shuffle**:
   `2s²`채널을 pixel shuffle 연산으로 재배열해 `2×sh×sw` 크기의 샘플링 offset map(x, y 두 좌표)을 생성한다.
3. **Grid sample**:
   원본 sampling grid(정규 격자, G)에 이 offset(O)을 더해 새 sampling 위치를 만들고, 그 위치에서 입력 feature를 grid sample해 최종 `c×sh×sw` upsampled feature map을 얻는다.
4. **Static/Dynamic scope factor**:
   Offset의 영향 범위를 조절하는 scope factor를 고정값(static, 예: 0.25)으로 두거나, 입력에 따라 추가로 예측되는 값(dynamic, sigmoid 기반)으로 둘 수 있다 — 두 변형이 존재한다.

> [!example]- CARAFE·nearest-neighbor와의 비교
> - Nearest-neighbor: 픽셀 공간 위치만으로 업샘플링 커널 결정 — content 정보를 전혀 쓰지 않아 소형 객체에서 pixel distortion·정보 손실이 발생하기 쉬움.
> - CARAFE: content-aware하지만 dynamic convolution과 별도 sub-network가 필요해 연산 비용이 크다.
> - Dysample: dynamic convolution 없이 point(샘플링 위치) 자체를 동적으로 옮기는 방식이라, content-aware하면서도 연산 비용이 nearest-neighbor에 가깝다.

# 왜 이렇게 되는가
- **왜 point 기반인가**: Convolution 커널 자체를 동적으로 생성하는 대신, "어느 위치에서 값을 가져올지"만 동적으로 정하면 훨씬 적은 파라미터(linear layer 하나)로 content-aware 효과를 낼 수 있다.
- **왜 pixel shuffle을 쓰는가**: Sub-pixel convolution에서 쓰이는 pixel shuffle을 재사용해, 채널 차원에 있던 정보를 공간 차원(고해상도)으로 재배열한다 — 별도의 upsampling 연산(deconv, interpolation) 없이 배율 `s`만큼의 해상도 증가를 얻는다.
- **비용/트레이드오프**: Offset을 예측하는 linear layer 하나만 추가되므로 CARAFE 대비 파라미터·GFLOPs가 낮다. 다만 dynamic convolution만큼의 표현력(커널 형태 자체를 바꾸는 능력)은 없고, "샘플링 위치 이동"이라는 제한된 자유도만 갖는다.

# 등장 논문
- [[2025_ESWA_LSOD-YOLO|LSOD-YOLO]] — Neck의 feature fusion 단계에서 기존 nearest-neighbor 업샘플링을 대체. Table 7 ablation에서 nearest-neighbor(mAP0.5 36.7) · CARAFE(36.9, GFLOPs 35.8) 대비 Dysample(37.0, GFLOPs 33.9)이 정확도·연산량·FPS(93) 모두에서 우위를 보임.
