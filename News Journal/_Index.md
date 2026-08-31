# 📰 News Journal

일자별 경제 뉴스/변동성 기록. 월별 폴더(`YYYY-MM/`) 안에 날짜별 파일(`YYYY-MM-DD.md`), 주간 리뷰(`_Weekly-YYYY-MM-DD.md`), 월간 리뷰(`_Monthly-YYYY-MM.md`)가 쌓인다.

## 최근 기록

```dataview
TABLE file.mtime as "수정일"
FROM "News Journal"
WHERE file.name != "_Index"
SORT file.name DESC
LIMIT 20
```

## 종목별 조회 예시

```dataview
LIST
FROM "News Journal" AND #삼성전자
SORT file.name DESC
```

## 섹터별 조회 예시

```dataview
LIST
FROM "News Journal" AND #방산
SORT file.name DESC
```
