
```dataviewjs

dv.span("**📖 Reading 📖**")

const bookIntensity = { "10페이지 이내": 1, "여러 챕터": 2, "한 권": 3 }

const bookData = {
    year: 2026,
    colors: {
        blue: ["#cfe8ff","#8fc7ff","#4fa3ff","#1d78e0"]
    },
    entries: []
}

for (let page of dv.pages('"Daily Note"').where(p => p.Learning_Book)) {
    bookData.entries.push({
        date: page.file.name,
        intensity: bookIntensity[page.Learning_Book] ?? 0,
        content: dv.span(`[](${page.file.name})`),
    })
}

renderHeatmapCalendar(this.container, bookData)
```

---

```dataviewjs

dv.span("**📄 Papers 📄**")

const paperIntensity = { "가볍게 여러개 훑어보기": 1, "적당히 공부 및 기록": 2, "논문 깊게 공부": 3 }

const paperData = {
    year: 2026,
    colors: {
        purple: ["#e0cfff","#b98fff","#8f4fff","#5e1de0"]
    },
    entries: []
}

for (let page of dv.pages('"Daily Note"').where(p => p.Learning_Paper)) {
    paperData.entries.push({
        date: page.file.name,
        intensity: paperIntensity[page.Learning_Paper] ?? 0,
        content: dv.span(`[](${page.file.name})`),
    })
}

renderHeatmapCalendar(this.container, paperData)
```

---

```dataviewjs

dv.span("**🗣️ Language 🗣️**")

const languageIntensity = { "단어 혹은 짧은 회화 연습": 1, "데일리 분량": 2, "집중 공부 데이": 3 }

const languageData = {
    year: 2026,
    colors: {
        green: ["#d6f5d6","#a8e6a8","#6fcf6f","#1f7a1f"]
    },
    entries: []
}

for (let page of dv.pages('"Daily Note"').where(p => p.Learning_Language)) {
    languageData.entries.push({
        date: page.file.name,
        intensity: languageIntensity[page.Learning_Language] ?? 0,
        content: dv.span(`[](${page.file.name})`),
    })
}

renderHeatmapCalendar(this.container, languageData)
```
