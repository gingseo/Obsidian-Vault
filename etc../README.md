# GingseoLife 볼트 가이드

라이프스타일 기록·트래킹용 Obsidian 볼트. 이 문서는 볼트 전체 구조와 사용법을 담는다. Finance 폴더의 자동화 규칙은 [Finance/SCHEMA.md](Finance/SCHEMA.md)를 참고.

## 폴더 구조

| 폴더 | 용도 |
| --- | --- |
| `Daily Note/` | 매일 생성되는 일일 노트. 건강·학습 기록 |
| `Journal/` | 월별 누적 저널. 다짐/생각을 날짜 헤딩 아래 자유서술 |
| `Finance/` | 경제 공부 허브 (개인 지출 아님) — Wiki/Companies/Reviews/News Journal/Trade Log |
| `HeatMap/` | Health/Learning 히트맵 캘린더 대시보드 |
| `Templete/` | 각 폴더용 Templater 템플릿 원본 |

## 필수 플러그인

| 플러그인 | 역할 |
| --- | --- |
| Templater | 새 노트 생성 시 날짜 등 자동 치환. 폴더별 템플릿 매핑 사용 |
| Meta Bind | 노트 본문에 입력창(토글/숫자/드롭다운)을 삽입해 값을 frontmatter 속성에 자동 저장 |
| Dataview | Wiki/Companies/Reviews `_Index.md`의 노트 목록 자동 집계 |
| Heatmap Calendar | HeatMap 폴더의 dataviewjs 블록에서 사용하는 캘린더 렌더러 |
| Calendar | Daily Note 캘린더 뷰 |

## Daily Note 사용법

**새 노트 만들기**: 코어 Daily Notes 플러그인의 "오늘의 일일 노트 열기"(리본 아이콘)를 사용한다. Templater 폴더 매핑(`Daily Note` 폴더 → `Templete/Daily Note.md`)이 걸려 있어 자동으로 템플릿이 채워진다.

> [!warning] "Templater: Create new note from template" 명령은 쓰지 않는다
> 이 명령은 폴더 매핑을 무시하고 `Templete` 폴더에 `Untitled`로 파일을 만든다. 반드시 코어 Daily Notes 경로로 생성할 것.

### 속성(frontmatter) 목록

| 속성 | 타입 | 값 | 히트맵 대상 |
| --- | --- | --- | --- |
| `Health_Sleep` | 숫자 | 수면 시간(시간 단위) | HeatMap/Health |
| `Health_Exercise` | 드롭다운 | 하 / 중 / 상 | HeatMap/Health |
| `Health_Pilates` | 토글 | true/false | HeatMap/Health |
| `Health_Stretching` | 토글 | true/false | HeatMap/Health |
| `Health_Upper` | 토글 | true/false | HeatMap/Health |
| `Health_kcal_Morning` / `_Afternoon` / `_Evening` | 숫자 | 끼니별 칼로리 | — |
| `Health_kcal` | 숫자 (자동계산) | 하루 총 칼로리, `VIEW` 필드가 세 끼니 합을 자동 반영 | HeatMap/Health |
| `Learning_Book` | 드롭다운 | 10페이지 이내 / 여러 챕터 / 한 권 | HeatMap/Learning |
| `Learning_Paper` | 드롭다운 | 가볍게 여러개 훑어보기 / 적당히 공부 및 기록 / 논문 깊게 공부 | HeatMap/Learning |
| `Learning_Language` | 드롭다운 | 단어 혹은 짧은 회화 연습 / 데일리 분량 / 집중 공부 데이 | HeatMap/Learning |

새 속성을 추가할 땐 `Health_` / `Learning_` 접두사를 유지하고, 같은 이름을 재사용하지 않는다 (Meta Bind의 `INPUT[toggle:속성명]`은 같은 속성명을 쓰는 모든 필드가 서로 연동되어 값이 겹친다).

### Meta Bind 문법 메모

