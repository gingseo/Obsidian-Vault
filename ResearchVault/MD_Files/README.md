# PaperWiki

Claude Code가 논문 PDF를 읽고 자동으로 정리·유지해주는 개인 논문 위키.

## 구조

위키 부속 문서(개념/비교/MOC)와 논문 분석 노트, 논문 **PDF 원본**은 모두 `ResearchVault/Papers/`(또는 그 옆 `ResearchVault/PaperStudy/`)에 있다. 볼트 최상위 `Projects/`는 실제 진행 중인 프로젝트(예: `MOLENet.md`) 전용이고 논문 모음은 두지 않는다.

```
ResearchVault/PaperStudy/
  Concepts/              개념/기법 노트 (task로 나누지 않고 한 곳에 모음)
  Comparisons/            논문 간 비교 문서 (task로 나누지 않고 한 곳에 모음)
  Moc/                    Map of Content — task별 서사/맥락 허브 + 전체 홈
  reading-list.md          task별로 정리된 "읽어볼 만한 논문" 모음
ResearchVault/MD_Files/
  Schema.md                모든 규칙 (Claude Code가 따르는 규칙)
  README.md                이 문서

ResearchVault/Papers/
  <Task>_overview.md          task별 개요 노트 (예: Small_Object_Detection_overview.md). task 개수만큼 존재하고, 그 task 논문 목록을 담는다
  <Task>_notes/                그 task의 논문 분석 노트들. 논문 1편 = 파일 1개
  PaperWiki.base               모든 <Task>_notes/를 가로질러 보는 Obsidian Bases 뷰 정의
  _pdf/     논문 PDF 원본 (git 추적은 .gitignore로 제외)
    _inbox/                   아직 처리 안 한 원본 PDF, 직접 읽고 싶어서 넣은 것 (다운받으면 여기에 넣는다) → 처리되면 노트에 source_type: personal
    _issue_paper/              아직 처리 안 한 원본 PDF, 커뮤니티에서 이슈가 된 논문을 트렌드 파악용으로 넣은 것 → 처리되면 노트에 source_type: community
    <Task>/                    처리 완료 후 이동되는 task별 PDF 폴더 (출처 폴더 구분 없이 한 곳에 모인다)
```

폴더·파일명 대소문자·구분자 규칙(단어별 대문자 시작 + `_` 구분, 논문 슬러그의 약어 대문자 규칙 등)은 `Schema.md`의 "폴더·파일 네이밍 규칙" 절을 따른다.

## 사용법

1. 새로 읽고 싶은 논문의 PDF를 넣는다 — 직접 읽으려는 논문이면 `ResearchVault/Papers/_pdf/_inbox/`에, 커뮤니티에서 화제라 트렌드 파악 차원에서 챙겨두는 논문이면 `ResearchVault/Papers/_pdf/_issue_paper/`에 넣는다. 둘 다 처리 대상이며, 어느 폴더에 넣었는지에 따라 노트의 `source_type`(`personal`/`community`)만 달라진다.
2. 이 폴더(PaperWiki)에서 Claude Code를 열고 `/process-papers`라고만 입력한다 (긴 프롬프트를 매번 안 써도 됨 — `.claude/skills/process-papers/`에 등록된 스킬). 또는 직접 "_inbox/에(또는 _issue_paper/에) 새로 추가한 논문 읽고 Schema.md 규칙대로 반영해줘"라고 요청해도 동일하게 동작한다.
3. Claude Code가 알아서:
   - PDF를 `{년도}_{venue}_{제목}.pdf`로 리네임하고, task를 판단해 `ResearchVault/Papers/_pdf/<Task>/`로 옮기고(어느 폴더에서 왔든 한 곳에 모인다)
   - `ResearchVault/Papers/<Task>_notes/`에 논문 분석 노트를 생성한다 (`task`, `direction`, `status: in-progress`, `venue`, `year`, `jcr_quartile`, `source_type` 등 속성 포함). `title`은 논문 원제만 채운다
   - 해당 `ResearchVault/Papers/<Task>_overview.md`의 "## Tasks" 목록에 이 논문을 추가하고(그 task로 처음 들어오는 논문이면 개요 노트와 `_notes` 폴더를 새로 만든다)
   - 관련된 `Concepts/` 문서를 찾아 갱신하거나 새로 만들고
   - 필요하면 `Comparisons/`에 비교 문서를 채우고
   - `reading-list.md`와 해당 `Moc/<Task>_Moc.md`도 갱신한다.
4. Obsidian으로 `ResearchVault/Papers/PaperWiki.base`를 열어 전체 논문을 가로질러 필터링하거나, 특정 분야의 `ResearchVault/Papers/<Task>_overview.md`를 열어 그 분야 목록만 본다.

완전 자동(정해진 시간마다 알아서 실행)은 지원하지 않는다 — Claude Code의 로컬 파일 접근과 "매일 정해진 시간에 알아서 실행"이 동시에 되는 방법이 없어서(클라우드 스케줄은 GitHub 등 원격 저장소 동기화가 필요), 매번 `/process-papers`로 수동 요청하는 방식을 택했다.

