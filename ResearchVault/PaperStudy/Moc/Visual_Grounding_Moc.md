---
title: "Visual Grounding MOC"
tags: [moc]
task: visual-grounding
created: 2026-08-24
updated: 2026-08-24
---

# 이 분야가 다루는 핵심 질문
- 자연어 표현으로 이미지 안의 특정 객체를 어떻게 정확히 지목(localize)할 것인가?
- 원격탐사 이미지 특유의 복잡한 배경·소형 타겟·클래스 간 높은 유사도 속에서, 언어와 시각 정보를 어느 단계에서 어떻게 상호작용시켜야 정밀한 grounding이 가능한가?

# 지금까지 다룬 흐름
아직 1편만 읽어서 "흐름"을 서술하기엔 이르다. 다음 논문이 추가되면 이 섹션을 갱신한다.
- [[VGRSS]] — 이 위키에서 visual grounding을 처음 다루는 논문. 원격탐사 선박 영상 특화 대규모 벤치마크(RSSVG, SARVG)를 자동화된 표현 생성으로 구축하고, 언어 정보로 시각 feature를 융합 이전에 미리 강화하는 LVFE와 공간 정보를 압축 없이 보존하는 VLF를 결합한 Transformer 기반 모델을 제안. 자연 이미지 VG 방법(TransVG, VLTVG 등)과 기존 원격탐사 VG 방법(RSVG/MGVLF)이 대형 타겟에 최적화되어 소형 선박에는 성능이 제한적이라는 갭을 메운다.

# 이 분야를 관통하는 개념
- [[Language_Guided_Pre_Fusion_Feature_Enhancement]] — VGRSS의 LVFE 핵심 기여. 융합 이전 단계에서 언어로 시각 feature를 미리 강화하는 전략.

# 비교 문서
(아직 없음 — 비교할 다른 visual grounding 논문이 추가되면 작성)

# 아직 못 채운 빈틈
- VGRSS가 반복 비교·인용하는 원조 원격탐사 VG 논문(RSVG/MGVLF, GeoVG)과 자연 이미지 VG 원조(TransVG, VLTVG)가 아직 이 위키에 없어, VGRSS가 실제로 무엇을 개선했는지 원본 논문 기준으로 재검증할 수 없다.
- Two-stage VG 방법(사전학습 detector 기반)이 이 위키에 아직 없어, one-stage/Transformer 계열과의 실질적 비교가 VGRSS의 서술에만 의존하는 상태.
- 이 위키의 small-object-detection 분야가 다루는 "완전성"(모든 객체를 찾는다) 문제와 visual grounding의 "특정성"(하나의 특정 객체만 찾는다) 문제 사이의 관계 — 예컨대 dynamic query DETR 계열의 밀도 기반 query 배정 아이디어가 visual grounding에도 적용 가능한지는 아직 다뤄지지 않았다.

# 관련 MOC
- [[000-Home]]
- [[Small_Object_Detection_Moc]] — 원격탐사 도메인과 소형 타겟이라는 문제의식을 공유하는 인접 분야. VGRSS의 LVFE가 dynamic query DETR 계열의 "feature 사전 강화" 패턴과 구조적으로 유사하다는 점에서 교차 참조 가치가 있다.
