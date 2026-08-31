# PaperWiki Schema

이 문서는 `ResearchVault/PaperStudy/`(Concepts/, Comparisons/, Moc/, reading-list.md)와 `Projects/`(task별 프로젝트 노트, `논문_<Task>_tasks/` 논문 분석 노트 폴더) 그리고 vault 밖 `/Users/GyeongSeo/Workspace/논문_pdf/`(논문 PDF 원본) 아래 문서를 생성·수정할 때 지켜야 하는 규칙이다.
Claude Code에게 위키 업데이트를 요청할 때, 이 문서가 유일한 규칙 소스다.

> [!note] PDF 원본은 vault 밖에 있다
> 논문 PDF 원본은 용량 문제로 Obsidian vault(`GingseoLife/`) 밖의 `/Users/GyeongSeo/Workspace/논문_pdf/`에 별도 보관한다. 분석 노트(task 노트)는 여전히 vault 안 `Projects/논문_<Task>_tasks/`에 있다. `/Users/GyeongSeo/Workspace/논문_pdf/` 아래 파일은 Obsidian `[[wiki-link]]` 대상이 아니므로(vault 밖이라 링크가 걸리지 않는다), 노트에서는 `source` frontmatter에 절대경로 문자열로만 참조한다.

## 폴더 구조

```
ResearchVault/PaperStudy/
  Concepts/              개념/기법 1개 = 파일 1개 (여러 논문에서 공유되는 개념, task로 나누지 않음)
  Comparisons/            논문 2편 이상을 특정 축으로 비교하는 문서 (task로 나누지 않음)
  Moc/                    Map of Content — task별 서사/맥락 허브 + 전체 홈
  reading-list.md          task별로 정리된 "읽어볼 만한 논문" 모음. 논문을 실제로 읽어 task 노트가 생기면 여기서는 제거한다
ResearchVault/MD_Files/
  README.md                사용법
  Schema.md                이 문서

Projects/                볼트 최상위. 논문 분석 노트는 여기 있다 (PDF 원본은 vault 밖, 아래 참고)
  논문_<Task>.md              task별 Project Manager 프로젝트 노트 (예: 논문_Small_Object_Detection.md, 논문_Anomaly_Detection.md). task 개수만큼 존재한다
  논문_<Task>_tasks/          그 task 프로젝트에 속한 논문 분석 노트(task 노트)들 (논문 1편 = 파일 1개). 반드시 `<프로젝트파일명>_tasks` 형식이어야 한다(아래 참고)
  논문_PaperWiki.base         모든 논문_<Task>_tasks/ 논문 노트를 가로질러 테이블로 보여주는 Obsidian Bases 뷰 정의. task/status 등으로 필터링은 여기서 한다

/Users/GyeongSeo/Workspace/논문_pdf/     vault 밖. 논문 PDF 원본 (용량 문제로 vault 밖에 보관)
  _inbox/                  아직 처리 안 한 원본 PDF (다운받으면 여기에 넣는다)
  <Task>/                   처리 완료 후 이동되는 task별 PDF 폴더
  _issue_paper/             (참고용, 처리 대상 아님) 직접 읽으려는 논문이 아니라 커뮤니티 등에서 이슈가 된 논문을 트렌드 파악용으로 모아두는 별도 폴더. `/process-papers` 워크플로우가 스캔하는 대상이 아니다.
```

- 새 PDF는 항상 `/Users/GyeongSeo/Workspace/논문_pdf/_inbox/`에 넣는다.
- 처리 완료되면 PDF 파일 자체를 `/Users/GyeongSeo/Workspace/논문_pdf/<Task>/`로 **이동**한다 (`_inbox/`에는 남기지 않는다). "처리됐는지"는 이제 폴더 위치로도 바로 보인다 — `_inbox/`에 남아있으면 미분류, `<Task>/`에 있으면 처리 완료.
- `<Task>` 이름은 논문의 목적/과제를 가리키며, 아래 "폴더·파일 네이밍 규칙"을 따른다 (예: `Small_Object_Detection`, `Anomaly_Detection`). 기존에 없던 task면 프로젝트 노트(`Projects/논문_<Task>.md`)와 논문 노트 폴더(`Projects/논문_<Task>_tasks/`)를 함께 새로 만든다. 폐쇄 목록이 아니다 — 논문을 보고 적절한 task가 없으면 새로 만든다. `/Users/GyeongSeo/Workspace/논문_pdf/<Task>/`(PDF 쪽)와 `Projects/논문_<Task>_tasks/`(노트 쪽)는 폴더명 규칙이 다르다는 점에 유의한다 — PDF 폴더는 접미사 없이 `<Task>` 그대로, 노트 폴더는 반드시 `<Task>_tasks`.
- **`논문_<Task>_tasks/` 폴더명은 `_tasks` 접미사가 필수다.** Project Manager 플러그인이 프로젝트 노트 `Projects/논문_<Task>.md`의 task를 스캔할 때 폴더 경로를 `파일 경로에서 .md를 _tasks로 치환`해서 계산하도록 하드코딩되어 있다(플러그인 소스 `projectTaskFolder`). 즉 `논문_Small_Object_Detection.md`의 task 폴더는 반드시 `논문_Small_Object_Detection_tasks/`여야 하며, 접미사가 없거나 다르면(예: `논문_Small_Object_Detection/`) Project Manager가 그 폴더를 전혀 스캔하지 않아 프로젝트의 task 개수가 0으로 표시된다.
- 논문 하나가 여러 task에 걸치면(드묾) 가장 핵심적인 task 하나의 프로젝트·폴더에만 PDF와 노트를 두고, task 노트의 `task` 속성에는 해당되는 task를 전부 적는다.
- 논문 분석 노트는 `ResearchVault/PaperStudy/` 안에 있지 않다 — Project Manager 플러그인과 통합 관리하기 위해 볼트 최상위 `Projects/논문_<Task>_tasks/`에 둔다. 자세한 내용은 아래 "Projects/논문_<Task>.md — 논문 분석 노트" 절 참고. `Concepts/`와 `Comparisons/`는 여러 task에 걸치는 경우가 많으므로 task로 나누지 않고 `PaperStudy/` 안에 폴더 하나로 모아둔다.

## 폴더·파일 네이밍 규칙

이 위키 전체(폴더명, task명, moc/concept/comparison 파일명)에 적용되는 대소문자·구분자 규칙이다. PDF 원본 파일명("PDF 파일명 규칙" 절)과 논문 슬러그("슬러그 대소문자 규칙" 절)는 이 규칙보다 우선하는 별도 규칙을 따른다.

- **일반 서술형 이름**(task 폴더명, moc/concept/comparison 파일명처럼 저자가 붙인 고유 표기가 없는 이름): 단어마다 첫 글자를 대문자로 하고 밑줄(`_`)로 구분한다. 하이픈(`-`)은 쓰지 않는다.
  예: `Small_Object_Detection`, `Detection_Oriented_Rectification`, `Small_Object_Detection_Moc`
- **논문 고유 슬러그**(저자가 스스로 붙인 약칭이 포함된 이름, 아래 "슬러그 대소문자 규칙" 참고)는 이 규칙 대신 다음 세부 규칙을 따른다:
  - 약어(원 논문이 두문자로 쓰는 표기, 예: `DETR`, `SR`, `TOD`, `CDATOD`, `FFSSTD`)는 전체 대문자로 쓴다.
  - 약어가 아닌 일반 단어(예: `Net`, `Reconstruction`)는 첫 글자만 대문자로 쓴다.
  - 약어와 약어가 이어질 때는 하이픈(`-`)으로 구분한다. 예: `SR-TOD`, `CDATOD-Diff`(`Diff`는 일반 단어이므로 대문자는 첫 글자만이지만, `CDATOD`라는 약어와 결합하므로 하이픈으로 구분).
  - 약어와 일반 단어, 또는 일반 단어끼리 이어질 때는 붙여 쓴다. 예: `FFSSTDNet`(`FFSSTD` + `Net`).
- 두 규칙이 겹치는 경우(예: 약어만으로 이루어진 task성 이름)는 판단이 애매하면 사용자에게 확인한다.

## 공통 규칙

