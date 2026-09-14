---
pm-task: true
projectId: "paperwiki-scientific-critique-automation"
parentId:
id: "t-toc-qy9rhsmji2"
title: "Tree-of-Concerns: Hierarchical Multi-Agent Debate for Unstated-Limitation Extraction in Scientific Critique"
type: "task"
status: "in-progress"
priority: "medium"
start: "2026-08-27"
due: ""
progress: 0
assignees: []
tags: []
subtaskIds: []
dependencies: []
createdAt: "2026-08-31T03:48:07.000Z"
updatedAt: "2026-09-14T01:21:47.486Z"
customFields:
  5hw2d9iih70nm11y: 2026
  g4v63dhvsx8j4vo9: "arXiv"
---

#paper #multi-agent-debate #llm-agent #scientific-critique #peer-review-automation #benchmark

> [!quote] 원제
> **Tree-of-Concerns: Hierarchical Multi-Agent Debate for Unstated-Limitation Extraction in Scientific Critique**
> Sahil Mishra, Niranjan Rajeev, Tanmoy Chakraborty — Department of Electrical Engineering, IIT Delhi, arXiv 2026
> https://arxiv.org/abs/2608.20777

# 한 줄 요약
<mark style="background: #FFF3A3A6;">논문이 스스로 밝히지 않은("unstated") 한계를 찾아내기 위해, scope·methodology·theoretical·reproducibility·fairness 5개 관점에 특화된 skeptic persona들이 각자 독립된 트리에서 4단계 적대적 논쟁(주장→반박→수정/철회→확장)으로 후보를 정제한 뒤, 사후 Panel Review가 교차 카테고리 중복·오분류·과장된 심각도를 재조정하는 Tree-of-Concerns 프레임워크를 제안하고, 414편 논문·1,905개 gold 미기재 한계로 구성된 ToC-Bench에서 최강 baseline 대비 Precision +79%, Coverage +11%를 달성한 논문.</mark>

# 정리

## 기존 방법의 한계
- **논문의 한계 과소 보고**:
  논문이 자신의 한계를 체계적으로 과소 보고한다(NeurIPS 2023 리뷰어 지적 약점의 73%가 논문의 Limitations 섹션에 없었다는 선행 분석 인용) — 이는 동료 심사자에게 과중한 부담을 주고, 유효 범위 밖에서 방법이 배포되어 조용한 실패·재현성 위기로 이어진다.

## 선행 연구는 어떻게 접근했고, 어떤 갭이 남았는가

**갈래 1 — Review-replication 계열**
- DIAGPaper, MARG, AgentReview: 리뷰어-저자 대화를 시뮬레이션해 human review의 스타일·범위를 *복제*하는 데 최적화.
- **타겟/해결**: 논문의 한계 과소 보고 문제를 겨냥하지만, 저자가 이미 자체 인정한 표면적 문제로 수렴 — 미기재 한계 발견에는 구조적으로 취약.

**갈래 2 — 동질적 멀티에이전트 debate**
- <mark style="background: #FFF3A3A6;">Tree-of-Debate 등: 여러 논문 간 pairwise 비교로 비판적 사고를 이끌어내는 멀티페르소나 debate tree.</mark>
- **타겟/해결**: <mark style="background: #FFF3A3A6;">교차 에이전트 조정을 지원하는 멀티에이전트 프레임워크 부재를 부분적으로 겨냥 — 동질적 에이전트의 pairwise 비교에 그쳐, 단일 논문 심층 비평·미기재 결함 발굴에는 적용되지 않음.</mark>

**갈래 3 — 벤치마크**
- PeerRead, NLPeer, AAAR-1.0: 리뷰 텍스트는 제공.
- **타겟/해결**: 기재되지 않은 한계 추출을 평가할 벤치마크 부재를 겨냥하지만, 기재/미기재를 분리하지 않고 per-claim 근거·타입 분류(taxonomy)가 없어 미기재 한계 추출을 엄밀히 평가할 수 없음.

