---
title: "Language-Guided Pre-Fusion Feature Enhancement"
tags: [concept, visual-grounding, multimodal, attention-mechanism]
created: 2026-08-24
updated: 2026-08-24
---

# 정의
Visual grounding에서 시각-언어 융합(fusion) 모듈에 들어가기 전에, 언어 feature를 query/key로 활용한 attention으로 시각 feature 자체를 미리 강화하는 기법. 전통적 방법이 언어와 시각 정보를 융합 단계에서만 상호작용시키는 것과 달리, 이 기법은 시각 feature 추출 단계와 융합 단계 사이에 별도의 언어 가이드 강화 단계를 둔다.

# 등장 논문
- [[VGRSS]] — 원조. LVFE(Language-guided Visual Feature Enhancement) 모듈로 이 개념을 제안. 시각 feature를 query, 언어 feature를 key/value로 하는 multihead self-attention을 3회 반복해 언어 정보를 점진적으로 시각 feature에 주입, 이후 원본 언어 feature와 채널 방향으로 concat(차원 압축 없이 공간 정보 보존).

# 변형/발전
시간 순 정리(등장 논문이 늘어날 때마다 갱신):
- 2025: VGRSS — LVFE(사전 강화)+VLF(공간 보존 융합) 2단계 분리 설계. Ablation에서 LVFE 단독(+2.15%p)보다 VLF 단독(+3.57%p)의 기여가 더 크지만, 결합 시 더 큰 시너지(+7.61%p)를 보임.

# 관련 개념
- [[Density_Guided_Dynamic_Query]] — 신호의 출처(언어 vs 밀도 맵)는 다르지만, "정제 대상이 되는 feature를 미리 보강해둔다"는 상위 전략을 공유한다. DQ-DETR 등의 CGFE가 density map으로 encoder feature를 미리 강화하는 것과 구조적으로 유사.