- 모든 문서는 Obsidian `[[wiki-link]]` 문법으로 서로 연결한다. 링크 대상 파일이 아직 없으면 만들지 말지 판단하고(아래 concepts 규칙 참고), 없는 링크를 남발하지 않는다.
- 아직 위키에 없는 논문을 언급하며 "나중에 링크 걸 대상"으로 남겨둘 때는 반드시 `#pending:<논문-슬러그>` 마커를 그 문장에 붙인다 (예: `RFLA #pending:rfla`). 이 마커는 나중에 그 논문이 실제로 추가될 때 `grep -r "#pending:<슬러그>" .`(`ResearchVault/PaperStudy/`와 `Projects/` 양쪽에서 실행)로 미완성 링크를 찾아 갱신하기 위한 것이다 — 전체 위키를 다시 읽지 않고 문자열 검색만으로 찾을 수 있게 하는 장치이므로, 마커 없이 "아직 없음"이라고만 적어두지 않는다.
- 내용은 한국어로 쓴다. 논문 제목, 저자명, 고유 용어(모델명, 메서드명)는 원문(영문) 그대로 둔다.
- 각 문서 frontmatter의 `tags`류 값은 소문자 kebab-case로 통일한다 (파일명·폴더명 네이밍 규칙과는 별개).
- 근거 없는 내용을 만들어내지 않는다. PDF에서 확인 안 되는 정보는 채우지 말고 빈칸으로 둔다.
- 볼트 전체에서 같은 파일명이 이미 쓰이고 있지 않은지 새 파일을 만들 때마다 확인한다 — Obsidian은 파일명 기준으로 `[[링크]]`를 해석하므로, 중복 파일명이 있으면 링크가 엉뚱한 파일을 가리킬 수 있다.

## PDF 파일명 규칙

`/Users/GyeongSeo/Workspace/논문_pdf/_inbox/`의 PDF를 처리할 때, 이동하기 전에 다음 규칙으로 리네임한다.

```
{년도}_{학술지/학회/venue}_{제목}.pdf
```

- 제목은 논문에 short title(예: 부제 앞의 짧은 이름, 저자들이 스스로 붙인 약칭)이 있으면 그것을 쓰고, 없으면 논문 제목 전체를 쓴다. 공백은 하이픈(`-`)으로.
- venue는 학회/저널 약칭을 쓴다 (예: `CVPR`, `NeurIPS`, `arXiv`). 확실하지 않으면 PDF에 적힌 대로 쓴다.
- 예: `2017_NeurIPS_Attention-Is-All-You-Need.pdf`, `2016_CVPR_Deep-Residual-Learning.pdf`

## Projects/논문_<Task>.md — 논문 분석 노트

논문 노트는 `ResearchVault/PaperStudy/` 안에 있지 않다. Project Manager 플러그인(볼트 전체 프로젝트/작업 관리)과 통합 관리하기 위해, 논문 1편 = **Project Manager의 task 노트 1개**로 볼트 최상위 `Projects/` 아래에 둔다. 원문 요약(Abstract/Introduction/Conclusion 번역) 파일은 만들지 않는다 — 사용자가 원문을 직접 읽고 정리하는 쪽을 선호한다.

### 프로젝트 구조

- **task(분야)마다 별도 Project Manager 프로젝트**를 둔다: `Projects/논문_<Task>.md`(`pm-project: true` frontmatter, 예: `논문_Small_Object_Detection.md`, `논문_Anomaly_Detection.md`). Project Manager는 `Projects/` 바로 아래에 있는 프로젝트 노트만 인식하므로(하위 폴더 재귀 스캔 안 함), 프로젝트 노트 자체는 항상 `Projects/` 바로 밑에 둔다.
  - 예전에는 모든 논문을 담는 단일 프로젝트(`논문 읽기.md`) 하나만 뒀었다. 하지만 Project Manager UI(Table/Kanban)가 프로젝트 하나를 열면 그 안의 모든 task를 분야 구분 없이 한 화면에 나열해서, 특정 분야에 집중해서 보기가 어려웠다 — 그래서 프로젝트 자체를 task 개수만큼 나눴다. Project Manager UI에서 프로젝트를 전환하는 것으로 분야를 구분해서 본다.
  - task가 새로 생기면 프로젝트 노트도 함께 새로 만든다(아래 "새 task 노트 생성 시 프로젝트 노트도 함께 갱신" 절 참고).
- 논문 노트(task 노트): `Projects/논문_<Task>_tasks/<논문-슬러그>.md` — 그 task 프로젝트에 속한 논문 노트들을 모아두는, 프로젝트 노트와 이름이 같은 폴더. 이 폴더는(프로젝트 노트와 달리) 재귀적으로 스캔되므로 하위 폴더로 둬도 Project Manager가 정상 인식한다. 어느 분야인지는 어느 프로젝트에 속하는지(`projectId`)와 노트의 `task` 속성 둘 다로 확인 가능하고, `Projects/논문_PaperWiki.base`에서 `task` 컬럼으로 필터링해서도 본다(아래 "상태 확인하기" 절 참고).
- 논문 하나가 여러 task에 걸치면(드묾), 가장 핵심적인 task 하나의 프로젝트·폴더에만 노트를 두고 `task` 속성에 해당되는 task를 전부 배열로 적는다.

파일명(슬러그): `<논문-슬러그>.md` (슬러그는 예: 저자 성 + 핵심 키워드, 또는 잘 알려진 약칭. 리네임한 PDF의 제목 부분을 재사용해도 된다)

**슬러그 대소문자 규칙**: 논문에 저자가 스스로 붙인 short title(예: "QueryDet: Cascaded Sparse Query for..."처럼 제목 앞부분에 오는 약칭)이 있으면, 그 표기를 대소문자까지 정확히 그대로 슬러그로 쓴다 (`FANet`, `LSOD-YOLO`, `QueryDet`, `RS-TOD`, `UAV-DETR`, `Unc-SOD`, `ReContrast`처럼). 전부 소문자로 뭉뚱그리지 않는다 — 실제 논문에서 쓰는 표기가 그 자체로 식별성을 갖기 때문이다(예: `ReContrast`를 `recontrast`로 쓰면 다른 논문 제목의 일부처럼 보일 수 있다). 원문에 명시적 short title이 없으면(예: 제목이 일반 서술형 문장인 논문) 슬러그는 위 "폴더·파일 네이밍 규칙"의 일반 서술형 이름 규칙(`Word_Word`, 예: `Detection_Oriented_Rectification`)을 따른다. 애매하면 PDF를 다시 확인해 저자가 스스로 어떻게 부르는지(본문에서 반복 사용하는 모델/프레임워크 이름) 찾아보고, 그래도 불명확하면 사용자에게 묻는다 — 나중에 슬러그를 바꾸면 위키 전체의 `[[링크]]`를 전수 갱신해야 하는 비용이 크므로 처음에 정확히 정하는 편이 낫다.

**파일명과 title은 항상 동일한 이름(위 슬러그 규칙 기준)으로 통일한다** — short title이 있으면 파일명·`title` 필드 둘 다 그 short title로, short title이 없으면 둘 다 논문 원제로 채운다. `title`에 연도·venue 등 접두어를 붙이지 않는다(그 정보는 `year`/`venue`/`customFields`가 이미 담당). 원제 전체(특히 short title로 줄였을 때 사라지는 부제까지)는 본문 최상단의 `> [!quote] 원제` 콜아웃에 남긴다(아래 "본문 구성" 절 참고).

> [!warning] Project Manager의 Task 편집 모달을 통해 title을 저장하면 파일명·frontmatter가 깨질 수 있다
> Project Manager는 Task 모달에서 title을 저장할 때마다 **파일명을 title 기반 slug(소문자+하이픈, 최대 60자)로 자동 리네임**하고, 이 동작을 끄는 설정이 없다(플러그인에 하드코딩됨). 자동 리네임되면 task 폴더 밖으로 파일이 튀어나오고, 지정한 슬러그(`QueryDet` 등)가 망가진다. 더 나아가 **PaperWiki 고유 속성(`year`/`venue`/`jcr_quartile`/`task`/`direction`/`paper_tags`/`source`)이 저장 과정에서 통째로 사라지는 사고가 실제로 관찰됐다** — Project Manager가 자신이 아는 스키마 필드만 다시 써서 frontmatter를 재구성하고, 모르는 커스텀 필드는 버리는 것으로 보인다. 따라서 **Project Manager UI에서 논문 task의 title은 편집하지 않는다.** title을 고칠 필요가 있으면 Obsidian 편집기로 파일을 직접 열어 frontmatter만 수정한다. 만약 실수로 UI에서 title을 저장해버렸다면: (1) 파일명과 폴더 위치를 원래대로 되돌리고, (2) frontmatter에 PaperWiki 속성이 남아있는지 확인해서 없으면 이 문서와 해당 `Projects/논문_<Task>.md`의 링크 텍스트(원제)를 참고해 복구한다.