## 처리 여부 확인하기

**Claude Code가 노트를 만들었는지**와 **내가 실제로 읽었는지**는 다른 축이지만, 후자만 `status` 속성 하나로 관리한다.

- **Claude Code 처리 여부**: 폴더로 확인한다 — `ResearchVault/Papers/_pdf/_inbox/`나 `_issue_paper/`에 파일이 남아있으면 아직 미처리, `ResearchVault/Papers/_pdf/<Task>/`로 옮겨졌으면 노트가 만들어진 것.
- **내가 실제로 읽었는지**: 논문 분석 노트의 `status` 속성으로 관리한다 — `to-do`(아직 안 봄) / `in-progress`(Claude Code가 노트를 만든 직후 기본값, 아직 다 안 읽음) / `additional-study-needed`(한 번 읽었지만 더 깊이 볼 필요 있음) / `done`(다 읽고 이해함). Claude Code는 새 노트를 만들 때 항상 `in-progress`로 시작하고, 이 값을 스스로 `done`으로 바꾸지 않는다 — 직접 다 읽고 나서 frontmatter에서 바꾼다. `start` 속성은 그 논문 PDF를 `_inbox/`나 `_issue_paper/`에 처음 넣은 날짜.
- **직접 읽으려던 논문인지 커뮤니티 트렌드 추적용이었는지**: 논문 분석 노트의 `source_type` 속성(`personal`/`community`)으로 구분한다. `PaperWiki.base`의 "커뮤니티 트렌드" 뷰로 `community`만 걸러서 볼 수 있다.

Obsidian에서 `ResearchVault/Papers/PaperWiki.base` 파일을 열면 Notion 데이터베이스 뷰처럼, 모든 `ResearchVault/Papers/<Task>_notes/` 폴더를 가로질러 테이블로 필터링할 수 있다 (전체 / 해야할 것 / 진행 중 / 추가 공부 요청 / 완료 / Task별 / 커뮤니티 트렌드 / Q1만 / JCR 등급 확인 필요). Obsidian 버전이 오래되어 Bases가 없다면 설정 > 코어 플러그인에서 "Bases"를 켠다.

**특정 분야에만 집중하고 싶을 때**: "Task별" 뷰는 전체를 정렬만 해서 보여주므로 다른 분야가 한 화면에 섞인다. 대신 task마다 `<Task폴더명> (year)` / `<Task폴더명> (venue)` 뷰 쌍(예: "Small_Object_Detection (year)", "Small_Object_Detection (venue)")이 있다 — 그 분야 논문만 걸러서 보여주고, 연도순으로 보고 싶으면 (year) 뷰를, 학회/저널별로 묶어보고 싶으면 (venue) 뷰를 Bases 탭에서 클릭하면 된다. 또는 그 분야의 `<Task>_overview.md`를 열어 목록만 빠르게 확인해도 된다.

## 다음에 읽을 논문 찾기

`reading-list.md`를 열면 각 논문 노트가 추천한 "읽어볼 만한 논문"이 task별로 한곳에 모여있다. 참고문헌 기반 추천(원문 그대로, 신뢰도 높음)과 자유 추천("(검증 필요)" 표시 + 검색 키워드 포함, 검증 필요)이 구분되어 있다. 이 목록에 있던 논문을 실제로 읽어서 위키에 넣으면 자동으로 목록에서 빠진다.

## 새 논문을 기존 지식과 연결해서 공부하기

`/process-papers`가 논문을 정식 노트로 자동 변환하는 스킬이라면, `/study-paper`는 그 논문을 Claude Code와 **실시간으로 함께 읽을 때** 쓰는 스킬이다("이 논문 같이 읽자" 같은 요청). 본문을 읽기 전에 먼저 `000-Home.md`·해당 task의 Moc·`Concepts/`·`Architecture Design/`을 훑어, 이미 읽은 논문·개념과 어떤 모듈/메커니즘이 겹치거나 대비되는지 미리 짚어준 뒤 함께 읽어나간다 — task가 달라도 원리가 같은 개념(예: reconstruction error 기반 신호)까지 찾는다.

`000-Home.md`의 "Task를 가로지르는 개념" 섹션이 이런 교차 연결을 모아두는 곳이다 — 새 논문이 이미 있는 개념을 다른 task에서 재사용하면 이 섹션에 추가된다.

## direction 속성이란

논문이 연구 흐름에서 하는 역할을 나타내는 다중 선택 태그. 예를 들어 Attention Is All You Need, ResNet, Mamba, YOLO 같은 논문은 `foundational`로 묶이고, 기존 모델을 개선한 논문은 `improvement`, 새로운 방법론을 처음 시도하는 논문은 `novel-approach`로 묶인다. 한 논문이 여러 방향성에 동시에 해당될 수 있어 다중 선택이며, 카테고리 자체도 폐쇄 목록이 아니라 필요하면 늘어난다 (자세한 규칙은 `Schema.md`의 "direction 카테고리" 절 참고).

## 규칙을 바꾸고 싶다면

`Schema.md`를 직접 수정하면 된다. 이후 요청부터 바뀐 규칙이 적용된다.
