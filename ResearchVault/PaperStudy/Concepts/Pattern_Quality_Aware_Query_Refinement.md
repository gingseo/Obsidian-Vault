---
title: "Pattern Quality Aware Query Refinement"
tags: [concept, detr, dynamic-query, clustering, query-pruning]
created: 2026-08-24
updated: 2026-08-24
---

# 정의
DETR 계열 decoder의 dense candidate query 집합을, 개별 후보 간 공간적 유사도 기반 클러스터링(pattern-aware merging)과 객체다움 신뢰도 기반 pruning(quality-aware filtering)을 순차 결합해 이미지마다 다른 최종 개수·내용으로 동적 정제하는 메커니즘. [[Density_Guided_Dynamic_Query]]가 이미지 전체의 전역 밀도를 신호로 쓰는 것과 달리, 이 개념은 개별 후보 간의 관계와 개별 후보 자체의 품질이라는 인스턴스 레벨 신호를 직접 사용한다.

# 등장 논문
- [[PaQ-DETR]] — 원조. Pattern-Aware Clustering(PAC)으로 공간적으로 유사한 candidate query를 병합하고, Quality-Aware Pruning(QAP)으로 저품질 query를 제거. Tiny/aerial 특화가 아니라 COCO 일반 객체 탐지에서 검증한 유일한 dynamic query DETR 사례.

# 변형/발전
시간 순 정리(등장 논문이 늘어날 때마다 갱신):
- 2025: PaQ-DETR — dense candidate 생성 후 PAC(병합)→QAP(제거) 2단계 순차 정제. 전통적 NMS+confidence thresholding 파이프라인을 decoder 진입 이전 단계로 내재화한 것으로 해석 가능.
- 2026: DQA-DETR(참고 등장) — "등장 논문"에는 넣지 않았으나(핵심 개념은 [[Density_Guided_Dynamic_Query]] 쪽에 등록) 병합 메커니즘(Query Aggregator의 attention 기반 정보 흡수)이 이 개념과 유사한 인스턴스 레벨 처리를 수행. 다만 DQA-DETR은 PAC처럼 명시적 클러스터링이 아니라 rotated-NMS로 대표를 뽑은 뒤 cross-attention으로 흡수하며, QAP 같은 별도 pruning 단계가 없다는 점에서 차이.

# 관련 개념
- [[Density_Guided_Dynamic_Query]] — 둘 다 "DETR query를 이미지 내용에 따라 동적으로 조정한다"는 상위 목표를 공유하지만, 전자는 전역 밀도(이미지 레벨), 이 개념은 개별 후보 간 관계·품질(인스턴스 레벨)이라는 다른 층위의 신호를 사용한다는 점에서 구분된다.