### Frontmatter

Project Manager의 task 스키마(`pm-task`, `projectId`, `id`, `type`, `priority`, `progress`, `assignees`, `subtaskIds`, `dependencies`, `createdAt`, `updatedAt` 등)에 아래 PaperWiki 고유 속성을 **frontmatter 최상위에 평평하게** 추가한다. PaperWiki 고유 속성(`year`/`venue`/`jcr_quartile`/`task`/`direction`/`paper_tags`/`source`)은 Project Manager의 `customFields`(중첩 객체) 안에 넣지 않는다 — Obsidian Bases는 중첩된 객체를 컬럼별로 필터링하지 못하기 때문에, `논문_PaperWiki.base`에서 `venue`/`year`/`jcr_quartile` 등을 각각 필터링하려면 반드시 최상위 키여야 한다. 단, Project Manager UI의 Table 뷰(Bases가 아니라 Project Manager 자체 뷰)에서 Year/Venue를 컬럼으로 보기 위해 사용자가 프로젝트별로 추가한 `customFields`(Year/Venue/Summary/Tags)는 최상위 속성과 별개로 병행 사용한다 — 최상위 `year`/`venue`가 정본이고, `customFields`의 Year/Venue는 Project Manager UI 표시용으로 같은 값을 중복 기입한다(아래 "Frontmatter" 절의 `customFields` 항목 참고).

```yaml
---
pm-task: true
projectId: "<프로젝트의 id>"
parentId:
id: "<고유 id>"
title: "<논문 원제>"
type: "task"
status: to-do | in-progress | additional-study-needed | done
priority: "medium"
start: "<YYYY-MM-DD>"
due:
progress: 0
assignees: []
tags: []
customFields:
  "<프로젝트의 Year 필드 id>": <YYYY>
  "<프로젝트의 Venue 필드 id>": "<venue 약어>"
subtaskIds: []
dependencies: []
year: <YYYY>
venue: "<학회/저널/arXiv>"
jcr_quartile: Q1 | Q2 | Q3 | Q4 | arXiv | Workshop | null
task: [<task1>, <task2>, ...]
direction: [<direction1>, <direction2>, ...]
paper_tags: [paper, <세부 주제 태그...>]
source: "/Users/GyeongSeo/Workspace/논문_pdf/<Task>/<리네임된 파일명>"
createdAt: "<ISO 8601>"
updatedAt: "<ISO 8601>"
---
```

- `title`: 논문 원제만 쓴다(연도·venue 접두어를 붙이지 않는다).
- `customFields`: Project Manager UI에서 프로젝트별로 추가한 커스텀 필드 값을 담는다(키는 그 프로젝트 노트의 `customFields` 정의에 있는 필드 `id`). 각 task 프로젝트(`Projects/논문_<Task>.md`)는 `Year`(number), `Venue`(text), `Summary`(text), `Tags`(multiselect) 4개 커스텀 필드를 갖고 있고, 새 논문 노트를 만들 때 그 프로젝트의 Year/Venue 필드 id를 찾아 `year`/venue 약어 값을 채운다. field id는 해당 `Projects/논문_<Task>.md`의 `customFields:` 블록에서 확인한다(프로젝트마다 다른 임의 id이므로 재사용하지 않는다).
  - Venue customField에 채우는 약어는 `venue` 필드 값 중 괄호 안에 이미 약어가 있으면 그 약어만 쓴다 (예: `venue: "IEEE Journal of Selected Topics in Applied Earth Observations and Remote Sensing (JSTARS)"` → `JSTARS`).
  - venue에 괄호가 없거나 괄호 안이 약어가 아니라 출판사명이면(예: `Sensors (MDPI)`, `Expert Systems With Applications`), venue의 첫 단어/핵심 식별어만 추출해서 쓴다 (예: `Sensors`, `Expert Systems With Applications` 그대로, `Neural Networks (Elsevier)` → `Elsevier`).
  - 자주 등장하는 venue → 약어 매핑은 아래 표를 따르고, 표에 없는 새 venue를 만나면 위 두 규칙으로 판단해 표에도 추가한다.

  | venue (frontmatter) | Venue customField에 쓸 약어 |
  |---|---|
  | CVPR | CVPR |
  | CVPRW (CVPR Workshops) | CVPRW |
  | NeurIPS | NeurIPS |
  | ECCV | ECCV |
  | ICCV | ICCV |
  | ICLR | ICLR |
  | ICASSP | ICASSP |
  | IEEE TIP | IEEE TIP |
  | IEEE TPAMI | IEEE TPAMI |
  | IEEE TGRS / IEEE Transactions on Geoscience and Remote Sensing (TGRS) | TGRS |
  | IEEE Journal of Selected Topics in Applied Earth Observations and Remote Sensing (JSTARS) | JSTARS |
  | Remote Sensing (MDPI) | MDPI |
  | Sensors (MDPI) | Sensors |
  | Image and Vision Computing (Elsevier) | Elsevier |
  | Neural Networks (Elsevier) | Elsevier |
  | Remote Sensing Applications: Society and Environment (Elsevier) | Elsevier |
  | Expert Systems With Applications | Expert Systems With Applications |
  | SSRN (preprint, submitted to Elsevier, not peer-reviewed) | SSRN |
  | arXiv | arXiv |

- `tags`(빈 배열)는 Project Manager 자체 태그 시스템 필드이므로 건드리지 않는다. PaperWiki의 주제 태그는 이름이 겹치지 않도록 **`paper_tags`**에 넣는다.
- `task`: 이 논문이 다루는 과제. 노트가 위치한 `Projects/논문_<Task>_tasks/` 폴더명과 동일한 값을 쓴다 (여러 개면 배열).
- `direction`: 이 논문이 연구 흐름에서 어떤 역할을 하는지 나타내는 **다중 선택 개방형** 태그. 한 논문이 여러 방향성에 동시에 해당될 수 있다 (예: 처음엔 새로운 시도였지만 지금은 foundational이기도 함). 아래 "direction 카테고리" 절 참고.
- `status`: 이 논문을 실제로 읽고 이해한 진행 상태를 나타낸다. Project Manager 프로젝트의 "Statuses" 설정(해야할 것/진행 중/추가 공부 요청/완료)과 값을 공유한다.
  - `to-do`: 아직 손대지 않음.
  - `in-progress`: 새로 처리 중 — Claude Code가 분석 노트를 막 생성했을 때의 기본값이 이 상태다. 사용자가 아직 논문을 다 읽지 않았어도 이 상태로 둔다.
  - `additional-study-needed`: 한 번 읽었지만 더 깊이 공부가 필요하다고 판단됨.
  - `done`: 사용자가 실제로 다 읽고 이해를 마침. Claude Code는 이 값을 스스로 `done`으로 바꾸지 않는다 — 사용자가 직접 다 읽고 나서 바꾸는 값이다.
