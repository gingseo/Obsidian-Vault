---
title: "Collaborative Receptive Field-Texture Optimization"
tags: [concept, receptive-field, texture-preservation, remote-sensing, feature-extraction]
created: 2026-08-24
updated: 2026-08-24
---

# 정의
수용영역 확장(전역 문맥 포착)과 texture 보존(국소 디테일 유지)이 근본적으로 상충한다는 관찰에서, 이 둘을 순차적으로 절충하는 대신 backbone의 feature 추출 단계부터 대·소 커널 병렬 브랜치로 동시에 추구해 trade-off 자체를 설계로 해소하는 프레임워크. 이후 단계에서 발생하는 잔여 texture 손실은 별도의 적응적 강화 모듈로 추가 보상한다.

# 등장 논문
- [[RTP-Net]] — 원조. GLEM(대·소 커널 depthwise conv 병렬 추출, backbone 소스 단계에서부터 개입) + AWEM(다중 pooling 기반 texture 적응적 복원) + MSAF(CBAM 기반 스케일 간 융합 정제) 3단계로 구현. GFLOPs를 오히려 27.2% 줄이면서 정확도도 개선한 예외적 사례.

# 변형/발전
시간 순 정리(등장 논문이 늘어날 때마다 갱신):
- 2026: RTP-Net — 병렬 대·소 커널(5×5/7×7 depthwise separable convolution)로 소스 단계 trade-off를 해소, CRM(고/저주파 성분을 spatial domain에서 residual 근사로 분리)으로 주파수 도메인 FFT 없이 저비용 고/저주파 분리.

# 관련 개념
- [[Dual_Stream_Foreground_Background_Attention]] — BAFNet의 접근과 마찬가지로 "넓게 보되 세밀함을 잃지 않기"라는 상위 딜레마를 다루지만, 이 개념은 backbone 소스 단계에서 병렬 커널로 대응하는 반면 BAFNet은 fusion 단계에서 전경/배경 attention과 경계 supervision으로 대응한다는 점에서 개입 층위가 다르다.
- [[Frequency_Domain_Feature_Enhancement]] / [[Frequency_Domain_Feature_Attention]] — FFT 기반 주파수 분리 대신 spatial domain의 large-kernel down/up-sampling residual로 고/저주파를 근사한다는 점에서, 같은 목표(고주파 texture 보존)를 저비용으로 달성하려는 대안적 설계로 볼 수 있다.
