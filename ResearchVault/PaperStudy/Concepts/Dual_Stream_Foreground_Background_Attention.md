---
title: "Dual-Stream Foreground-Background Attention"
tags: [concept, attention-mechanism, remote-sensing, feature-fusion]
created: 2026-08-24
updated: 2026-08-24
---

# 정의
고레벨 semantic feature로부터 전경(foreground) 우선 attention map을 생성하고, 이를 전체 1 행렬에서 뺀 여집합을 배경(background) 우선 attention map으로 삼아, 별도 파라미터 없이 두 상보적 attention을 동시에 얻는 기법. 두 attention map을 각각 저레벨 spatial feature에 곱해 전경·배경을 모두 명시적으로 모델링한 뒤 결합함으로써, 전경 강조만 하는 단일 스트림 attention보다 배경과의 혼동을 더 적극적으로 억제한다.

# 등장 논문
- [[BAFNet]] — 원조. Dual-Stream Attention Module(DSAM)로 이 개념을 제안. 최고레벨 feature `P4`에서 FPAM을 생성하고 `BPAM = E - FPAM`으로 유도, 각각 dilated convolution 브랜치(rate 3/5/7)로 처리 후 결합.

# 변형/발전
시간 순 정리(등장 논문이 늘어날 때마다 갱신):
- 2025: BAFNet — 여집합 관계(`E - A^F`)로 배경 attention을 별도 학습 없이 유도. P4(최고레벨)가 P3보다 우수함을 ablation으로 확인.

# 관련 개념
- [[Remote_Sensing_Attention_Module]] — RS-TOD의 채널+공간 attention과 마찬가지로 detection head 주변에 attention을 삽입하지만, 이 개념은 전경뿐 아니라 배경까지 명시적으로 별도 모델링한다는 점에서 차별화된다.