- `start`: 이 논문을 `_inbox/`에 처음 넣은 날짜. `_inbox/`에 있던 PDF 파일의 수정 시각(mtime)을 확인해 채운다.
- `jcr_quartile`: 이 논문이 실린 저널/학회의 등급. **절대로 추측해서 채우지 않는다.** 규칙:
  - **학회 논문**(NeurIPS, CVPR, ICCV, ECCV, ICML, ICLR처럼 이 분야에서 명백히 top-tier로 통용되는 학회)은 `Q1`로 표기해도 된다 — 이건 업계에서 널리 인정되는 사실이라 확신 가능하다. 그 외 애매한 학회는 함부로 등급을 매기지 않는다.
  - **저널 논문**은 실제 공식 JCR 등급을 알아야 하는데, 이건 매년 갱신되고 같은 저널도 분류 카테고리(예: Engineering vs Computer Science)에 따라 등급이 다를 수 있어 Claude Code가 확신을 갖고 판단할 수 없다. **모르면 `null`로 비워두고, 사용자에게 물어본다** ("이 논문 — <venue> — JCR 등급 아시면 알려주세요" 형태로). 사용자가 답을 주면 그 값을 채운다.
    - 사용자가 이미 등급을 확인해준 저널은 아래 "사용자가 확정한 저널 등급" 목록에 있으니, 매번 다시 물어보지 않고 이 값을 그대로 쓴다.
  - **arXiv, 프리프린트, SSRN 등 미출판 저장소**는 JCR 등급 체계 대상이 아니므로 `jcr_quartile: arXiv`로 채운다(등급을 몰라서 비워두는 `null`과는 다른, "애초에 등급이 없는 게 확정된" 상태). venue 필드에도 저장소 이름을 명확히 적는다(`"arXiv"`, `"SSRN preprint"`).
  - **학회 워크숍**(CVPRW, ICCVW처럼 메인 학회의 부속 workshop track)도 JCR 등급 체계 대상이 아니므로 `jcr_quartile: Workshop`으로 채운다 — 메인 트랙(Q1)과 동일시하지 않는다. venue 필드에 `"CVPRW (CVPR Workshops)"`처럼 워크숍임을 명시해 메인 트랙 논문과 구분되게 한다.
  - `null`은 **오직** "저널 논문인데 등급을 몰라서 확인이 필요한" 상태에만 쓴다. arXiv/워크숍처럼 등급 체계 자체가 적용되지 않는 경우와 혼동하지 않는다 — `논문_PaperWiki.base`의 "JCR 등급 확인 필요" 뷰가 `jcr_quartile == null`로 걸러내므로, `arXiv`/`Workshop`으로 채워두면 그 뷰에서 자동으로 빠져 "더 확인할 필요 없음"으로 정확히 표시된다.
  - 새 논문을 처리할 때 저널 논문이면서 등급을 모르면, 노트 작성을 등급 때문에 멈추지 않는다 — 일단 `null`로 두고 노트는 정상 완성한 뒤, 처리 완료 보고 시점에 "JCR 등급 확인이 필요한 논문" 목록으로 따로 물어본다.

**사용자가 확정한 저널 등급** (매번 다시 묻지 않고 그대로 적용):
- IEEE Transactions on Geoscience and Remote Sensing (IEEE TGRS): `Q1`
- IEEE Journal of Selected Topics in Applied Earth Observations and Remote Sensing (JSTARS): `Q1`
- Neural Networks (Elsevier): `Q1`
- Image and Vision Computing (Elsevier): `Q1` (사용자는 "Q1-Q2 정도"로 표현 — 단일 값 필드 제약상 Q1로 단순화해 기록. 애매하면 재확인)
- ICASSP: `Q2` (사용자 본인 판단 기준, 공식 학회 등급 체계는 아님 — "본인이 그 정도로 생각한다"는 맥락으로 적용)

### 새 task 노트 생성 시 프로젝트 노트도 함께 갱신

- task 노트를 새로 만들면, 그 논문이 속한 분야의 `Projects/논문_<Task>.md`(해당 task 프로젝트 노트)의 `taskIds` 배열에 새 논문 노트의 `id`를 추가하고, 본문 "## Tasks" 아래에 `- [ ] [[<슬러그>|<논문 원제>]]` 줄을 추가한다. 노트의 `projectId`도 이 프로젝트 노트의 `id`와 일치해야 한다.
- 이번이 그 task로는 처음 들어오는 논문이라 `Projects/논문_<Task>.md` 프로젝트 노트 자체가 없으면, 새로 만든다(`pm-project: true`, 고유 `id`(예: `paperwiki-<task-slug>`), `title`은 task 폴더명을 사람이 읽기 편하게 쓴 이름, `taskIds`에 이번 논문의 `id` 하나만 넣고 시작). 동시에 `Projects/논문_<Task>_tasks/` 논문 노트 폴더도 만든다.
- 새 프로젝트 노트에는 다른 프로젝트들과 동일하게 `customFields`에 4개 필드를 정의한다: `Year`(number), `Venue`(text), `Summary`(text), `Tags`(multiselect, `options: []`로 시작). 각 필드 `id`는 다른 프로젝트와 절대 겹치지 않는 새 임의 문자열로 만든다(기존 프로젝트에서 쓰는 id를 재사용하지 않는다).
- Project Manager의 `select`/`multiselect` custom field는 이 워크플로우에서 쓰지 않는다(위 Frontmatter 절 참고, PaperWiki 속성은 전부 최상위 필드).

### 본문 구성 — 분석 노트 (`<Slug>.md`)

**독자 전제**: 이 노트는 object detection의 기본 개념(anchor, IoU, NMS, mAP 등)은 이미 아는 사람이, 그 논문이 제안하는 **모델 구조**(어떤 레이어가 무엇을 어떻게 하는지)를 처음부터 상세히 배우기 위한 자료다. 목표는 "이 노트만 보고 전체 흐름을 이해해서 다른 사람에게 설명할 수 있는 수준" — 단순 요약이 아니라 학습 자료로 취급한다.

frontmatter 바로 아래, `# 한 줄 요약` 헤딩보다도 위에 frontmatter의 `paper_tags` 배열을 `#해시태그` 형태로 한 줄 옮겨 적는다 (예: `#paper #object-detection #transformer`). frontmatter의 속성 패널은 hover 미리보기에서 항상 렌더링되는 게 아니라서, 본문 첫 줄에도 명시적으로 둬야 미리보기에서 바로 태그를 확인할 수 있다. `paper_tags` 배열이 바뀌면 이 줄도 함께 갱신한다. 그 아래 `> [!quote] 원제` 콜아웃으로 논문 원제·저자·venue를 적는다 — `title` 필드는 short title(없으면 원제)만 담으므로, 원제 전체는 본문에서 이 콜아웃으로 항상 볼 수 있게 한다.

```markdown
#paper #<세부 주제 태그...>

> [!quote] 원제
> **<논문 원제 전체>**
> <저자진> — <소속>, <venue> <year>

# 한 줄 요약
(노랑 하이라이트) 이 논문을 한 문장으로.

> [!info] 내 메모
> 

# 정리

| | 문제 ① | 문제 ② | ... |
|---|---|---|---|
| **문제 정의** | 이 문제가 왜 문제인지 | | |
| **풀고자 하는 문제** | 이 논문이 여기서 구체적으로 뭘 풀려는지 | | |
| **선행 연구 접근** | 갈래별 bullet + 갭 | | |
| **해결 방법** | 이 논문이 실제로 어떻게 풀었는지 1~2문장 | | |
| **예상되는 문제점** | 이 해결 방법 자체가 새로 만드는 구조적 문제 | | |

**갭 종합**: (노랑 하이라이트) 문제들이 공통으로 어떤 원인에서 나오는지, 이 논문의 통찰이 무엇인지 1~2문장.

> [!info] 내 메모
> 

# 제안 방법

(노랑 하이라이트) 핵심 아이디어를 1~2문장으로. 문장 안에서 이 논문이 처음 제시하는 핵심 장치는 <span style="color:#c0392b; font-weight:bold;">빨간 글씨</span>로 강조한다(HTML span에 마크다운 `**`를 중첩하지 않는다 — `font-weight:bold`를 span 스타일에 직접 넣는다. 중첩하면 Obsidian이 파싱하지 못해 `**` 기호가 그대로 텍스트로 보인다).

## 전체 파이프라인 (Fig. X 기준)

코드블록으로 단계 ①②③...을 화살표로 잇는 다이어그램을 그리고, 각 화살표 옆에 그 단계의 **출력 tensor shape**을 표기한다.

```
입력 (...)
       │
       ▼
① <단계 이름>              → (출력 shape)      [필요하면 보충 설명]
       │
       ▼
② <단계 이름>              → (출력 shape)
       │
       ▼
