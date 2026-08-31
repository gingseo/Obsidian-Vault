---
title: "Density-Guided Dynamic Query"
tags: [concept, detr, dynamic-query, density-map, object-detection]
created: 2026-08-24
updated: 2026-08-24
---

# 정의
DETR 계열 decoder에서 고정된 개수·위치의 object query 대신, 입력 이미지로부터 추정한 density map(객체 밀도/위치 정보)을 이용해 query의 개수와 content·position을 이미지마다 동적으로 결정하는 메커니즘. 밀집 이미지에서는 query 수를 늘려 미검출(FN)을 줄이고, 희소 이미지에서는 query 수를 줄여 오탐(FP)과 연산 낭비를 줄인다.

# 등장 논문
- [[DQ-DETR]] — 원조. Density map으로 이미지 인스턴스 수를 4단계로 분류(회귀 대신 분류를 택함)해 query 개수(300/500/900/1500)를 선택하고, 동일 density map으로 encoder feature를 spatial+channel attention 보강한 뒤 top-K 선별로 query content·position을 생성.
- [[Density-Aware-DETR]] (D3Q) — DQ-DETR의 이산 분류를 crowd counting 기법 기반 연속적 density map 회귀로 대체. 점 단위 density focal loss로 supervision하고, static embedding+동적 위치 정보를 결합하는 dynamic mix selection으로 query 초기화. DQ-DETR과 직접 대조 실험(같은 DINO baseline 기준)으로 회귀 방식의 우위를 실증.
- [[IG-DETR]] — DQ-DETR의 4단계 분류를 6단계로 세분화(HIP 모듈). Feature 강화 시 곱셈 마스킹 대신 덧셈 residual 주입(`1+W`)을 써 semantic backbone 정보를 보존한다는 점이 차별점. Query 생성도 "salient seed" top-K 선택으로 구체화.
- [[DQA-DETR]] — DQ-DETR과 유사한 4단계 밀도 분류(ACP)를 쓰지만, 그 출력이 "최종 query 수"가 아니라 이후 병합(aggregation) 단계를 위한 대략적 사전(prior)일 뿐이라는 점에서 역할이 다르다. 실제 핵심 기여는 [[Pattern_Quality_Aware_Query_Refinement]]에 더 가까운 "유사 query를 attention으로 병합"(제거가 아님)이라는 메커니즘.
- [[DQP-DETR]] — Density map을 query 개수 결정뿐 아니라 encoder memory 강화(양방향 cross-modulation, BCME)와 토큰별 순위 결정(Ranking Consistency Supervision, RCS)까지 확장한 가장 포괄적인 사례. GT 밀도 기반 참조 우선순위로 "순위 결정 능력 자체"를 margin ranking loss로 직접 감독한다는 점이 이 계열에서 처음.

# 변형/발전
시간 순 정리(등장 논문이 늘어날 때마다 갱신):
- 2024: DQ-DETR — density map 기반 categorical counting(분류)으로 query 개수를 이산적으로 결정. 2단계 학습 필수.
- 2025: Density-Aware DETR(D3Q) — 분류를 연속 회귀로 대체(Instance Density Estimation + Density Focal Loss), end-to-end 학습 가능. Balance factor T로 query 개수를 조정하고 anchor L1 measure(log-ratio 손실)로 tiny object의 정규화 좌표 문제까지 함께 해결.
- 2026: IG-DETR — 이산 분류 구간을 4→6단계로 세분화. Feature enhancement의 결합 방식을 곱셈에서 덧셈 residual(`1+W`)로 바꿔 semantic 정보 손실을 방지. Query 생성에 "salient seed" top-K 선택이라는 명시적 단계 도입.
- 2026: DQA-DETR — 밀도 분류의 역할을 "최종 개수 결정"에서 "병합 단계를 위한 대략적 사전"으로 축소하고, 실제 query 수 축소는 유사 query를 attention으로 병합하는 방식([[Pattern_Quality_Aware_Query_Refinement]]과 유사한 인스턴스 레벨 처리)으로 달성 — 전역 밀도 신호와 인스턴스 레벨 병합을 함께 쓴 첫 사례.
- 2026: DQP-DETR — Density map을 (1) encoder memory 강화(BCME, 양방향), (2) query 개수 추정, (3) 개별 토큰 순위 결정(RCS) 세 곳 모두에 참여시켜 이 계열 중 가장 포괄적으로 확장. Pixel-level density regression과 token-level query 선택 사이의 간극을 margin ranking loss로 직접 감독한다는 점이 새로움.

이 4편(DQ-DETR, Density-Aware DETR, IG-DETR, DQP-DETR)을 관통하는 흐름: 밀도 신호를 "무엇에" 쓰는지가 점점 확장되어 왔다 — 처음엔 query 개수 산정(DQ-DETR)에만, 이어서 회귀로 정밀화(Density-Aware DETR), 세분화된 분류+additive feature 강화(IG-DETR), 마지막엔 encoder 표현 자체와 토큰별 순위 결정까지(DQP-DETR).

# 관련 개념
- [[Deformable_Sampling_Offset]] — 둘 다 "고정된 값을 입력 조건부로 바꾼다"는 상위 철학은 공유하나, 전자는 attention의 sampling location(위치)을, 이 개념은 query 자체의 개수·생성(양과 내용)을 대상으로 한다는 차이가 있다. DQ-DETR 등 이 계열 4편 모두 baseline으로 Deformable DETR을 채택해 두 개념이 실제로 같은 아키텍처 안에서 함께 쓰인다.
- [[Pattern_Quality_Aware_Query_Refinement]] — 둘 다 "DETR query를 이미지 내용에 따라 동적으로 조정한다"는 상위 목표를 공유하지만, 이 개념은 전역 밀도(이미지 레벨) 신호를, 후자는 개별 후보 간 관계·품질(인스턴스 레벨) 신호를 사용한다는 점에서 구분된다. DQA-DETR은 두 개념의 요소를 모두 갖춘 경계 사례.