**갭**: <mark style="background: #FFF3A3A6;">일반 LLM은 제약 없이 프롬프트되면 저자가 이미 스스로 인정한 것과 같은 뻔한 방법론·범위 문제로 쏠리고(critique groupthink), 이론·재현성·공정성처럼 덜 탐색되는 영역은 방치된다. 동질적 debate 프레임워크는 단일 논문 심층 비평에 맞지 않고, 벤치마크는 미기재 한계를 엄밀히 평가할 수단이 없다.</mark>

## 이 논문이 풀고자 하는 문제
1. 기재되지 않은 한계 추출을 평가할 벤치마크 자체가 없음(기존 데이터셋은 기재된/미기재 한계를 구분하지 않고 외부 검증된 gold annotation이 없음).
2. 단일 논문 비평에 교차 에이전트 조정을 지원하는 멀티에이전트 프레임워크가 없음(기존 Tree-of-Debate 등은 동질적 에이전트의 pairwise 비교에 그쳐 다양한 실패 모드를 못 찾음).

**갭 종합**: <mark style="background: #FFF3A3A6;">"한계를 찾아라"고 제약 없이 프롬프트된 LLM은 저자와 동일한 사고 패턴을 반복해 이미 알려진 문제로 수렴한다는 것이 핵심 통찰이며, 이 논문은 이 문제를 각 에이전트의 분석 렌즈를 좁게 강제하는 "전문화"로 깨뜨리고, 전문화가 낳는 새로운 부작용(카테고리 드리프트·중복)은 별도의 전역 조정 단계로 해결하는 2단 구조를 취한다.</mark>

> [!info] 내 메모
> 

# 해결 방법 요약

- **해결 방법**:
  5개 전문화된 skeptic persona가 병렬·독립 트리에서 각자의 실패-모드 렌즈로만 논문을 비평하게 강제해(specialization) 뻔한 문제로의 쏠림을 깨고, 각 후보 주장을 advocate의 반박에 노출시켜 근거 없는 주장을 걸러내며(tree-structured debate), 격리된 브랜치가 만드는 카테고리 중복·오분류를 사후 Panel Review로 교차 조정한다(cross-branch reconciliation).
- **예상되는 문제점**:
  5개 독립 트리 + 4단계 논쟁 + Panel Review까지 논문 1편당 다수의 LLM 호출이 필요해 비용·지연이 클 것으로 예상되며(논문도 Appendix Q에서 비용·지연을 별도 보고), 단일 논문 분석에 그쳐 여러 논문에 걸친 비교·재현 실패 같은 한계는 원천적으로 포착하지 못한다(저자도 Limitations에서 인정).

> [!info] 내 메모
> 

# 제안 방법

<mark style="background: #FFF3A3A6;">Scope·Methodology·Theoretical·Reproducibility·Fairness 5개 <span style="color:#c0392b; font-weight:bold;">전문화된 skeptic persona</span>가 서로 통신하지 않는 독립 병렬 트리에서 각자 논문을 분석해, 각 후보 한계 주장이 advocate와의 4단계 논쟁(주장→반박→수정/철회→모더레이터의 확장 판단)을 거치며 정제되는 <span style="color:#c0392b; font-weight:bold;">Tree-of-Concerns</span> 프레임워크를 구성하고, 모든 브랜치가 종료되면 <span style="color:#c0392b; font-weight:bold;">Panel Review</span>가 5개 관점을 동시에 고려해 살아남은 주장 각각에 Endorse/Merge/Downgrade/Reclassify/Reject 중 하나를 적용해 최종 출력을 만든다.</mark>

## 전체 파이프라인 (Fig. 2 기준)