...
```

> [!info] 내 메모
> 

### ① <기법/모듈 이름>
- **역할**: 이 단계가 무엇을 위해 존재하는지. **논문이 처음 제시하는 모듈·약어는 반드시 역할부터 설명한 뒤에 그 이름/약어를 쓴다** — 약어부터 던지고 나중에 설명하지 않는다.
- **구현**: 어떤 레이어(1×1 conv, FC, self-attention 등)로 이루어져 있는지. 이미 [[Architecture Design]] 안에 있는 범용 구조(1×1 conv, self-attention 등)를 쓰면 `[[문서명]]`으로 링크만 하고 재설명하지 않는다(아래 "Architecture Design 연동" 절 참고). 이 논문에서 처음 제시하는 구조라면 여기서 직접 설명한다.
- **입출력 shape**: `(이전 shape)` → `(이 단계 출력 shape)`, 그 사이 어떤 연산으로 shape이 바뀌는지.

```python
# 의사코드 또는 논문/공식 구현 기반 코드. 실제 코드가 없으면 논문 수식을 그대로 코드로 옮긴 의사코드임을 주석으로 밝힌다.
```

(연한 노랑 하이라이트, 1~2문장) 왜 이게 "정리" 표에서 언급한 문제를 해결하는가.

> [!warning] 이 구조 때문에 예상되는 문제점
> 이 구조 자체가 낳는 부작용/비용/한계(있는 경우만). "정리" 표의 "예상되는 문제점" 행이나 뒤의 "실험 결과"·"Discussion"과 연결되면 서로 참조.

> [!info] 내 메모
> 

### ② <기법/모듈 이름>
(위와 동일한 구조 반복 — 기법이 여럿이면 ①②③...으로, "제안 방법" 파이프라인의 단계 번호와 일치시킨다)

## 파이프라인 정리표

| 단계 | 입력 shape | 출력 shape | 역할 | 구조/구현 |
|---|---|---|---|---|
| ① ... | | | | |

> [!info] 내 메모
> 

# 실험 결과

### 핵심 결과 — Table/Fig. X
**표/그림을 보는 법**: 이 표·그림을 어떻게 읽어야 하는지 한 문장. 표/그림 번호를 반드시 명시한다.

표 하나로 압축 (벤치마크 | 지표 | Before | After). 논문 전체에서 가장 중요한 1~2개 벤치마크·그림만 — 나머지는 아래 콜아웃으로.

> [!note]- 세부 결과 및 Ablation
> #### Table/Fig. X — <제목>
> **보는 법**: 한 문장.
> (나머지 벤치마크 표, ablation 표, 분석 그림을 전부 여기에 넣는다. 항목마다 번호+보는 법 한 문장을 반드시 포함)

> [!info] 내 메모
> 

# Discussion

각 항목은 표제 문장 1줄 + 근거 1~2줄로 제한한다. 길게 설명하고 싶으면 콜아웃으로 뺀다. Discussion 섹션 전체에 메모 콜아웃은 맨 끝에 **하나만** 둔다(소제목마다 넣지 않는다).

### 이 아이디어의 잠재적 부작용
- 부작용 후보 → (빨강 하이라이트) 논문이 해결했는지 여부, 한 줄.

### 한계
- (빨강 하이라이트) 저자가 명시한 한계, failure case. 항목당 1줄.

### 생각할 점
- (초록 하이라이트) 짧게.

### 내 주제와 연관된 후속 연구 아이디어
- (초록 하이라이트) 사용자의 다른 위키 문서와 연결, 짧게.

> [!info] 내 메모
> 

# 관련 개념
- [[concept-a]], [[concept-b]] — 이 논문이 사용하거나 확장하는 개념(위키 개념 문서, `ResearchVault/PaperStudy/Concepts/`)
- [[architecture-a]] — 이 논문에서 쓰인 범용 구조(`ResearchVault/Architecture Design/`, 아래 절 참고)

# 관련 문서
- 비교: [[comparison-xyz]] (있는 경우)

# 읽어볼 만한 논문
- 참고문헌 기반: <저자, "제목" [번호]> — 추천 이유
- 자유 추천(참고문헌에 없음, 검증 필요): <저자·제목 추정> — 추천 이유 / 검색 키워드: `...`
```

이 세 섹션(`# 관련 개념`, `# 관련 문서`, `# 읽어볼 만한 논문`)에는 메모 콜아웃을 넣지 않는다 — 사용자가 직접 채우는 목록이라 학습 메모를 붙일 지점이 아니다.

"제안 방법"과 "실험 결과"의 하위 heading(`###`/`####`)은 논문마다 실제 구조에 맞게 이름을 바꿔도 된다 — 위 템플릿은 뼈대이지, 섹션 제목을 토씨 하나까지 맞추라는 뜻이 아니다. 다만 상위 6개 헤딩(`#`, 한 줄 요약/정리/제안 방법/실험 결과/Discussion/관련 개념·관련 문서·읽어볼 만한 논문)과 그 배치 순서, 그리고 "왜 효과적인가"·"선행 연구 갭"·"파이프라인+tensor shape"·"Discussion 4항목" 같은 필수 요소는 반드시 포함한다.

"한계 및 열린 질문"이라는 별도 최상위 섹션은 만들지 않는다 — Discussion의 "한계"에 통합되어 있다.

### 메모 콜아웃 규칙

사용자가 논문을 공부하며 나중에 덧붙일 메모를 적을 자리를, `#`/`##`/`###` **모든 레벨의 제목 바로 뒤**에 하나씩 둔다(단, 위에서 예외로 둔 Discussion 소제목들과 관련 개념/관련 문서/읽어볼 만한 논문 3곳은 제외).

```markdown
> [!info] 내 메모
> 
```

- 접기(`-`)를 쓰지 않는다 — 항상 펼쳐진 상태로 둔다.
- 안내 문구("여기에 적어주세요" 같은)를 넣지 않는다 — 완전히 빈 줄로 둔다.
- Discussion 섹션은 예외로, 소제목 4개(부작용/한계/생각할 점/후속 연구 아이디어)를 전부 지나고 섹션 끝에 메모 콜아웃 1개만 둔다 — 소제목마다 넣지 않는다.
- `# 관련 개념`, `# 관련 문서`, `# 읽어볼 만한 논문`에는 메모 콜아웃을 아예 넣지 않는다.

### Architecture Design 연동 (`ResearchVault/Architecture Design/`)

"제안 방법"에서 1×1 conv, self-attention, bipartite matching처럼 **여러 논문에 반복해서 등장하는 범용 구조/알고리즘**을 설명할 때는, 그때마다 새로 설명하지 않고 `ResearchVault/Architecture Design/`에 별도 노트로 분리해 링크한다.

- **이미 있는 개념**이면: 본문에서 `[[구조명]]`으로 링크만 걸고, 그 구조의 역할·동작을 재설명하지 않는다(이 논문 맥락에서 왜/어떻게 쓰였는지는 분석 노트에 남긴다).
- **아직 없는 개념**이면: 재사용 여부와 무관하게(이 논문 1편에만 등장해도) 새로 만든다.
- 판단 기준은 Concepts/와 동일하다 — "구조적으로 독립 설명할 가치가 있는가". 이 논문 실험 세팅에만 국한된 사소한 디테일은 만들지 않는다.

파일명: `Architecture Design/<구조-슬러그>.md` (슬러그는 "폴더·파일 네이밍 규칙"의 일반 서술형 이름 규칙을 따른다, 예: `Multi_Head_Self_Attention`, `1x1_Convolution`)

#### Frontmatter

```yaml
---
title: "<구조/알고리즘 이름>"
tags: [architecture, <분류 태그...>]
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
---
```

#### 본문 구성

```markdown
# 역할
이 구조가 무엇을 위해 존재하는지 간결하게.

# 구조
## 입력/출력
입력·출력 shape을 명시.

## 내부 동작
단계별로 무엇을 계산하는지 — 필요하면 수식/의사코드를 `> [!example]-` 콜아웃에.

# 왜 이렇게 되는가
이 구조가 왜 이런 형태를 갖는지, 대안과 무엇이 다른지, 비용/트레이드오프.

# 등장 논문
- [[paper-a]] — 이 논문에서 어떤 역할로 쓰였는지 한 줄
- [[paper-b]] — ...
```

새 논문에서 이미 있는 Architecture Design 노트를 다시 인용하면, "등장 논문" 목록에 그 논문을 추가 갱신한다(Concepts/의 "등장 논문" 갱신 규칙과 동일).

### 하이라이트 규칙 (Highlightr)

Obsidian의 Highlightr 플러그인으로 의미 단위를 색으로 구분한다. `<mark style="background: <색상코드>;">텍스트</mark>` HTML을 문장/구절 단위로 감싼다.