- `INPUT[...]`, `VIEW[...]`는 **인라인 코드(백틱)로 감싸야** 렌더링된다. Meta Bind는 렌더링된 HTML의 `<code>` 태그만 스캔해서 문법을 찾기 때문.
  - 예외: `select(...)` 타입은 백틱 대신 ` ```meta-bind ` 코드블록으로 감싸야 한다 (인라인 코드 안에서는 select 위젯 생성이 금지되어 있음).
- 마크다운 **표(테이블) 셀 안에서는 인라인 필드가 정상 렌더링되지 않는 경우가 있다** — 목록(bullet)이나 일반 문단으로 작성할 것.
- `VIEW[...][math:속성명]`처럼 계산식을 다른 속성에 자동 저장하려면 타입을 `number`가 아니라 **`math`**로 지정해야 한다.
- 설정 > Meta Bind > Excluded folders가 비어 있어야 템플릿 폴더에서도 미리보기가 된다.

## Journal 사용법

`Journal/YYYY-MM.md` 형태로 월 단위 누적. 새 달 노트는 `Journal` 폴더에 새 노트를 만들면 Templater가 자동으로 `# YYYY-MM` 제목과 오늘 날짜 헤딩을 채운다. 이후 매일 손으로 `## YYYY-MM-DD` 헤딩을 추가하고 그 아래 자유롭게 기록한다. 특정 생각이 길어지면 `[[페이지명]]`으로 별도 노트를 만들어 링크만 남긴다.

## Finance 사용법

개인 지출이 아니라 **경제 공부·트래킹** 전용. 하위 폴더 역할과 자동화(클라우드 스케줄) 관련 세부 규칙은 [Finance/SCHEMA.md](Finance/SCHEMA.md)를 참고한다.

## HeatMap 사용법

`HeatMap/Health.md`, `HeatMap/Learning.md`는 각각 dataviewjs 블록으로 `Daily Note` 폴더를 스캔해서 히트맵 캘린더를 그린다. 텍스트 단계값(하/중/상 등)은 블록 내부의 매핑 객체(`intensityMap` 등)로 숫자 intensity로 변환된다. 새 단계값을 추가하면 이 매핑도 같이 갱신해야 한다.

히트맵 캘린더는 **데이터 안에서 상대적 스케일**로 색상 강도를 계산한다 — 기록이 하루뿐이면 무조건 가장 진한 색으로 보이는 게 정상이며, 기록이 쌓일수록 상대적으로 조정된다.

## GitHub 연동 메모

`Obsidian-Vault` 저장소(`gingseo/Obsidian-Vault`)에 볼트를 push해두면, claude.ai 클라우드 스케줄(routine)이 자동으로 뉴스 조사·기록 작업을 수행할 수 있다.

> [!warning] 클라우드 routine은 기본 GitHub 연동으로 push할 수 없다
> claude.ai 설정의 기본 "GitHub 연동" 커넥터는 읽기 전용이다. routine이 저장소에 커밋하려면 **커스텀 MCP 커넥터**를 GitHub 원격 MCP 서버(`https://api.githubcopilot.com/mcp/`)로 별도 추가하고, GitHub에서 발급받은 OAuth Client ID/Secret으로 인증해야 한다 (Authorization callback URL: `https://claude.ai/api/mcp/auth_callback`). routine 지침에는 "GitHub MCP 커넥터로 main 브랜치에 직접 커밋"하도록 명시할 것 — 그렇지 않으면 임의의 새 브랜치에만 커밋되고 `main`에는 반영되지 않는다.

### 로컬 ↔ GitHub 자동 동기화 (Obsidian Git)

클라우드 routine은 GitHub `main`에 커밋할 뿐, 로컬 볼트로 자동으로 가져오지는 않는다. **Obsidian Git** 플러그인이 이 로컬↔원격 동기화를 담당한다.

- 설정: `autoSaveInterval` / `autoPushInterval` / `autoPullInterval` 모두 1440분(24시간)으로 하루 1회 자동 pull → commit → push.
- `autoPullOnBoot: true` — Obsidian을 열 때마다 최신 원격 내용을 먼저 받아온다.
- 이 타이머는 **Obsidian이 실행 중일 때만** 동작한다. 앱이 꺼져 있으면 자동 동기화도 멈춘다.
- 급하게 최신 내용을 받고 싶으면 명령 팔레트(`Cmd+P`) → "Git: Pull"을 수동 실행해도 된다.
- 로컬에서 여러 파일을 고친 상태로 원격에 새 routine 커밋이 쌓이면 병합 충돌이 날 수 있다 — 이 경우 Obsidian Git이 충돌 마커(`<<<<<<<`)를 파일에 남기므로 직접 열어 해결하고 다시 커밋해야 한다.