```
입력: 논문 전문 (제목·초록·섹션·그림 캡션·참고문헌)
       │
       ▼
① 5개 skeptic persona 병렬 배정      → 5개 독립 트리 루트
       (Scope/Methodology/Theoretical/Reproducibility/Fairness,
        각자 저자가 이미 밝힌 한계는 무시하도록 지시됨)
       │
       ▼
② 트리 내 4단계 노드별 논쟁            → 노드마다 (claim, cat, sev, evidence quote)
       (Argue with Evidence → Advocate Response →
        Revise or Withdraw → Moderate and Expand)
       │  ※ 최대 depth 1까지만 확장 (child node 1개)
       ▼
③ 5개 트리의 생존 claim 전체 수집       → claim 후보 풀
       │
       ▼
④ Panel Review (단일 LLM 호출, 5개 관점 동시 고려)  → 최종 한계 목록 L
       (Endorse / Merge / Downgrade / Reclassify / Reject)
```

> [!info] 내 메모
>

### ① 5개 전문화 Skeptic Persona

- **역할**: 일반 LLM이 제약 없이 비평하면 저자와 같은 뻔한 문제(방법론·범위)로 쏠리는 것을 막기 위해, 각 persona를 좁은 실패-모드 렌즈 하나에만 묶어 서로 다른 종류의 미기재 결함을 발굴하게 한다.
- **구현**: 5개의 독립 system prompt로 구동되는 병렬 에이전트. 각 persona는 논문 전문(섹션 청크 단위)을 컨텍스트로 받고, "저자가 이미 밝힌 한계는 무시하라"는 stated-limitations filter 지시를 공통으로 받는다.
  - Scope Skeptic: 주장의 실험적 조건 밖 일반화 가능성, 데이터셋 커버리지, 언어 다양성.
  - Methodology Skeptic: 실험 설계, baseline 누락, 평가 지표, ablation 완결성.
  - Theoretical Skeptic: 형식적 주장, 증명 가정, 복잡도 분석, 논리적 정합성.
  - Reproducibility Skeptic: 하이퍼파라미터, 전처리, 연산 요구사항, 코드/데이터 공개 여부.
  - Fairness Skeptic: 사회적 영향, 편향, 이중 사용 가능성, 평가 subgroup 형평성.
- **입출력 shape**: 논문 전문(텍스트) → persona별 후보 한계 claim 집합(자연어 claim, category, severity, evidence quote 4-tuple).

<mark style="background: #FFF9D6A6;">이 좁은 렌즈 강제가 "정리" 표의 critique groupthink 문제를 해결한다 — No-Branching ablation(전문화 제거)에서 Coverage@10이 36.1%→7.6%, Novelty가 4.2→2.6으로 급락한 실험(Table 2)이 이 설계의 기여를 직접 증명한다.</mark>

> [!info] 내 메모
>

### ② Per-Node 4단계 적대적 논쟁

- **역할**: 단발성 생성은 근거 없는·쉽게 반박되는 주장을 걸러내지 못하므로(LLM이 존재하지 않는 baseline을 상상하거나 쉽게 반박되는 우려를 제기하는 경향), 후보 주장 하나하나를 논쟁 과정에 노출시켜 정제·강화한다.
- **구현**: 트리의 각 노드에서 4단계가 순차 진행된다.
  1. Argue with Evidence: skeptic이 논문 인용구(evidence quote)로 뒷받침되는 claim 생성 — 근거 없는 claim은 이후 단계에서 불리해지도록 설계.
  2. Advocate Response: 저자 관점을 취하는 advocate가 claim이 범위 밖·이미 다뤄짐·추측성이라고 반박.
  3. Revise or Withdraw: skeptic이 반박에 맞서 claim을 수정하거나 철회 — 흔한 수정은 미기재 제약 근거 추가, 심각도(major→minor) 하향.
  4. Moderate and Expand: 모더레이터가 수정된 claim이 (a) 더 깊은 이슈를 시사하는지, (b) 증거가 더 구체적인 하위 claim을 지지하는지, (c) 확장이 실제로 새 정보를 낳는지 3개 기준 중 2개 이상 충족 시 자식 노드 생성(depth 1까지).
- **입출력 shape**: 초기 claim 후보 → (모든 노드 순회 후) 생존 claim 집합, 각 claim은 (claim, category ∈ {scope, methodology, theoretical, reproducibility, fairness}, severity ∈ {major, minor}, evidence quote).