| 색 | 코드 | 용도 |
|---|---|---|
| 노랑 | `#FFF3A3A6` | 핵심 요약 — "한 줄 요약" 섹션 전체, 각 섹션에서 가장 중요한 한 문장(제안 방법의 핵심 아이디어 등) |
| 연한 노랑 | `#FFF9D6A6` | 왜 효과적인가 — "제안 방법"에서 각 기법을 설명한 뒤, 그게 "문제 정의"에서 언급한 문제를 왜/어떻게 해결하는지 논리적으로 연결하는 문장. 핵심 요약(진한 노랑)과 구분하기 위해 옅은 톤을 쓴다 |
| 빨강 | `#FF5582A6` | 한계/주의 — "한계 및 열린 질문", Discussion의 "부작용"·"한계", failure case |
| 초록 | `#A6E3A1A6` | 후속 연구/생각할 점 — Discussion의 "생각할 점"·"후속 연구 아이디어" |

- 색은 문단 전체가 아니라, 그 문단에서 실제로 핵심인 절/문장에만 좁게 적용한다. 문단 전체를 색칠하면 강조 효과가 없어지고 가독성이 오히려 떨어진다.
- 한 문장 안에 여러 색을 섞지 않는다.
- 표, 코드블록 안에는 적용하지 않는다 — 표는 이미 시각적으로 구분되어 있어 하이라이트가 불필요하고, 코드블록 안 HTML은 렌더링되지 않고 그대로 텍스트로 보인다.
- 이 네 가지 용도 외에 임의로 색을 추가하지 않는다 — 색이 늘어날수록 "어떤 색이 무슨 뜻이었는지" 다시 찾아봐야 해서 오히려 가독성이 떨어진다.

### 콜아웃(접기) 규칙

Obsidian 콜아웃은 제목 뒤에 `-`를 붙이면 기본적으로 접힌 상태로 렌더링된다 (`> [!note]-`처럼). 분석 노트(`<Slug>.md`)에서 아래 두 지점에는 반드시 콜아웃을 써서 세부 내용을 기본적으로 접어둔다 — 파일을 열었을 때 스크롤 없이 핵심만 먼저 보이게 하기 위함이다.

- **제안 방법의 구현 디테일**: 수식, 의사코드, 하이퍼파라미터 값처럼 "필요할 때만 펼쳐보는" 내용은 `> [!example]- 구현 디테일` 콜아웃 안에 넣는다. 기법의 핵심 아이디어(bullet 3~5개)는 콜아웃 밖에 그대로 둔다.
- **실험 결과의 세부 ablation**: 핵심 벤치마크 표 1~2개만 콜아웃 밖에 두고, 나머지 벤치마크·ablation 표·세부 발견은 `> [!note]- 세부 결과 및 Ablation` 콜아웃 안에 몰아넣는다.

문법:
```markdown
> [!example]- 구현 디테일
> 첫 줄
> 두 번째 줄
>
> ```
> 코드블록도 콜아웃 안에 넣을 수 있다
> ```
```
콜아웃 안의 모든 줄은 `>`로 시작해야 한다는 점에 유의한다 (코드블록·표 포함).

### 줄글 최소화 규칙

분석 노트 전체에서 3줄을 넘는 줄글 문단을 쓰지 않는다. 설명이 길어질 것 같으면:
1. 먼저 bullet로 쪼갤 수 있는지 본다 (대부분 가능하다 — "A이고, B이며, 그 결과 C다"는 bullet 3개로 쪼개진다).
2. 표로 옮길 수 있는 데이터(수치, Before/After, 비교)면 표로 옮긴다.
3. 위 둘 다 아니고 정말 풀어서 설명해야 하는 내용이면(예: 복잡한 인과관계), 콜아웃 안에 넣어 기본적으로는 접혀 있게 한다.
"줄글로 3줄 이상 쓰고 있다"는 것 자체가 위 세 가지 중 하나를 안 했다는 신호로 간주한다.

### bullet 줄바꿈 규칙

`**항목명**: 긴 설명...` 형태로 한 bullet 안에 라벨과 설명을 한 줄로 이어 쓰지 않는다. Obsidian에서 이런 bullet은 설명이 길어지면 두 번째 줄부터 라벨 너비만큼 어정쩡하게 밀려 들여써져서 읽기 불편하다. 대신 항목명 뒤에서 줄을 바꾸고, 설명은 다음 줄에 들여쓰기 없이 쓴다.

```markdown
- **항목명**:
  설명. 여러 문장이어도 이 한 줄(또는 자연스러운 개행)로 이어 쓴다.
- **다음 항목명**:
  설명.
```

이 규칙은 문제 정의의 "기존 방법의 한계", "선행 연구는 어떻게 접근했고 어떤 갭이 남았는가"처럼 **"라벨 + 긴 설명"으로 구성된 bullet 목록 전체**에 적용한다. 라벨 없이 짧은 한 줄로 끝나는 bullet(예: "이 논문이 풀고자 하는 문제"의 번호 목록)에는 굳이 적용하지 않아도 된다.

### "제안 방법" 작성 시 필수 규칙

각 기법/모듈을 설명할 때는 What(무엇을 하는지)만으로 끝내지 않는다. 반드시 이어서 **"왜 이게 '문제 정의'에서 언급한 문제를 해결하는가"**를 한두 문장으로 명시하고, 그 문장에 연한 노랑 하이라이트를 적용한다. 단순히 메커니즘만 나열하고 그게 왜 효과적인지 논리적 연결이 빠진 설명은 이 규칙 위반이다.

### "읽어볼 만한 논문" 작성 규칙

두 종류를 구분해서 적는다.

- **참고문헌 기반**: 이 논문이 실제로 인용한 문헌 중, 사용자의 연구 방향과 특히 관련 있어 보이는 것을 추천한다. 저자·제목·연도·참고문헌 번호를 원문 그대로 정확히 옮긴다 — 이 항목은 원문에 있는 사실이므로 허구 위험이 없다.
- **자유 추천**: 참고문헌에 없어도 관련 있다고 판단되는 논문을 추천할 수 있다. 다만 이건 모델의 사전지식에 의존하므로 제목·저자·정확한 존재 여부가 틀릴 수 있다 — 반드시 "(검증 필요)"라고 표시하고, 사용자가 직접 검색해 확인할 수 있도록 구체적인 검색 키워드를 함께 제공한다. 논문 제목을 확신 없이 지어내지 않는다.
- 추천마다 왜 읽어볼 만한지 한 줄 이유를 붙인다 (예: "이 논문의 X 기법이 확장한 원조 방법이라 배경 이해에 도움", "같은 문제를 다른 도메인에서 다룸").
- 이 섹션에 추천을 추가했으면, 같은 항목을 `reading-list.md`의 해당 task 섹션에도 추가한다 (아래 "reading-list.md" 절 참고).

## reading-list.md — 읽을 논문 모음

`reading-list.md`는 여러 논문 노트에 흩어진 "읽어볼 만한 논문" 추천을 task별로 한곳에 모은 단일 파일이다. "다음에 뭘 읽을지" 고민할 때 이 파일 하나만 보면 되도록 하는 것이 목적이다.

### 구성

```markdown
# 읽을 논문 모음

## <task-1>
- [ ] <저자, "제목"> [번호 또는 없음] — 추천 이유 (출처: [[추천한-논문-슬러그]])
- [ ] <저자, "제목" (검증 필요)> — 추천 이유 / 검색 키워드: `...` (출처: [[추천한-논문-슬러그]])

## <task-2>
- [ ] ...
```

### 갱신 규칙

- 새 논문 노트에 "읽어볼 만한 논문" 항목을 추가할 때마다, 해당 task 섹션에 동일한 항목을 체크박스(`- [ ]`)로 추가하고 어느 논문 노트에서 추천했는지(`출처: [[...]]`) 남긴다.
- 이미 `reading-list.md`에 있는 논문을 나중에 실제로 `_inbox/`에 넣어 처리하면(즉 `Projects/논문_<Task>_tasks/`에 그 논문의 정식 task 노트가 새로 생기면), `reading-list.md`에서 해당 항목을 **삭제**한다 (체크만 하고 남겨두지 않는다 — 목록이 "아직 안 읽은 것"만 남도록 유지한다).
- task 섹션이 없으면 새로 만든다.

## direction 카테고리

논문이 연구 흐름에서 차지하는 역할을 나타낸다. **다중 선택**이다 — 하나의 논문이 여러 역할을 동시에 가질 수 있어서 단일 선택으로 강제하면 억지로 하나만 골라야 하는 문제가 생긴다.

현재 카테고리 (폐쇄 목록 아님 — 아래에 안 맞는 논문을 만나면 새 카테고리를 추가하고 이 목록도 갱신한다):

