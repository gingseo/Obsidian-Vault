---
title: "Frequency-Domain Feature Enhancement (FF module)"
tags: [concept, feature-fusion, frequency-domain, fourier-transform, small-object-detection]
created: 2026-08-04
updated: 2026-08-04
---

# 정의
CNN feature map을 FFT(Fast Fourier Transform)로 주파수 도메인으로 변환한 뒤, 학습 가능한 필터(주로 GAP + 1×1 conv)로 주파수 성분을 선택적으로 정제하고 IFFT로 다시 공간 도메인에 되돌리는 기법. 정제된 주파수-강화 feature와 원본 공간 feature를 학습된 게이팅 파라미터(α, β)로 가중합해 결합한다. 목적은 다운샘플링·feature fusion 과정에서 쉽게 소실되는 고주파(edge, texture 등 세부 디테일) 성분을 보존/복원하는 것으로, 특히 픽셀 수가 적어 세부 정보 의존도가 높은 소형 객체 탐지에 유효하다.

핵심 연산(단순화):
```
X_freq = IFFT( Conv1x1(GAP(X)) · FFT(X) )
X_out  = α · X_freq + β · X_spatial   (α, β는 학습 파라미터, residual 포함)
```

# 등장 논문
- [[UAV-DETR]] — 이 기법을 "Frequency-Focused (FF) 모듈"로 정식화해 제안한 논문. MSFF-FE(멀티스케일 feature fusion), FD(다운샘플링), SAC(서로 다른 fusion 경로 간 정렬) 세 곳 모두에 반복 삽입해 재사용하는 공통 빌딩 블록으로 사용.

# 변형/발전
- UAV-DETR(2025)에서 최초로 "FF 모듈"이라는 재사용 가능한 단위로 정식화됨. 논문 내에서도 세 가지 변형으로 응용됨:
  1. MSFF-FE 내부: 채널 attention과 멀티스케일(1×1/3×3/5×5/31×31) conv와 결합해 멀티스케일 주파수 강화에 사용.
  2. FD 내부: 다운샘플링 시 평균/맥스 풀링과 병렬 배치해 공간 축소 중 고주파 보존에 사용.
  3. SAC 내부: 서로 다른 fusion 경로의 feature를 정렬하기 전, 업샘플링된 feature를 주파수 강화하는 전처리 단계로 사용.
- 저자들은 관련 연구로 image restoration([16] Omni-kernel network)과 소형 폴립(polyp) segmentation([17] FTMF-Net)에서의 Fourier 기반 멀티스케일 융합을 언급하나, 이들은 멀티스케일 공간-주파수 결합까지는 다루지 못했다고 차별점을 주장함 — 즉 UAV-DETR 이전에도 개별 도메인에서 유사한 아이디어의 선행 시도가 존재.

# 관련 개념
