---
Date: <% tp.date.now("YYYY-MM-DD") %>
Ticker: 
Action: 
Price: 
Quantity: 
TargetPrice: 
StopLoss: 
HoldingPlan: 
SellReasonTag: 
---

<!-- Action: 매수 / 매도 -->
<!-- HoldingPlan: 단타 / 스윙 / 장투 -->
<!-- SellReasonTag(매도일 때만): 계획대로 / 감정 / 새로운근거 -->

## 진입 계획 (매수 시점에 작성)

> [!note] 진입 이유
> 

**목표가:** `INPUT[number:TargetPrice]`
**손절가:** `INPUT[number:StopLoss]`

**보유 계획:**
```meta-bind
INPUT[select(option(단타), option(스윙), option(장투)):HoldingPlan]
```

## 당시 시장 상황

> 

---

## 청산 기록 (매도 시점에 작성)

**매도가/손익:** 

**매도 사유 태그:**
```meta-bind
INPUT[select(option(계획대로), option(감정), option(새로운근거)):SellReasonTag]
```

> [!note] 매도 이유
> 

> [!warning] 계획 대비 이탈
> 진입 시 목표가/손절가와 실제 매도가를 비교 — 왜 달랐는지, 그때 무슨 생각/감정이었는지
> 

---

## 회고 (나중에 작성)

> [!tip] 돌아보니
> 
