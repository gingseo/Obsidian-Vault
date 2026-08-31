# Assets 폴더 작성 스키마

이 문서는 `Life/Assets/`에 자산 기록을 작성할 때 따르는 규칙이다. `Life/Finance/`는 경제 공부 허브이며 개인 자산/지출 기록이 아니므로, 자산 기록은 이 폴더에서 별도로 관리한다.

## 폴더 구조

```
Life/Assets/
  SCHEMA.md          ← 이 문서
  _Index.md           ← 진입점
  Assets.md            ← Tracker 대시보드 (그래프 모음)
  Records/
    2026-08-30.md      ← 기록 시점 1건 = 파일 1개
    2026-09-06.md
    2026-09-30.md
    ...
```

## 기록 주기

- **주간**: 매주 일요일(또는 그 주의 마지막 기록일) 기준으로 1건.
- **월간**: 매달 말일 기준으로 1건.
- 같은 날짜에 이미 파일이 있으면 덮어쓰지 않고 그 파일을 그대로 사용/수정한다.
- 가계부가 아니다. 항목별 거래 내역을 적지 않고, **그 시점의 총액(잔고/평가액)만** frontmatter에 숫자로 기록한다.

## 파일명

`Records/YYYY-MM-DD.md` — 기록한 날짜(주간이면 그 주 기록일, 월간이면 말일).

## 새 기록 파일 추가하는 방법

`Life/Assets/Records/` 폴더 안에서 새 노트를 만들면 Templater가 `etc../Templete/Assets Record.md` 템플릿을 자동으로 채워준다 (파일명에 날짜가 오늘 날짜로 자동 입력됨).

1. `Records/` 폴더를 선택한 상태에서 새 노트 생성 (사이드바 우클릭 → New note, 또는 해당 폴더에서 `Ctrl/Cmd+N`).
2. 파일명을 `YYYY-MM-DD` 형식으로 저장 (템플릿이 `Date`를 오늘 날짜로 채워주지만, 다른 날짜로 기록할 경우 frontmatter의 `Date`와 파일명을 직접 그 날짜로 바꾼다).
3. frontmatter의 각 필드에 그 시점 숫자만 채운다. 각 필드 의미는 노트 본문에 주석(`<!-- -->`)으로 같이 적혀 있으니 파일을 열어서 참고한다.
4. `Type`은 `Weekly` 또는 `Monthly`로 적는다.
5. 저장하면 `Assets.md`의 Tracker 그래프가 자동으로 새 데이터를 반영한다 (별도 조작 불필요).

템플릿이 자동 적용되지 않으면(Obsidian을 방금 재시작했거나 설정 반영 전이면), 설정 → Templater → Folder Templates에 `Life/Assets/Records → etc../Templete/Assets Record.md`가 등록되어 있는지 확인한다. 자동 적용이 안 되면 기존 `Records/2026-08-25.md` 파일을 복사해서 날짜만 바꿔도 된다.

## Frontmatter 필드 (숫자, 단위: 원 — `Toss_USD`/`Overseas_Cost`/`Overseas_Value`만 예외로 USD/USDT)

### 있는 돈 (자산)

| 필드 | 의미 |
| --- | --- |
| `Cash` | 현금 (계좌) |
| `StockKR_Cost` | 국내주식 매입원금 (들어간 돈) |
| `StockKR_Value` | 국내주식 평가액 (현재 가치) |
| `StockUS_Cost` | 해외주식 매입원금 |
| `StockUS_Value` | 해외주식 평가액 |
| `Toss_KRW` | 증권계좌(토스) 원화 |
| `Toss_USD` | 증권계좌(토스) 달러 — **달러 액면 그대로** 기록, 원화 환산하지 않음 |
| `Upbit_Cash` | 업비트 원화 예치금 |
| `Upbit_Coin_Cost` | 업비트 코인 매입원금 |
| `Upbit_Coin_Value` | 업비트 코인 평가액 |
| `Overseas_Cost` | 해외거래소(바이낸스/게이트) 원금, **USDT 액면 그대로** (지갑/선물 구분 없이 합산) |
| `Overseas_Value` | 해외거래소 평가금, USDT 액면 그대로 |
| `ExchangeRate` | 그 시점 USD/KRW 환율 (숫자만, 예: `1380`) |
| `Overseas_Cost_KRW` | `Overseas_Cost × ExchangeRate` — **직접 입력하지 않음**, 아래 버튼으로 자동 계산 |
| `Overseas_Value_KRW` | `Overseas_Value × ExchangeRate` — 자동 계산 |
| `Loan_Out` | 빌려준 돈 |

