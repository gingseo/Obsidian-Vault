---
Date: <% tp.date.now("YYYY-MM-DD") %>
Type: 
Cash:
StockKR_Cost:
StockKR_Value:
StockUS_Cost:
StockUS_Value:
Toss_KRW:
Toss_USD:
Upbit_Cash:
Upbit_Coin_Cost:
Upbit_Coin_Value:
Overseas_Cost:
Overseas_Value:
ExchangeRate:
Overseas_Cost_KRW:
Overseas_Value_KRW:
Loan_Out:
Debt_Card:
Debt_Insurance:
Debt_StudentLoan:
Debt_KUMOH:
---

<!-- Type: Weekly / Monthly -->

<!-- 있는 돈 (자산) -->
<!-- Cash: 현금 (계좌) -->
<!-- StockKR_Cost: 국내주식 매입원금 (들어간 돈) -->
<!-- StockKR_Value: 국내주식 평가액 (현재 가치) -->
<!-- StockUS_Cost: 해외주식 매입원금 -->
<!-- StockUS_Value: 해외주식 평가액 -->
<!-- Toss_KRW: 토스 증권계좌 원화 -->
<!-- Toss_USD: 토스 증권계좌 달러 (달러 액면 그대로, 환산하지 않음) -->
<!-- Upbit_Cash: 업비트 원화 예치금 -->
<!-- Upbit_Coin_Cost: 업비트 코인 매입원금 -->
<!-- Upbit_Coin_Value: 업비트 코인 평가액 -->
<!-- Overseas_Cost: 해외거래소(바이낸스/게이트) 원금, USDT 액면 그대로 -->
<!-- Overseas_Value: 해외거래소 평가금, USDT 액면 그대로 (지갑/선물 구분 없이 합산) -->
<!-- ExchangeRate: 그 시점 USD/KRW 환율 (예: 1380) -->
<!-- Overseas_Cost_KRW / Overseas_Value_KRW: 자동 계산됨 — 아래 버튼 클릭 -->
<!-- Loan_Out: 빌려준 돈 -->

<!-- 나갈 돈 (부채, 매달 말 총액만) -->
<!-- Debt_Card: 카드값 (그 달 결제 예정 총액) -->
<!-- Debt_Insurance: 보험비 -->
<!-- Debt_StudentLoan: 학자금대출 잔액 -->
<!-- Debt_KUMOH: 과기사관 반환금 잔액 -->

<!-- 값을 모르거나 해당 없으면 필드를 비워두거나 삭제 (0으로 채우지 않기) -->
<!-- 자산 구성 그래프(Assets 전체 합산)에는 *_Value(평가액)만 쓰인다. *_Cost(원금)는 주식/코인 원금 vs 평가액 비교 그래프 전용 -->

# <% tp.date.now("YYYY-MM-DD") %> 자산 기록

Overseas_Cost / Overseas_Value / ExchangeRate를 채운 뒤 아래 버튼을 눌러 원화 환산값을 계산한다.

```meta-bind-button
label: 해외거래소 원화 계산
icon: calculator
style: primary
actions:
  - type: runTemplaterFile
    templateFile: "etc../Templete/Assets Calc Overseas KRW.md"
```
