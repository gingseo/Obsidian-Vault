<%*
const file = tp.file.find_tfile(tp.file.path(true));
const fm = app.metadataCache.getFileCache(file)?.frontmatter ?? {};
const rate = Number(fm.ExchangeRate);

if (!rate) {
    new Notice("ExchangeRate 값을 먼저 채워주세요.");
} else {
    await app.fileManager.processFrontMatter(file, (frontmatter) => {
        if (frontmatter.Overseas_Cost !== undefined && frontmatter.Overseas_Cost !== "") {
            frontmatter.Overseas_Cost_KRW = Math.round(Number(frontmatter.Overseas_Cost) * rate);
        }
        if (frontmatter.Overseas_Value !== undefined && frontmatter.Overseas_Value !== "") {
            frontmatter.Overseas_Value_KRW = Math.round(Number(frontmatter.Overseas_Value) * rate);
        }
    });
    new Notice("Overseas_*_KRW 계산 완료");
}
-%>
