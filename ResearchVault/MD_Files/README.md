# PaperWiki

Claude Code가 논문 PDF를 읽고 자동으로 정리·유지해주는 개인 논문 위키.

## 구조

위키 부속 문서(개념/비교/MOC)는 `ResearchVault/PaperStudy/`에, 논문 분석 노트는 Project Manager 플러그인과 통합 관리하기 위해 볼트 최상위 `Projects/`에 있다. 논문 **PDF 원본**은 용량 문제로 Obsidian vault(`GingseoLife/`) 밖 `/Users/GyeongSeo/Workspace/논문_pdf/`에 별도 보관한다.

```
ResearchVault/PaperStudy/
  Concepts/              개념/기법 노트 (task로 나누지 않고 한 곳에 모음)
  Comparisons/            논문 간 비교 문서 (task로 나누지 않고 한 곳에 모음)
  Moc/                    Map of Content — task별 서사/맥락 허브 + 전체 홈
  reading-list.md          task별로 정리된 "읽어볼 만한 논문" 모음
ResearchVault/MD_Files/
  Schema.md                모든 규칙 (Claude Code가 따르는 규칙)
  README.md                이 문서

Projects/
  논문_<Task>.md              task별 Project Manager 프로젝트 노트 (예: 논문_Small_Object_Detection.md). task 개수만큼 존재
  논문_<Task>_tasks/          그 task의 논문 분석 노트들. 논문 1편 = 파일 1개. 폴더명은 반드시 `<프로젝트파일명>_tasks` 형식(Project Manager 하드코딩 규칙)
  논문_PaperWiki.base         모든 논문_<Task>_tasks/를 가로질러 보는 Obsidian Bases 뷰 정의

/Users/GyeongSeo/Workspace/논문_pdf/     vault 밖. 논문 PDF 원본
  _inbox/                   아직 처리 안 한 원본 PDF (다운받으면 여기에 넣는다)
  <Task>/                    처리 완료 후 이동되는 task별 PDF 폴더
  _issue_paper/              (참고용, 처리 대상 아님) 커뮤니티에서 이슈가 된 논문을 트렌드 파악용으로 모아두는 폴더
```

폴더·파일명 대소문자·구분자 규칙(단어별 대문자 시작 + `_` 구분, 논문 슬러그의 약어 대문자 규칙 등)은 `Schema.md`의 "폴더·파일 네이밍 규칙" 절을 따른다.

## 사용법

1. 새로 읽고 싶은 논문의 PDF를 `/Users/GyeongSeo/Workspace/논문_pdf/_inbox/`에 넣는다.
2. 이 폴더(PaperWiki)에서 Claude Code를 열고 `/process-papers`라고만 입력한다 (긴 프롬프트를 매번 안 써도 됨 — `.claude/skills/process-papers/`에 등록된 스킬). 또는 직접 "_inbox/에 새로 추가한 논문 읽고 Schema.md 규칙대로 반영해줘"라고 요청해도 동일하게 동작한다.
3. Claude Code가 알아서:
   - PDF를 `{년도}_{venue}_{제목}.pdf`로 리네임하고, task를 판단해 `/Users/GyeongSeo/Workspace/논문_pdf/<Task>/`로 옮기고
   - `Projects/논문_<Task>_tasks/`에 논문 분석 노트(=task 노트)를 생성한다 (`task`, `direction`, `status: in-progress`, `venue`, `year`, `jcr_quartile` 등 속성 포함). `title`은 논문 원제만 채우고, 연도·venue는 프로젝트별 Year/Venue customField에 별도로 채운다(약어 매핑은 `Schema.md`의 Frontmatter 절 참고)
   - 해당 `Projects/논문_<Task>.md` 프로젝트 노트의 `taskIds`와 "## Tasks" 목록에 이 논문을 추가하고(그 task로 처음 들어오는 논문이면 프로젝트 노트와 `_tasks` 폴더를 새로 만든다)
   - 관련된 `Concepts/` 문서를 찾아 갱신하거나 새로 만들고
   - 필요하면 `Comparisons/`에 비교 문서를 채우고
   - `reading-list.md`와 해당 `Moc/<Task>_Moc.md`도 갱신한다.
4. Obsidian으로 해당 분야의 `Projects/논문_<Task>.md`를 열어 Project Manager의 Table/Kanban/Gantt 뷰로 진행 상황을 관리하거나(분야마다 프로젝트가 따로 있어 UI에서 프로젝트를 전환하면 그 분야만 보인다), `Projects/논문_PaperWiki.base`로 전체를 가로질러 필터링한다.

완전 자동(정해진 시간마다 알아서 실행)은 지원하지 않는다 — Claude Code의 로컬 파일 접근과 "매일 정해진 시간에 알아서 실행"이 동시에 되는 방법이 없어서(클라우드 스케줄은 GitHub 등 원격 저장소 동기화가 필요), 매번 `/process-papers`로 수동 요청하는 방식을 택했다.