- `foundational` — Attention Is All You Need, ResNet, Mamba, YOLO처럼 이후 연구 흐름 전체의 기반이 된 원조/핵심 논문
- `improvement` — 기존 모델/기법을 개선하거나 변형한 논문
- `novel-approach` — 기존과 다른 새로운 방법론을 처음 시도하는 논문
- `survey` — 특정 분야를 정리·비교하는 서베이/리뷰 논문

새 카테고리가 필요하다고 판단되면(위 4개 중 어느 것도 이 논문의 역할을 제대로 설명하지 못할 때) 만들어서 쓰고, 이 목록에도 한 줄로 추가해 갱신한다.

## Concepts/ — 개념/기법 문서

**독립적으로 설명할 가치가 있는 개념/기법**이면 만든다. "이미 다른 논문에서도 쓰였는가"는 기준이 아니다 — 재사용 여부는 논문을 한 편만 봐서는 판단할 수 없는 미래 정보이기 때문에, 매번 판단 가능한 기준으로 대신한다.

판단 기준: 이 논문의 핵심 기여가 특정 기법/아이디어를 제안하거나 확장하는 것이라면, 그 기법은 그 자리에서 바로 concept 문서로 만든다 (지금 이 논문 1편에만 등장하더라도 상관없다). 반대로 논문 안에서만 의미가 있는 부수적 디테일(하이퍼파라미터 선택, 사소한 ablation 변형, 이 논문의 실험 세팅에 국한된 트릭)은 concept으로 만들지 않고 그 논문 노트 안에만 적는다.

새 논문을 처리하다가 이미 존재하는 개념을 다시 언급하면, 새 concept 문서를 만들지 말고 기존 문서의 "등장 논문" 목록에 추가 갱신한다.

파일명: `Concepts/<개념-슬러그>.md` (슬러그는 위 "폴더·파일 네이밍 규칙"의 일반 서술형 이름 규칙을 따른다, 예: `Latent_Reconstruction_Error`)

### Frontmatter

```yaml
---
title: "<개념/기법 이름>"
tags: [concept, <분류 태그...>]
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
---
```

### 본문 구성

```markdown
# 정의
이 개념/기법이 무엇인지 간결하게.

# 등장 논문
- [[paper-a]] — 어떤 역할로 사용했는지 한 줄
- [[paper-b]] — 어떤 역할로 사용했는지 한 줄

# 변형/발전
시간 순으로 이 개념이 어떻게 변형·확장되었는지 (등장 논문이 늘어날 때마다 갱신)

# 관련 개념
- [[other-concept]]
```

## Comparisons/ — 비교 문서

논문 2편 이상을 공통 축(예: 같은 문제를 푸는 다른 접근, 같은 벤치마크 성능, 시계열적 후속 관계)으로 비교할 가치가 있을 때 만든다. 모든 새 논문마다 만들 필요는 없다 — 명확히 비교되는 대상이 있을 때만.

파일명: `Comparisons/<비교-주제-슬러그>.md` (슬러그는 위 "폴더·파일 네이밍 규칙"의 일반 서술형 이름 규칙을 따른다, 예: `Small_Object_Detection_Approaches`)

### Frontmatter

```yaml
---
title: "<비교 주제>"
tags: [comparison]
papers: [[[paper-a]], [[paper-b]], ...]
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
---
```

### 본문 구성

```markdown
# 비교 축
무엇을 기준으로 비교하는지.

# 비교표
| 논문 | 핵심 방법 | 성능/지표 | 비고 |
|---|---|---|---|
| [[paper-a]] | | | |
| [[paper-b]] | | | |

# 분석
공통점, 차이점, 어느 상황에 어떤 논문의 접근이 유리한지.
```

## Moc/ — Map of Content

`Projects/논문_<Task>_tasks/`의 task 노트들이나 `논문_PaperWiki.base`가 "무엇이 있는지"를 테이블로 보여준다면, MOC는 **그 사이의 맥락과 서사**를 담는 곳이다. 단순히 논문 목록을 나열하지 않는다 — 그건 Bases 테이블이 이미 더 잘한다.

MOC는 두 계층으로 구성한다.

### Task MOC — `Moc/<Task>_Moc.md`

`Projects/논문_<Task>_tasks/`, `/Users/GyeongSeo/Workspace/논문_pdf/`에 존재하는 task 폴더마다 하나씩 만든다 (예: `Moc/Small_Object_Detection_Moc.md`).

파일명: `Moc/<Task>_Moc.md`

#### Frontmatter

```yaml
---
title: "<task> MOC"
tags: [moc]
task: <task>
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
---
```

#### 본문 구성

```markdown
# 이 분야가 다루는 핵심 질문
이 task가 풀려는 문제를 1~3개 질문으로. (예: "작은 객체의 약한 feature를 어떻게 보강할 것인가?")

# 지금까지 다룬 흐름
이 task에 속한 논문들을 시간순 또는 접근법 계열별로 서술 (단순 나열이 아니라 서로 어떤 관계인지 — 같은 문제를 다른 각도로 풀었는지, 후속작인지, 반박인지).
- [[paper-a]] — 이 흐름에서 어떤 위치인지 한 줄
- [[paper-b]] — ...

# 이 분야를 관통하는 개념
- [[concept-x]] — 왜 이 task에서 중요한지
- [[concept-y]] — ...

# 비교 문서
- [[comparison-xyz]]

# 아직 못 채운 빈틈
이 task에서 읽어야 할 것 같은데 아직 없는 논문/각도, 풀리지 않은 질문.

# 관련 MOC
- [[000-Home]]
- [[다른-Task_Moc]] (겹치거나 인접한 분야가 있으면)
```

### 홈 MOC — `Moc/000-Home.md`

전체 위키의 진입점. 한 번만 만들고 이후 갱신한다.

```markdown
# 연구 지도

## Task별 MOC
- [[Small_Object_Detection_Moc]]
- [[Anomaly_Detection_Moc]]
- ...

## 지금 무엇에 집중하고 있는가
현재 관심사, 최근에 읽은 흐름.
```

### 갱신 규칙
- 새 논문이 `Projects/논문_<Task>_tasks/`에 추가될 때마다, 해당 `<Task>_Moc.md`가 없으면 새로 만들고, 있으면 "지금까지 다룬 흐름"·"이 분야를 관통하는 개념" 섹션을 갱신한다 (아래 워크플로우 8번 참고).
- 이전에 없던 task 폴더가 새로 생기면 `000-Home.md`의 "Task별 MOC" 목록에도 추가한다.
- 단순 논문 나열이 되지 않도록 주의한다 — "지금까지 다룬 흐름"은 항상 논문 간 관계·서사를 한 줄이라도 포함해야 한다. 관계를 못 쓰겠으면(정말 처음 추가되는 논문이라) 다음 논문이 들어올 때 채운다.

## 새 논문 처리 워크플로우

`/Users/GyeongSeo/Workspace/논문_pdf/_inbox/`에 새 PDF가 추가되고 "_inbox/에 새로 추가한 논문 읽고 반영해줘" 요청을 받으면:

