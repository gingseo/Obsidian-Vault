---
title: "Cross Coding Twice Module (CCTM)"
tags: [concept, feature-fusion, detr, small-object-detection]
created: 2026-09-15
updated: 2026-09-15
---

# 정의
DETR류(transformer encoder-decoder) 구조에서, backbone feature $B$(fine-grained하지만 short-range 정보 위주)와 transformer encoder feature $E$(semantic하게 정제됐지만 attention을 거치며 blur·소형 객체 정보 손실 위험이 있는)를 **두 단계의 gating으로 교차 인코딩**해, encoder feature가 backbone feature의 세부 정보를 점진적으로 흡수하도록 만드는 기법.

- **1단계**: cross-attention에서 영감을 받은 gating — $B$, $E$ 각각을 Dense→LayerNorm→GELU로 변환한 gating map $B'$, $E'$을 만들고, 한쪽을 다른 쪽의 gating으로 재조합한 1차 cross feature($E^1_{cross} = E + B \cdot (1-E')$)를 만든다.
- **2단계**: GRN(Global Response Normalization)+MLP로 채널 방향 대비·선택성을 높인 뒤, 1단계에서 만든 crossing gating map($B' \cdot E'$)으로 backbone 정보를 한 번 더 선택적으로 반영해 최종 Cross Feature $E_{cf}$를 만든다.

단순 합/concat이 아니라 gating을 **두 번** 거치는 것이 핵심 — 무분별한 정보 결합이 아니라 "어떤 backbone 정보가 지금 encoder feature에 유용한지"를 두 단계에 걸쳐 적응적으로 선택한다.

# 등장 논문
- [[2025_TMM_Cross-DINO|Cross-DINO]] — 이 개념을 최초로 제안한 논문. DINO의 encoder feature blurring 문제(Fig. 6에서 시각적으로 확인)를 backbone feature 재주입으로 완화. Ablation(Table VIII)에서 gating 없는 MLP만 쓴 경우(AP 49.1)보다 1회 gating(49.4), 2회 gating(CCTM, 49.8) 순으로 개선되어 "두 단계 gating" 자체의 설계 기여를 실증.

# 변형/발전
(현재 이 논문 1편에만 등장. 향후 다른 DETR류 논문에서 유사한 backbone-encoder 재결합 기법이 등장하면 여기에 추가한다.)

# 관련 개념
- (없음 — UAV-DETR의 [[Frequency_Domain_Feature_Enhancement]]와 "정보 손실 지점에 원본 신호를 재주입한다"는 상위 전략은 유사하지만, 신호 소스(주파수 vs backbone feature)와 적용 지점(fusion/downsampling vs encoder-decoder 사이)이 달라 별도 개념으로 유지한다.)