> [!example]- 구현 디테일
> ```
> for node in tree:
>     claim = skeptic.argue(paper_context, persona_lens)          # evidence-grounded
>     rebuttal = advocate.respond(claim, author_perspective=True)
>     revised_or_withdrawn = skeptic.revise_or_withdraw(claim, rebuttal)
>     if revised_or_withdrawn is not None:
>         if moderator.should_expand(revised_or_withdrawn, criteria=[
>             "deeper_unstated_issue", "specific_subclaim_evidence", "genuinely_new_info"
>         ]) >= 2 criteria met:
>             expand_child_node(revised_or_withdrawn)   # depth ≤ 1
> ```
> 실제 사례(Table 6): Case 1은 "broad applicability" 주장 → advocate 반박 → "harmonic-plus-noise 구조 한계·다중 악기 미평가"로 구체화되는 sharpening 과정을 보여줌. Case 2는 이론적 claim이 반박 앞에서 즉시 철회(concede)되는 사례로, 이런 case가 전체 노드의 13.6%를 걸러내 조작된 결함을 방지한다.

<mark style="background: #FFF9D6A6;">이 논쟁 구조가 "정리" 표의 "단발 생성이 근거 없는/쉽게 반박되는 주장을 걸러내지 못한다"는 문제를 해결한다 — No-Expansion ablation(4단계 중 확장 단계 제거)에서 Specificity가 4.0→2.8, Coverage@10이 36.1%→14.9%로 급락(Table 2)해, 반박에 노출시키는 구조 자체가 주장의 구체성·범위 모두에 기여함을 보여준다.</mark>

> [!warning] 이 구조 때문에 예상되는 문제점
> 5개 트리 × 4단계 논쟁 × 최대 depth 1 확장 → 논문 1편당 상당한 수의 LLM 호출이 필요하다(Appendix Q에서 비용·지연을 별도 보고할 정도). 실시간 심사 보조로 쓰기엔 비용·지연 트레이드오프가 존재할 것으로 보인다.

> [!info] 내 메모
>

### ③ Panel Review (사후 교차 조정)

- **역할**: 5개 트리가 서로 통신 없이 독립적으로 동작하는 것(brand independence)이 초반 수렴(premature convergence)은 막아주지만, 그 대가로 카테고리 드리프트(예: 이론 스켑틱이 scope 문제를 잘못 분류)와 브랜치 간 중복(같은 이슈를 여러 관점이 각기 다르게 지적)이라는 새 실패 모드를 만든다 — 이를 사후에 전역적으로 바로잡는다.
- **구현**: 생존한 모든 claim을 단일 LLM 호출이 5개 skeptic 관점을 동시에 고려해 재평가하며, 각 claim에 정확히 하나의 액션을 부여한다: Endorse(그대로 통과), Merge(여러 카테고리에 걸친 중복 기록), Downgrade(심각도 과장 완화), Reclassify(카테고리 오분류 재배정), Reject(무효/이미 기재됨/추측성 — 대부분의 무효 claim은 이미 per-node 논쟁에서 걸러져 이 단계까지 오는 비율은 낮음).
- **입출력 shape**: 5개 트리의 생존 claim 풀(총 908개 검토, 실제 실험 기준) → 최종 한계 목록 L = {(claim, category, severity, evidence)}.

> [!example]- 구현 디테일
> ```
> for claim in all_surviving_claims:
>     action = panel_llm.review(claim, all_5_persona_context=True)
>     if action == "Endorse": pass
>     elif action == "Merge": tag_cross_category_overlap(claim)
>     elif action == "Downgrade": claim.severity = "minor"
>     elif action == "Reclassify": claim.category = corrected_category
>     elif action == "Reject": drop(claim)
> ```
> 실제 검증 분포(Table 3, 908개 claim 기준): Endorse 56.1%, Merge 18.8%, Downgrade 17.1%, Reclassify 7.7%, Reject 0.2% — 수정률(Endorse 외) 43.8%. 거의 0에 가까운 Reject 비율(0.2%)은 "패널이 게이트키퍼가 아니라 캘리브레이터로 작동한다"는 저자 해석을 뒷받침(무효 주장은 이미 이전 단계에서 대부분 걸러짐).