> [!warning] Project Manager UI에서 논문 task의 title을 편집하지 않는다
> Task 모달에서 title을 저장하면 파일명이 title 기반 slug로 자동 리네임되고(끄는 설정 없음), PaperWiki 고유 속성(venue/year/jcr_quartile 등)이 통째로 사라지는 사고가 실제로 있었다. title을 고칠 땐 Obsidian 편집기로 파일을 직접 열어 frontmatter만 수정한다. 자세한 내용은 `Schema.md`의 관련 경고 참고.

## 처리 여부 확인하기

**Claude Code가 노트를 만들었는지**와 **내가 실제로 읽었는지**는 다른 축이지만, 후자만 `status` 속성 하나로 관리한다.

- **Claude Code 처리 여부**: 폴더로 확인한다 — `/Users/GyeongSeo/Workspace/논문_pdf/_inbox/`에 파일이 남아있으면 아직 미처리, `/Users/GyeongSeo/Workspace/논문_pdf/<Task>/`로 옮겨졌으면 노트가 만들어진 것.
- **내가 실제로 읽었는지**: 논문 분석 노트(task 노트)의 `status` 속성으로 관리한다. Project Manager 프로젝트의 "Statuses" 설정과 값을 공유한다 — `to-do`(아직 안 봄) / `in-progress`(Claude Code가 노트를 만든 직후 기본값, 아직 다 안 읽음) / `additional-study-needed`(한 번 읽었지만 더 깊이 볼 필요 있음) / `done`(다 읽고 이해함). Claude Code는 새 노트를 만들 때 항상 `in-progress`로 시작하고, 이 값을 스스로 `done`으로 바꾸지 않는다 — 직접 다 읽고 나서 frontmatter에서 바꾸거나, Project Manager의 Kanban 보드에서 드래그해서 바꾼다. `start` 속성은 그 논문 PDF를 `_inbox/`에 처음 넣은 날짜.

Obsidian에서 `Projects/논문_PaperWiki.base` 파일을 열면 Notion 데이터베이스 뷰처럼, 모든 `Projects/논문_<Task>_tasks/` 폴더를 가로질러 테이블로 필터링할 수 있다 (전체 / 해야할 것 / 진행 중 / 추가 공부 요청 / 완료 / Task별 / Q1만 / JCR 등급 확인 필요). Obsidian 버전이 오래되어 Bases가 없다면 설정 > 코어 플러그인에서 "Bases"를 켠다. Project Manager 플러그인 자체의 Kanban/Gantt 뷰로 분야별로 나눠 보려면, 그 분야의 `Projects/논문_<Task>.md` 프로젝트 노트를 열면 된다 — 분야마다 프로젝트가 따로 있으므로 UI에서 프로젝트를 전환하면 그 분야 논문만 보인다.

**특정 분야에만 집중하고 싶을 때**: "Task별" 뷰는 전체를 정렬만 해서 보여주므로 다른 분야가 한 화면에 섞인다. 대신 task마다 `<Task폴더명> (year)` / `<Task폴더명> (venue)` 뷰 쌍(예: "Small_Object_Detection (year)", "Small_Object_Detection (venue)")이 있다 — 그 분야 논문만 걸러서 보여주고, 연도순으로 보고 싶으면 (year) 뷰를, 학회/저널별로 묶어보고 싶으면 (venue) 뷰를 Bases 탭에서 클릭하면 된다.

## 다음에 읽을 논문 찾기

`reading-list.md`를 열면 각 논문 노트가 추천한 "읽어볼 만한 논문"이 task별로 한곳에 모여있다. 참고문헌 기반 추천(원문 그대로, 신뢰도 높음)과 자유 추천("(검증 필요)" 표시 + 검색 키워드 포함, 검증 필요)이 구분되어 있다. 이 목록에 있던 논문을 실제로 읽어서 위키에 넣으면 자동으로 목록에서 빠진다.

## direction 속성이란

논문이 연구 흐름에서 하는 역할을 나타내는 다중 선택 태그. 예를 들어 Attention Is All You Need, ResNet, Mamba, YOLO 같은 논문은 `foundational`로 묶이고, 기존 모델을 개선한 논문은 `improvement`, 새로운 방법론을 처음 시도하는 논문은 `novel-approach`로 묶인다. 한 논문이 여러 방향성에 동시에 해당될 수 있어 다중 선택이며, 카테고리 자체도 폐쇄 목록이 아니라 필요하면 늘어난다 (자세한 규칙은 `Schema.md`의 "direction 카테고리" 절 참고).

## 규칙을 바꾸고 싶다면

`Schema.md`를 직접 수정하면 된다. 이후 요청부터 바뀐 규칙이 적용된다.
