
```dataviewjs

dv.span("**😴 Sleep 😴**")

const sleepIntensity = (h) => {
    if (h >= 8) return 4
    if (h >= 7) return 3
    if (h >= 6) return 2
    if (h >= 4) return 1
    return 0
}

const sleepData = {
    year: 2026,
    colors: {
        blue: ["#cfe8ff","#8fc7ff","#4fa3ff","#1d78e0","#0b4f9c"]
    },
    entries: []
}

for (let page of dv.pages('"Daily Note"').where(p => p.Health_Sleep)) {
    sleepData.entries.push({
        date: page.file.name,
        intensity: sleepIntensity(Number(page.Health_Sleep)),
        content: dv.span(`[](${page.file.name})`),
    })
}

renderHeatmapCalendar(this.container, sleepData)
```

---

```dataviewjs

dv.span("**🍽️ Calories 🍽️**")

const kcalIntensity = (k) => {
    if (k >= 2500) return 4
    if (k >= 2000) return 3
    if (k >= 1500) return 2
    if (k > 0) return 1
    return 0
}

const kcalData = {
    year: 2026,
    colors: {
        green: ["#d6f5d6","#a8e6a8","#6fcf6f","#3daa3d","#1f7a1f"]
    },
    entries: []
}

for (let page of dv.pages('"Daily Note"').where(p => p.Health_kcal)) {
    kcalData.entries.push({
        date: page.file.name,
        intensity: kcalIntensity(Number(page.Health_kcal)),
        content: dv.span(`[](${page.file.name})`),
    })
}

renderHeatmapCalendar(this.container, kcalData)
```

---

```dataviewjs

dv.span("**🏋️ Exercise Intensity 🏋️**")

const intensityMap = { "하": 1, "중": 2, "상": 3 }

const exerciseData = {
    year: 2026,
    colors: {
        red: ["#ff9e82","#ff7b55","#ff4d1a","#e73400","#bd2a00"]
    },
    entries: []
}

for (let page of dv.pages('"Daily Note"').where(p => p.Health_Exercise)) {
    exerciseData.entries.push({
        date: page.file.name,
        intensity: intensityMap[page.Health_Exercise] ?? 0,
        content: dv.span(`[](${page.file.name})`),
    })
}

renderHeatmapCalendar(this.container, exerciseData)
```

---

```dataviewjs

dv.span("**🧘 Pilates 🧘**")

const pilatesData = {
    year: 2026,
    colors: {
        purple: ["#e0cfff"]
    },
    entries: []
}

for (let page of dv.pages('"Daily Note"').where(p => p.Health_Pilates === true)) {
    pilatesData.entries.push({
        date: page.file.name,
        intensity: 1,
        content: dv.span(`[](${page.file.name})`),
    })
}

renderHeatmapCalendar(this.container, pilatesData)
```

---

```dataviewjs

dv.span("**🦵 하체 스트레칭 🦵**")

const stretchingData = {
    year: 2026,
    colors: {
        teal: ["#c2f0e8"]
    },
    entries: []
}

for (let page of dv.pages('"Daily Note"').where(p => p.Health_Stretching === true)) {
    stretchingData.entries.push({
        date: page.file.name,
        intensity: 1,
        content: dv.span(`[](${page.file.name})`),
    })
}

renderHeatmapCalendar(this.container, stretchingData)
```

---

```dataviewjs

dv.span("**💪 상체 운동 💪**")

const upperData = {
    year: 2026,
    colors: {
        orange: ["#ffe0b3"]
    },
    entries: []
}

for (let page of dv.pages('"Daily Note"').where(p => p.Health_Upper === true)) {
    upperData.entries.push({
        date: page.file.name,
        intensity: 1,
        content: dv.span(`[](${page.file.name})`),
    })
}

renderHeatmapCalendar(this.container, upperData)
```
