# PaperWiki

Claude Code가 논문 PDF를 읽고 자동으로 정리·유지해주는 개인 논문 위키.

## 구조

```
raw/
  inbox/                아직 처리 안 한 원본 PDF (다운받으면 여기에 넣는다)
  <task>/                처리 완료 후 이동되는 task별 폴더 (예: object-detection/)
wiki/
  papers/
    <task>/                raw/<task>/와 동일한 폴더 구조. 논문 노트 (논문 1편 = 파일 1개)
  concepts/              개념/기법 노트 (task로 나누지 않고 한 곳에 모음)
  comparisons/            논문 간 비교 문서 (task로 나누지 않고 한 곳에 모음)
schema.md                wiki/, raw/ 규칙 (Claude Code가 따르는 규칙)
PaperWiki.base            Obsidian Bases 뷰 정의 (상태/task/방향성 테이블)
```

## 사용법

1. 새로 읽고 싶은 논문의 PDF를 `raw/inbox/`에 넣는다.
2. 이 폴더(PaperWiki)에서 Claude Code를 열고 `/process-papers`라고만 입력한다 (긴 프롬프트를 매번 안 써도 됨 — `.claude/skills/process-papers/`에 등록된 스킬). 또는 직접 "raw/에 새로 추가한 논문 읽고 schema.md 규칙대로 wiki/에 반영해줘"라고 요청해도 동일하게 동작한다.
3. Claude Code가 알아서:
   - PDF를 `{년도}_{venue}_{제목}.pdf`로 리네임하고, task를 판단해 `raw/<task>/`로 옮기고
   - `wiki/papers/`에 논문 노트를 생성하고 (`task`, `direction`, `user_read`, `added` 속성 포함)
   - 관련된 `wiki/concepts/` 문서를 찾아 갱신하거나 새로 만들고
   - 필요하면 `wiki/comparisons/`에 비교 문서를 채우고
   - `wiki/reading-list.md`와 해당 `wiki/moc/<task>-moc.md`도 갱신한다.
4. Obsidian으로 `wiki/` 폴더를 열어 그래프 뷰/백링크로 탐색한다.

완전 자동(정해진 시간마다 알아서 실행)은 지원하지 않는다 — Claude Code의 로컬 파일 접근과 "매일 정해진 시간에 알아서 실행"이 동시에 되는 방법이 없어서(클라우드 스케줄은 GitHub 등 원격 저장소 동기화가 필요), 매번 `/process-papers`로 수동 요청하는 방식을 택했다.

## 처리 여부 확인하기

**Claude Code가 처리했는지**와 **내가 실제로 읽었는지**는 서로 다른 축이다 — 둘 다 확인할 수 있다.

- **Claude Code 처리 여부**
  - 폴더로 보기: `raw/inbox/`에 파일이 남아있으면 아직 미처리, `raw/<task>/`로 옮겨졌으면 처리 완료.
  - 속성으로 보기: `status`(`unread`/`read`/`reviewed`) 속성.
- **내가 실제로 읽었는지**: `user_read` 속성(`true`/`false`). Claude Code는 새 노트를 만들 때 항상 `false`로 시작하고, 이 값을 스스로 `true`로 바꾸지 않는다 — 직접 다 읽고 나서 frontmatter에서 `true`로 바꿔야 한다. `added` 속성은 그 논문 PDF를 `raw/inbox/`에 처음 넣은 날짜.

Obsidian에서 `PaperWiki.base` 파일을 열면 Notion 데이터베이스 뷰처럼 테이블로 필터링할 수 있다 (전체 / 미처리(Claude) / 내가 아직 안 읽음 / 리뷰 완료 / Foundational 논문 / Task별). Obsidian 버전이 오래되어 Bases가 없다면 설정 > 코어 플러그인에서 "Bases"를 켠다.

## 다음에 읽을 논문 찾기

`wiki/reading-list.md`를 열면 각 논문 노트가 추천한 "읽어볼 만한 논문"이 task별로 한곳에 모여있다. 참고문헌 기반 추천(원문 그대로, 신뢰도 높음)과 자유 추천("(검증 필요)" 표시 + 검색 키워드 포함, 검증 필요)이 구분되어 있다. 이 목록에 있던 논문을 실제로 읽어서 위키에 넣으면 자동으로 목록에서 빠진다.

## direction 속성이란

논문이 연구 흐름에서 하는 역할을 나타내는 다중 선택 태그. 예를 들어 Attention Is All You Need, ResNet, Mamba, YOLO 같은 논문은 `foundational`로 묶이고, 기존 모델을 개선한 논문은 `improvement`, 새로운 방법론을 처음 시도하는 논문은 `novel-approach`로 묶인다. 한 논문이 여러 방향성에 동시에 해당될 수 있어 다중 선택이며, 카테고리 자체도 폐쇄 목록이 아니라 필요하면 늘어난다 (자세한 규칙은 `schema.md`의 "direction 카테고리" 절 참고).

## 규칙을 바꾸고 싶다면

`schema.md`를 직접 수정하면 된다. 이후 요청부터 바뀐 규칙이 적용된다.
