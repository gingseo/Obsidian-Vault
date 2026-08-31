---
Health_Exercise_LowerStretch: true
Health_Exercise_Lower: true
Health_Exercise_Cardio: true
---
# 전체 기간 트래커

`Life/Daily Note/` 아래 모든 월 폴더를 가로질러(재귀) 보는 트래커.

## 📚 순수 공부 시간 · 🍽️ 칼로리 · 🏃 운동 시간 · ⚖️ 공복 몸무게

%% col-start %%

%% col-break:31,b:secondary %%

```tracker
searchType: frontmatter
searchTarget: Learning_StudyMinutes
folder: Life/Daily Note
line:
    title: "순수 공부 시간 (분)"
    yAxisLabel: 분
    lineColor: '#5e1de0'
```

%% col-break:36,b:secondary %%

```tracker
searchType: frontmatter
searchTarget: Health_kcal, Health_Exercise
folder: Life/Daily Note
line:
    title: "칼로리 (kcal) · 운동 시간 (분)"
    yAxisLabel: kcal, 분
    yAxisLocation: left, right
    lineColor: "#ff9f43, #1f7a1f"
```

%% col-break:33,b:secondary %%

```tracker
searchType: frontmatter
searchTarget: Health_Weight
folder: Life/Daily Note
line:
    title: "공복 몸무게 (kg)"
    yAxisLabel: kg
    lineColor: '#00b8d9'
```

%% col-end %%

## 🧘 필라테스 · 🦵 하체 스트레칭 · 🏃‍♀️ 유산소

%% col-start %%

%% col-break:b:secondary %%

```tracker
searchType: frontmatter
searchTarget: Health_Exercise_Pilates
folder: Life/Daily Note
month:
    startWeekOn: "Mon"
    showCircle: true
    color: '#8f4fff'
    headerYearColor: white
    headerMonthColor: '#8f4fff'
    dividingLineColor: gray
    todayRingColor: white
```

%% col-break:b:secondary %%

```tracker
searchType: frontmatter
searchTarget: Health_Exercise_LowerStretch
folder: Life/Daily Note
month:
    startWeekOn: "Mon"
    showCircle: true
    color: '#4fa3ff'
    headerYearColor: white
    headerMonthColor: '#4fa3ff'
    dividingLineColor: gray
    todayRingColor: white
```

%% col-break:b:secondary %%

```tracker
searchType: frontmatter
searchTarget: Health_Exercise_Cardio
folder: Life/Daily Note
month:
    startWeekOn: "Mon"
    showCircle: true
    color: '#1f7a1f'
    headerYearColor: white
    headerMonthColor: '#1f7a1f'
    dividingLineColor: gray
    todayRingColor: white
```

%% col-end %%

## 💪 상체 · 🦿 하체

%% col-start %%

%% col-break:b:secondary %%

```tracker
searchType: frontmatter
searchTarget: Health_Exercise_Upper
folder: Life/Daily Note
month:
    startWeekOn: "Mon"
    showCircle: true
    color: '#00b8d9'
    headerYearColor: white
    headerMonthColor: '#00b8d9'
    dividingLineColor: gray
    todayRingColor: white
```

%% col-break:b:secondary %%

```tracker
searchType: frontmatter
searchTarget: Health_Exercise_Lower
folder: Life/Daily Note
month:
    startWeekOn: "Mon"
    showCircle: true
    color: '#ff9f43'
    headerYearColor: white
    headerMonthColor: '#ff9f43'
    dividingLineColor: gray
    todayRingColor: white
```

%% col-end %%

## 📖 독서 · 🗣️ 언어 공부 · 📄 논문 공부

%% col-start %%

%% col-break:b:secondary %%

```tracker
searchType: frontmatter
searchTarget: Learning_Book
folder: Life/Daily Note
month:
    startWeekOn: "Mon"
    showCircle: true
    color: '#e0426b'
    headerYearColor: white
    headerMonthColor: '#e0426b'
    dividingLineColor: gray
    todayRingColor: white
```

%% col-break:b:secondary %%

```tracker
searchType: frontmatter
searchTarget: Learning_Language
folder: Life/Daily Note
month:
    startWeekOn: "Mon"
    showCircle: true
    color: '#8f4fff'
    headerYearColor: white
    headerMonthColor: '#8f4fff'
    dividingLineColor: gray
    todayRingColor: white
```

%% col-break:b:secondary %%

```tracker
searchType: frontmatter
searchTarget: Learning_Paper
folder: Life/Daily Note
month:
    startWeekOn: "Mon"
    showCircle: true
    color: '#5e1de0'
    headerYearColor: white
    headerMonthColor: '#5e1de0'
    dividingLineColor: gray
    todayRingColor: white
```

%% col-end %%