<mark style="background: #FFF9D6A6;">이 조정 단계가 "정리" 표의 카테고리 드리프트·중복 문제를 해결한다 — ToC no-Panel(패널 제거) ablation에서 Precision이 40.3%→34.6%(-5.7pp), Validity가 4.3→3.9로 하락(Table 2)해, 패널이 실제로 정밀도·타당성에 순 기여함을 보여준다. Table 6의 Case 3(이론 스켑틱이 "k-NN 에피소드 보너스가 모든 state 커버리지를 보장한다"는 과장된 일반화를 형식적 증명 오류로 오분류한 것을 패널이 scope/minor로 재분류)은 이 메커니즘의 구체적 작동을 보여준다.</mark>

> [!info] 내 메모
>

## 파이프라인 정리표

| 단계 | 입력 | 출력 | 역할 | 구조/구현 |
|---|---|---|---|---|
| ① 5개 skeptic persona | 논문 전문 | 5개 독립 후보 claim 풀 | 전문화로 groupthink 방지 | 5개 병렬 system prompt, stated-limitations filter |
| ② Per-node 4단계 논쟁 | 초기 claim | 정제된 생존 claim (depth ≤1) | 근거 부실 주장 필터링·구체화 | Argue→Respond→Revise/Withdraw→Moderate |
| ③ Panel Review | 5개 트리의 생존 claim 전체 | 최종 한계 목록 | 카테고리 드리프트·중복 조정 | 단일 LLM, 5개 관점 동시 고려, 5개 액션 |

> [!info] 내 메모
>

# 실험 결과

### 핵심 결과 — Table 2 (ToC-Bench held-out 100편, LLM-as-Judge + Hybrid Likert)
**표를 보는 법**: Coverage@K는 gold 한계 중 상위 K개 예측 안에서 맞춘 비율, Precision은 예측 중 실제 유효한 비율, Hybrid Likert 3항목(Validity/Specificity/Novelty)은 1–5점 척도의 사람+GPT-4o+Claude 패널 평가.

| 벤치마크 | 지표 | Before(최강 baseline: Single-skeptic CoT) | After(ToC+Panel) |
|---|---|---|---|
| ToC-Bench | Coverage@10 / Precision | 32.6 / 22.5 | 36.1(+11%) / 40.3(+79%) |
| ToC-Bench | Validity / Specificity / Novelty (1–5) | 3.5 / 3.4 / 3.2 | 4.3 / 4.0 / 4.2 |

