# 💰 매매일지

거래 발생 시에만 기록. 새 거래는 [[Templete/Trade Log|매매일지 템플릿]]으로 생성.

## 목록

```dataview
TABLE Ticker as "종목", Action as "매수/매도", Price as "가격", Date as "날짜"
FROM "Finance/Trade Log"
WHERE file.name != "_Index"
SORT Date DESC
```