1. `/Users/GyeongSeo/Workspace/논문_pdf/_inbox/`에 있는 각 PDF를 읽는다. (`/Users/GyeongSeo/Workspace/논문_pdf/_issue_paper/`는 이 워크플로우의 대상이 아니다 — 커뮤니티 트렌드 추적용 별도 보관 폴더이므로 스캔하지 않는다.)
2. PDF 내용을 바탕으로 이 논문의 task를 판단한다 (기존 `/Users/GyeongSeo/Workspace/논문_pdf/<Task>/` 폴더 중 맞는 게 있으면 그걸 쓰고, 없으면 새 task 폴더명을 정한다).
3. "PDF 파일명 규칙"대로 `{년도}_{venue}_{제목}.pdf`로 리네임하고 `/Users/GyeongSeo/Workspace/논문_pdf/<Task>/`로 이동한다 (`_inbox/`에는 남기지 않는다).
4. `Projects/논문_<Task>_tasks/<Slug>.md`(분석 노트, task 노트)를 만든다. `title`은 논문 원제만 채운다(접두어 없음). `source`에 이동 후 PDF 경로(`/Users/GyeongSeo/Workspace/논문_pdf/<Task>/<리네임된 파일명>`)를 채우고, `task`, `direction` 속성을 채운다. 해당 `Projects/논문_<Task>.md`의 `customFields:` 정의에서 Year/Venue 필드 id를 확인해, task 노트의 `customFields`에 같은 값을 채운다(위 Frontmatter 절 참고). `status`는 항상 `in-progress`로 시작한다(사용자가 직접 다 읽고 나서 `done`으로 바꾸는 값이므로, 새로 처리했다고 `done`으로 채우지 않는다). `start`(PDF의 `_inbox/` 진입 날짜, mtime 기준)도 함께 채운다. `jcr_quartile`은 위 Frontmatter 절의 규칙대로 채운다 — 학회가 명백한 top-tier면 `Q1`, 저널이거나 등급을 모르면 `null`로 두고 나중에 사용자에게 물어볼 목록에 추가한다(추측 금지). 분석 노트 템플릿(콜아웃·bullet 규칙 포함)을 따른다. 마지막으로 해당 `Projects/논문_<Task>.md` 프로젝트 노트의 `taskIds`와 "## Tasks" 목록에 이 논문을 추가한다(프로젝트 노트가 아직 없으면 위 "새 task 노트 생성 시 프로젝트 노트도 함께 갱신" 절 규칙대로 새로 만든다). `projectId`는 이 프로젝트 노트의 `id`와 일치시킨다.
5. `grep -r "#pending:<이번 논문의 슬러그>" .`(`ResearchVault/PaperStudy/`와 `Projects/` 양쪽에서 실행)를 실행해서, 기존 노트 중 이 논문을 미완성 링크(`#pending:` 마커)로 남겨둔 곳이 있는지 확인한다. 있으면 해당 문장을 실제 `[[wiki-link]]`로 갱신하고 마커를 지운다.
6. 논문에서 "독립적으로 설명할 가치가 있는" 개념/기법(핵심 기여인 기법·아이디어)이 있는지 판단한다 — 재사용 여부와 무관하게, 이 논문 1편만 보고 판단한다.
   - 그런 개념이 있고 `Concepts/`에 이미 있으면: 해당 concept 문서의 "등장 논문"에 이번 논문을 추가하고, 필요하면 "변형/발전" 섹션을 갱신한다.
   - 그런 개념이 있고 아직 없으면: 지금 이 논문 1편에만 등장하더라도 새 concept 문서를 만든다.
   - 논문 안에서만 의미 있는 부수적 디테일이면 concept으로 만들지 않는다.
6-1. "제안 방법"에서 쓰인 범용 구조/알고리즘(1×1 conv, self-attention 등)이 `ResearchVault/Architecture Design/`에 이미 있는지 확인한다 — 있으면 링크만 걸고, 없으면 새로 만든다("Architecture Design 연동" 절 기준).
7. 이번 논문이 기존 논문과 비교할 가치가 있으면(같은 문제, 같은 벤치마크, 직접적 후속작 등) `Comparisons/`에 문서를 만들거나 기존 비교 문서를 갱신한다.
8. comparison만 애매하면 만들지 않는다 — 나중에 다른 논문이 들어와 근거가 쌓이면 그때 만든다. (concept은 위 6번 기준으로 그때그때 판단한다.)
9. `Moc/<Task>_Moc.md`를 갱신한다 — 없으면 새로 만들고, 있으면 "지금까지 다룬 흐름"에 이번 논문을 추가하고 다른 논문과의 관계를 서술한다. 새로 만든 concept/comparison이 있으면 해당 섹션에도 링크를 추가한다. task 폴더가 이번에 처음 생겼다면 `Moc/000-Home.md`의 "Task별 MOC" 목록에도 추가하고, `Projects/논문_PaperWiki.base`에도 `<Task> (year)`/`<Task> (venue)` 필터 뷰 한 쌍을 추가한다(위 "상태 확인하기" 절 참고).
10. 이번 논문이 다루는 내용 중, 아직 위키에 없는 다른 논문을 언급하며 나중에 링크할 대상으로 남겨야 하는 경우(예: 비교 대상으로 인용되지만 PDF가 없는 논문) `#pending:<그-논문-슬러그>` 마커를 붙여 남긴다.
11. 논문 노트의 "읽어볼 만한 논문" 섹션을 채운다("읽어볼 만한 논문" 작성 규칙 참고). 여기 추가한 항목은 `reading-list.md`의 해당 task 섹션에도 동일하게 추가한다.
12. 이번에 처리한 논문이 `reading-list.md`에 이미 있던 항목이라면(즉 예전에 추천되어 대기 중이던 논문을 지금 실제로 읽은 것이라면), 그 항목을 `reading-list.md`에서 삭제한다.
13. 이번에 처리한 논문 중 `jcr_quartile`이 `null`로 남은 저널 논문(등급 확인 필요)이 있으면, 처리 완료 보고의 마지막에 "JCR 등급을 확인해 주세요"라는 목록으로 venue와 함께 사용자에게 물어본다. 사용자가 답을 주면 해당 논문 노트의 `jcr_quartile`을 갱신한다.
14. 노트 작성을 마치면 원본 PDF와 다시 대조해 검수한다 — "정리" 표·파이프라인·tensor shape·수치가 원문과 정확히 일치하는지, 중요한 내용이 빠지거나 잘못 설명된 부분은 없는지 확인한다. 발견한 오류는 그 자리에서 바로 고친다(사용자에게 "검수했다"고만 보고하지 않고, 실제로 고친 뒤 보고한다).

## 상태 확인하기 (Bases · Project Manager)

Obsidian의 **Bases** 플러그인을 쓰면 Notion 데이터베이스 뷰처럼 모든 `Projects/논문_<Task>_tasks/`(task 프로젝트마다 있는 논문 노트 폴더 전체)를 가로질러 테이블로 보면서 `status`, `task`, `direction`, `jcr_quartile` 컬럼으로 필터·정렬할 수 있다. `Projects/논문_PaperWiki.base` 파일이 `task != null` 필터(PaperWiki 논문 노트에만 있는 고유 필드라 다른 프로젝트 노트와 섞이지 않는다)로 이 뷰들을 미리 정의해 둔 것이므로, Obsidian에서 그 파일을 열면 바로 확인 가능하다. "해야할 것/진행 중/추가 공부 요청/완료" 뷰로 진행 상태를, "Q1만"/"JCR 등급 확인 필요" 뷰로 저널 등급을 걸러볼 수 있다.

분야(task)별로 집중해서 보고 싶을 땐 "Task별" 뷰(전체를 `task` 기준 정렬만 함)보다, task마다 있는 `<Task폴더명> (year)` / `<Task폴더명> (venue)` 뷰 쌍(예: "Small_Object_Detection (year)", "Small_Object_Detection (venue)")을 쓴다 — 이 뷰들은 그 분야의 논문만 걸러서 보여주고, 이름 그대로 `year` 또는 `venue` 기준으로 정렬(`sort`)되어 있으므로 화면에 다른 분야가 섞이지 않는다. 새 task 폴더가 생기면 `논문_PaperWiki.base`에 `task.contains("<task-slug>")` 필터를 쓰는 `(year)`/`(venue)` 뷰 한 쌍을 추가한다(`task-slug`는 그 task 노트들의 `task` frontmatter 값, kebab-case).

**Project Manager 플러그인 자체 UI**(Table/Kanban/Gantt)로 분야별로 나눠 보려면, task마다 있는 프로젝트 노트(`Projects/논문_<Task>.md`, 예: `논문_Small_Object_Detection.md`, `논문_Anomaly_Detection.md`)를 열면 된다 — Project Manager UI의 프로젝트 목록/드롭다운에서 원하는 분야의 프로젝트를 선택하면 그 분야 논문만 나열되고, task를 클릭하면 분석 노트 본문이 그대로 렌더링되며 `status`를 드래그 앤 드롭으로 바꿀 수 있다. (예전에는 모든 논문이 `논문 읽기.md`라는 단일 프로젝트 안에 있어서 분야 구분 없이 다 섞여 보였는데, 그 문제 때문에 프로젝트 자체를 task별로 나눴다 — 위 "프로젝트 구조" 절 참고.) **title은 이 UI에서 편집하지 않는다** (위 경고 참고).

폴더 자체로도 확인 가능하다 — `/Users/GyeongSeo/Workspace/논문_pdf/_inbox/`에 파일이 남아있으면 아직 위키에 반영 안 된 논문, `/Users/GyeongSeo/Workspace/논문_pdf/<Task>/`에 있으면 처리 완료.
