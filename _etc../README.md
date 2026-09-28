# GingseoLife 볼트 가이드

라이프스타일 기록·트래킹용 Obsidian 볼트. 이 문서는 볼트 전체 구조와 사용법을 담는다. 경제 공부 폴더의 자동화 규칙은 [경제 공부/SCHEMA.md](경제%20공부/SCHEMA.md)를 참고.

## 폴더 구조

| 폴더 | 용도 |
| --- | --- |
| `How2Live/` | 생활 지혜·공부 메모 |
| `How2Live/기억할 문구/` | 인용구·문장 모음 |
| `경제 공부/` | 경제 공부 허브 (개인 지출 아님) — Wiki/Companies |
| `News Journal/` (볼트 최상위) | 뉴스·이슈 저널. 경제 공부에서 독립해 최상위로 분리됨 |
| `ResearchVault/` | 논문/연구 위키 (별도 시스템, `ResearchVault/MD_Files/Schema.md` 참고) |
| `Projects/` | 실제 진행 중인 프로젝트 전용 (논문 모음은 두지 않음 — `ResearchVault/Papers/` 참고) |
| `_etc../` | 자주 열어보지 않는 템플릿·가이드류. `Templete/` 템플릿 원본, 이 README 포함 |

> [!note] Daily Note / HeatMap / Assets / Trade Log / Review(책, 강의) / Journal은 폐지됨
> 건강·학습 일일 기록(Daily Note, HeatMap 대시보드), 자산 기록(Assets), 매매일지(Trade Log), 책/강의 리뷰(Review), 월별 개인 저널(Journal)은 더 이상 사용하지 않는다. 관련 템플릿(`_etc../Templete/`의 Daily Note·Assets Record·Assets Calc Overseas KRW·Trade Log·Journal)도 삭제했고, Templater/Daily Notes 설정에서도 매핑을 제거했다.

## 필수 플러그인

| 플러그인 | 역할 |
| --- | --- |
| Templater | 새 노트 생성 시 날짜 등 자동 치환. 폴더별 템플릿 매핑 사용 |
| Dataview | Wiki/Companies `_Index.md`의 노트 목록 자동 집계 |

## 경제 공부 / News Journal 사용법

`경제 공부/`는 개인 지출이 아니라 **경제 공부·트래킹** 전용 (Wiki/Companies). `News Journal/`은 뉴스·이슈 저널로, 경제 공부 하위가 아니라 볼트 최상위 폴더로 독립되어 있다 (자동화 대상이라 별도 관리가 편해서 분리함). 하위 폴더 역할과 자동화(클라우드 스케줄) 관련 세부 규칙은 [경제 공부/SCHEMA.md](경제%20공부/SCHEMA.md)를 참고한다.

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
