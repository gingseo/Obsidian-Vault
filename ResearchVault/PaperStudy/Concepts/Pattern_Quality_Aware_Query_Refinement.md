---
title: "Pattern Quality Aware Query Refinement"
tags: [concept, detr, dynamic-query, query-representation, one-to-many-assignment]
created: 2026-08-24
updated: 2026-08-31
---

> [!warning] 2026-08-31 전면 정정
> 이 문서는 원래 [[PaQ-DETR]]이 "dense candidate query를 공간적 유사도로 클러스터링해 병합(Pattern-Aware Clustering)하고, 객체다움 신뢰도 이하를 제거(Quality-Aware Pruning)한다"는 내용으로 작성되어 있었다. PaQ-DETR 논문 PDF(당시 경로는 vault 밖이었으나 현재는 `Projects/논문_pdf/Object_Detection/2025_arXiv_PaQ-DETR.pdf`)를 원문과 전체 대조한 결과, **이 클러스터링·pruning 메커니즘은 실제 논문에 존재하지 않는다** — PDF 전체 텍스트에 "candidate"/"prune"/"AI-TOD"/"aerial" 등의 단어가 전혀 등장하지 않으며, "PAC"라는 문자열은 실제로는 "com**pac**t"(compact set of shared patterns)의 일부일 뿐 별도 모듈명이 아니다. 실제 PaQ-DETR의 핵심은 (1) query를 소수의 공유 base pattern의 이미지 조건부 볼록결합으로 구성하는 것과, (2) 예측 품질에 따라 GT당 positive 수를 동적으로 정하는 one-to-many assignment이며, "병합"이나 "pruning" 개념 자체가 없다. 아래 내용은 실제 PDF 기준으로 전면 재작성했다. 이 개념을 인용하던 다른 문서(`ResearchVault/PaperStudy/Comparisons/Small_Object_Detection_Approaches.md`, `ResearchVault/PaperStudy/Moc/Small_Object_Detection_Moc.md`, `Small_Object_Detection_계보.md`)에도 같은 오류가 퍼져 있었다. `Projects/논문_Small_Object_Detection_tasks/DQA-DETR.md`, `DQ-DETR.md`, `IG-DETR.md`는 2026-08-31에 이미 정정 완료됐다.

# 정의
DETR 계열에서 object query를 **소수(m ≪ n)의 학습되는 공유 base pattern의 이미지 조건부 볼록결합(convex combination)** 으로 구성해 query 표현 측의 활성화 불균형(activation imbalance)을 완화하고, 동시에 **예측 품질(IoU–분류 신뢰도 일치도)에 따라 GT당 positive 샘플 수를 동적으로 정하는 one-to-many assignment**로 supervision 측 불균형을 완화하는, 표현과 공급(supervision) 두 측면을 함께 다루는 메커니즘. [[Density_Guided_Dynamic_Query]]가 이미지 전체의 밀도로 query "개수"를 조절하는 것과 달리, 이 개념은 query 개수는 고정한 채(n=300/900) query의 "내용을 구성하는 방식"과 "학습 신호가 분배되는 방식"을 이미지·예측 품질에 조건부로 동적화한다는 점에서 다른 축의 개념이다.

# 등장 논문
- [[PaQ-DETR]] — 원조. Content-Aware Weight Generator가 encoder feature로부터 만든 이미지 조건부 가중치 `W^D`로 base pattern `Q^P`를 볼록결합해 content query `Q^C`를 구성(Pattern-based Representation Module)하고, 중간 decoder layer에서 quality score(`s_i,j = IoU - γ·conf`) 기반으로 GT마다 다른 개수의 positive를 선정하는 Quality-Aware One-to-Many Assignment를 결합. DETR 계열 query activation 불균형(Gini 계수 최대 0.97)을 표현·공급 양쪽에서 동시에 완화해, PaQ-DINO 기준 Gini 0.97→0.89, COCO val2017 12-epoch mAP 50.3→51.9(DINO++ 대비)를 달성.

# 변형/발전
시간 순 정리(등장 논문이 늘어날 때마다 갱신):
- 2025(arXiv, v2 2026-03): PaQ-DETR — pattern 기반 볼록결합 query 구성 + quality-aware 동적 1:多 assignment의 원조. Diversity loss(Eq. 8)로 패턴 간 중복을 억제하고, 최종 decoder layer는 표준 1:1 matching을 유지해 추론 방식(NMS-free 등)은 그대로 보존.

# 관련 개념
- [[Density_Guided_Dynamic_Query]] — 둘 다 "DETR query를 이미지 내용에 따라 동적으로 조정한다"는 상위 목표를 공유하지만, 전자는 전역 밀도로 query "개수"를, 이 개념은 고정 개수 query의 "구성 방식"과 "supervision 분배"를 이미지·품질 조건부로 바꾼다는 점에서 다른 층위의 신호를 사용한다.
- [[Bipartite_Matching_Hungarian_Algorithm]] — 이 개념이 "구조적 불균형의 원인"으로 지목하는 DETR 표준 1:1 매칭. Quality-Aware One-to-Many Assignment는 중간 layer에서만 이 매칭을 확장하고 최종 layer는 그대로 유지한다.

