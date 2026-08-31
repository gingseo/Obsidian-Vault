# 💰 Assets

개인 자산/부채 기록 전용 폴더 (Finance는 경제 공부 허브이므로 분리).

- [[Life/Assets/Assets|📊 자산 트래커]] — 주간/월간 그래프
- [[Life/Assets/SCHEMA|📐 작성 스키마]] — 기록 규칙
- `Records/` — 기록 시점별 원본 파일 (직접 열어서 수정)

```dataview
LIST
FROM "Life/Assets/Records"
SORT file.name DESC
LIMIT 10
```