> [!note]- 세부 결과 및 Ablation
> #### 전체 비교 (Table 2)
> | Method | Cov@1 | Cov@5 | Cov@10 | Precision | Validity | Specificity | Novelty |
> |---|---|---|---|---|---|---|---|
> | Zero-shot LLM | 0.9 | 14.2 | 18.6 | 14.4 | 3.2 | 2.6 | 2.5 |
> | DIAGPaper | 2.3 | 7.0 | 14.5 | 11.7 | 2.7 | 2.5 | 2.3 |
> | Single-skeptic CoT | 8.5 | 23.7 | 32.6 | 22.5 | 3.5 | 3.4 | 3.2 |
> | No-Branching | 2.6 | 7.6 | 7.6 | 24.0 | 3.7 | 3.1 | 2.6 |
> | No-Expansion | 2.3 | 14.9 | 14.9 | 21.4 | 3.6 | 2.8 | 3.4 |
> | ToC no-Panel | 6.0 | 26.0 | 34.0 | 34.6 | 3.9 | 3.7 | 3.9 |
> | **ToC(전체)** | **11.5** | **27.2** | **36.1** | **40.3** | **4.3** | **4.0** | **4.2** |
> - review-replication 계열(DIAGPaper)이 전 지표 최저(Precision 11.7%, Validity 2.7) — 저자-리뷰어 대화 시뮬레이션은 이미 기재된 한계와 겹치는 장황하고 일반적인 피드백만 생성한다는 것을 보여줌.
>
> #### Panel Review 검증 분포 (Table 3, 908개 claim)
> | Panel Action | Count | % |
> |---|---|---|
> | Endorse | 510 | 56.1 |
> | Merge | 171 | 18.8 |
> | Downgrade | 155 | 17.1 |
> | Reclassify | 70 | 7.7 |
> | Reject | 2 | 0.2 |
> - 수정률 43.8% — 패널이 상당한 재조정을 수행하지만 게이트키퍼가 아니라 캘리브레이터로 작동(reject는 0.2%뿐).
>
> #### 근거(gold) 출처별 회수율 (Table 4, ToC no-Panel 기준)
> | Gold Source | Cov@10 |
> |---|---|
> | OpenReview weaknesses | 45.1% |
> | Citation critiques | 31.8% |
> | Combined | 34.0% |
> - 리뷰어가 지적한 약점이 인용 비판보다 1.4배 더 쉽게 회수됨 — 리뷰어 지적은 논문 텍스트만으로 식별 가능하지만, 인용 비판은 종종 후속 연구에 대한 소급 지식에 의존하기 때문(단일 논문 분석의 근본적 한계와 연결).
>
> #### 증거 인용 충실도 (Table 5)
> | Method | Verbatim % |
> |---|---|
> | ToC+Panel | 61.6% |
> | ToC no-Panel | 64.6% |
> | Zero-shot LLM | 77.4% |
> | Single-skeptic CoT | 78.2% |
> - ToC 계열이 오히려 verbatim(원문 그대로 인용) 비율이 낮은데, 저자는 이를 "여러 섹션의 근거를 종합해 응집된 표현으로 재구성하기 때문"이라고 해석(할루시네이션이 아니라 더 깊은 분석적 통합의 부산물).

> [!info] 내 메모
>

# Discussion

### 이 아이디어의 잠재적 부작용
- Panel Review가 카테고리 드리프트·중복은 교정하지만, <mark style="background: #FF5582A6;">패널 자체가 단일 LLM 호출 하나에 의존하므로 패널의 판단이 잘못될 경우(예: 실제로 유효한 claim을 잘못 Reject) 이를 다시 교정할 상위 메커니즘이 없다 — 논문은 이 위험을 직접 다루지 않는다.</mark>

### 한계
- <mark style="background: #FF5582A6;">저자가 Limitations에서 직접 명시: 단일 도메인 범위(ML/NLP/CV 벤치마크에서만 검증, 생물학·물리학 등 타 분야로 확장하려면 도메인 특화 persona와 카테고리 재설계 필요).</mark>
- <mark style="background: #FF5582A6;">텍스트 전용 모달리티 — 그림·표·수식·코드는 텍스트로 나타나는 한도 내에서만 처리되며, plot에 대한 시각적 추론이나 수학적 증명의 정식 검증, 공개된 코드 저장소의 정적 분석은 범위 밖.</mark>
- <mark style="background: #FF5582A6;">단일 논문 분석만 수행 — 여러 논문에 걸친 비교, 동시대 기여 간 충돌, 재현 실패, 커뮤니티 수준의 방법론적 드리프트처럼 교차 논문 비교가 필요한 한계는 원천적으로 포착 불가. 실제로 모든 방법이 놓친 gold 한계의 43%가 후속 연구에 대한 지식을 요구함(Appendix M).</mark>
- <mark style="background: #FF5582A6;">Gold 한계가 리뷰어 비판·인용 비판이라는 외부 근거에 기반한 하한(lower bound)일 뿐 완전한 상한이 아니다 — 보고된 Coverage/Precision은 보수적 추정치로 해석해야 한다고 저자가 명시.</mark>