`_Cost`(원금)와 `_Value`(평가액)를 나눈 항목(주식, 코인)은 **`Assets.md`의 전체 자산 구성 합산 그래프에는 `_Value`만 들어간다.** `_Cost`는 별도의 "원금 vs 평가액" 그래프에서만 쓰인다 — 한 막대에 원금+평가액을 같이 넣으면 합산되어 버리기 때문에 반드시 분리해서 그린다.

### 해외거래소 원화 자동 환산

Tracker는 그래프 데이터셋 자체에 사칙연산(환율 곱하기)을 적용하는 기능이 없다 (요약 텍스트에만 수식 지원). 대신 `Overseas_Cost`/`Overseas_Value`(USDT)와 `ExchangeRate`를 채운 뒤, 기록 노트 안의 **"해외거래소 원화 계산" 버튼**(Meta Bind, `etc../Templete/Assets Calc Overseas KRW.md`를 Templater로 실행)을 누르면 `Overseas_Cost_KRW`/`Overseas_Value_KRW`가 frontmatter에 자동으로 채워진다. 환율이나 USDT 값을 고치면 버튼을 다시 눌러야 갱신된다 (자동 재계산 아님).

### 나갈 돈 (부채, 매달 말 총액만 기록)

| 필드 | 의미 |
| --- | --- |
| `Debt_Card` | 카드값 (그 달 결제 예정 총액) |
| `Debt_Insurance` | 보험비 |
| `Debt_StudentLoan` | 학자금대출 잔액 |
| `Debt_KUMOH` | 과기사관 반환금 잔액 |

값을 모르거나 해당 없는 항목은 필드 자체를 생략한다 (0으로 채우지 않는다 — Tracker가 결측을 그래프에서 자연스럽게 건너뛴다).

## 기록 예시

```yaml
---
Date: 2026-08-30
Type: Monthly
Cash: 1200000
StockKR_Cost: 3200000
StockKR_Value: 3500000
StockUS_Cost: 1900000
StockUS_Value: 2100000
Toss_KRW: 800000
Toss_USD: 120
Upbit_Cash: 100000
Upbit_Coin_Cost: 850000
Upbit_Coin_Value: 900000
Overseas_Cost: 950
Overseas_Value: 1020
ExchangeRate: 1380
Overseas_Cost_KRW: 1311000
Overseas_Value_KRW: 1407600
Loan_Out: 3000000
Debt_Card: 450000
Debt_Insurance: 120000
Debt_StudentLoan: 14700000
Debt_KUMOH: 21600000
---
# 2026-08-30 자산 기록 (월간)
```

`Type`은 `Weekly` 또는 `Monthly`로 구분한다 (Tracker 조회에는 영향 없음, 사람이 보기 위한 표시).

## Assets.md 대시보드

`Assets.md`에 Tracker 코드블록으로 그래프를 그린다.

- **자산 구성 누적 막대그래프 (stacked bar)**: 자산 항목(평가액 `_Value` 기준)을 하나의 막대에 색깔별로 쌓아서, 총액 증감과 항목별 비중을 동시에 본다.
- **주식/코인 원금 vs 평가액**: `_Cost`와 `_Value`를 한 막대에 같이 넣으면 두 값이 더해져 버리므로, **원금 막대 코드블록과 평가액 막대 코드블록을 따로 만들어 `%% col-start %% / col-break %% / col-end %%`로 나란히 배치**한다 (국내주식·해외주식·업비트코인·해외거래소코인 각각 한 쌍).
- **부채 구성 누적 막대그래프**: 부채 항목을 하나의 막대에 쌓는다.
- **순자산 추이 라인 그래프**: 자산 항목 합 − 부채 항목 합 (`summary` 텍스트 또는 여러 `searchTarget`을 `+`/`-`로 연산).

새 자산 항목이 생기면 `Records/`의 frontmatter 필드를 추가하고, `Assets.md`의 해당 `searchTarget`/데이터셋 목록에도 추가한다. Tracker의 `stack` 옵션은 코드블록 전체에 한 번만 적용되므로(데이터셋별 부분 stack 불가), 원금/평가액처럼 따로 쌓아야 하는 그룹은 항상 코드블록 자체를 분리한다.

자산 구성/순자산 그래프는 `Overseas_Value_KRW`(자동 계산된 원화값)를 쓴다 — 기록 시 버튼을 누르지 않으면 이 필드가 비어 있어 해외거래소 몫이 그래프에서 빠진다.