### 생각할 점
- <mark style="background: #A6E3A1A6;">"전문화로 groupthink를 깨고, 전문화가 만드는 새 부작용을 전역 조정 단계로 다시 봉합한다"는 2단 구조는, 이 위키의 detection 계열이 "여러 개별 모듈로 문제를 분해했다가 마지막에 다시 통합·조정하는" 패턴(예: multi-branch feature 추출 후 fusion)과 상위 전략이 유사하다 — 도메인은 완전히 다르지만 "분해 후 재조정"이라는 설계 원리는 반복적으로 나타난다.</mark>
- <mark style="background: #A6E3A1A6;">Panel Review의 reject율이 0.2%로 극히 낮다는 것은, per-node 4단계 논쟁이 이미 대부분의 필터링을 수행한다는 뜻인데, 이는 "사후 검증 단계가 실제로 무엇을 잡아내는가"를 재는 좋은 진단 지표가 될 수 있다 — 이 위키의 다른 논문에서도 ablation 설계 시 "최종 단계가 얼마나 많은 사례를 실제로 바꾸는가"를 함께 보고하면 기여를 더 명확히 분리할 수 있을 것으로 보인다.</mark>

### 내 주제와 연관된 후속 연구 아이디어
- <mark style="background: #A6E3A1A6;">이 논문의 "저자가 이미 밝힌 것은 무시하고 명시적으로 강제된 렌즈로만 본다"는 stated-limitations filter 아이디어는, 컴퓨터 비전 논문을 정리하는 이 위키 자체의 워크플로우(Schema.md)에도 참고할 만하다 — 예컨대 "Discussion/한계" 섹션 작성 시 저자가 이미 명시한 한계와 별개로, 특정 관점(재현성, 공정성 등)을 강제로 훑어보는 체크리스트를 추가하면 이 위키의 한계 분석 깊이도 개선될 수 있다.</mark>

> [!info] 내 메모
>

# 관련 개념
(아직 없음 — 이 위키에서 LLM 멀티에이전트 비평 자동화를 다룬 첫 논문이라 재사용 가능한 concept 후보는 있지만, "전문화된 페르소나 기반 병렬 논쟁"은 이 논문 1편에서만 등장하는 구체적 응용이라 별도 concept 문서로 분리하지 않았다.)

# 관련 문서
- 비교: (아직 없음 — 이 위키에서 scientific-critique-automation을 다룬 첫 논문이라 비교 대상이 없음)

# 읽어볼 만한 논문
- 참고문헌 기반: P. Kargupta, I. Agarwal, T. August, J. Han, "Tree-of-debate: Multi-persona debate trees elicit critical thinking for scientific comparative analysis" (ACL 2025) — 이 논문이 "동질적 에이전트의 pairwise 비교에 그친다"고 직접 비교·차별화하는 가장 가까운 선행 연구. 멀티페르소나 debate tree 설계의 원조 격.
- 참고문헌 기반: M. D'Arcy, T. Hope, L. Birnbaum, D. Downey, "Marg: Multi-agent review generation for scientific papers" (arXiv 2401.04259, 2024) — 이 논문이 review-replication 계열의 대표 baseline으로 직접 비교하는 논문(본문에서 성능 최저로 언급됨).
- 참고문헌 기반: S. Kapoor 외, "Reforms: Consensus-based recommendations for machine-learning-based science" (Science Advances, 2024) — 미기재 한계가 초래하는 재현성 위기 문제의식을 뒷받침하는 인용 논문. 배경 이해에 도움.
- 자유 추천(검증 필요): retrieval-augmented 멀티페이퍼 비평(교차 논문 비교로 재현 실패·방법론적 드리프트를 탐지하는 후속 연구) — 검색 키워드: `retrieval-augmented multi-paper critique cross-paper comparison LLM 2026`. 이 논문이 Limitations에서 직접 "future work"로 지목한 방향이라 후속 연구가 나올 가능성이 높음.

---
**보안 참고**: PDF 전체를 확인했으며, 프롬프트 인젝션이나 지시문처럼 보이는 텍스트는 발견되지 않았다.

Project: [[논문_Scientific_Critique_Automation|Scientific Critique Automation]]