# Editorial Resources

PURPOSE: 整理 Google Developer Documentation Style Guide 入口頁中的 editorial resources、reference hierarchy、annotations 與 break-the-rules 原則。
APPLIES TO: documentation style decisions, editorial review, terminology decisions, spelling and formatting questions.
關鍵字: editorial resources, reference hierarchy, project-specific style, third-party references, break the rules.

> 本文件依據 [style.md.txt](https://developers.google.com/style.md.txt) 提取整理；非官方譯文。

---

## Reference hierarchy

遇到 style、用字、拼字或格式問題時，來源頁建議按下列優先順序判斷。

1. **Project-specific style**

   先遵守專案或產品自己的 style guidance。這包含：

   - 專案必要例外。
   - 產品專用術語。
   - 專案既有一致用法。
   - 專案內部已明確規定的文件格式。

2. **Google Developer Documentation Style Guide**

   若專案沒有明確規定，再遵守 Google style guide。

3. **Third-party references**

   若前兩者都沒有明確指引，再依問題類型參考第三方資料：

   - **Spelling**：參考 [Merriam-Webster.com](https://www.merriam-webster.com/)，並可搭配 Google guide 的 spelling 指引。
   - **Nontechnical style**：參考 [The Chicago Manual of Style](https://www.chicagomanualofstyle.org/home.html)。
   - **Technical style**：可參考 [Microsoft Writing Style Guide](https://learn.microsoft.com/style-guide/welcome/)，但要判斷其中規則是否只適用於 Microsoft 產品或介面。

在多個層級中，也可參考 established usage，例如搜尋組織既有文件，或用 [Google Ngram Viewer](https://books.google.com/ngrams/) 觀察廣泛語料中的用法。

---

## Other editorial resources

來源頁提到，其他社群 style guides 可用來研究與輔助判斷，但不屬於 Google developer documentation style 本身。

例子：

- [Apple Style Guide](https://help.apple.com/applestyleguide/)
- [Red Hat supplementary style guide for product documentation](https://redhat-documentation.github.io/supplementary-style-guide/)

實務含義是：可以參考其他 style guide，但不要把它們和專案規範或 Google guide 混成同等權威。若採用其他 guide 的建議，應在專案內保持一致。

---

## Annotations

來源頁提到，Google guide 中有些指引只適用於特定文件集，例如：

- Android documentation。
- Google Cloud documentation。

使用時要注意該規則是否有 scope 限制。若某條規則只適用於 Android 或 Google Cloud，不應直接推廣到所有開發者文件。

---

## Break the rules

來源頁引用 George Orwell 的原則：與其說出糟糕的句子，不如先打破規則。

這份 guide 是 guidelines，不是不可違反的硬規則。當偏離指引能讓內容更清楚、更適合讀者或更符合特定領域用法時，可以破例。

破例時要遵守兩點：

- 有理由：例如某個領域慣用 hyphenated form，而 guide 建議 one word。
- 保持一致：同一份文件或同一個專案中不要來回切換。

---

## 來源

- [Google Developer Documentation Style Guide markdown text](https://developers.google.com/style.md.txt)
- [Highlights](https://developers.google.com/style/highlights)
- [Voice and tone](https://developers.google.com/style/tone)
- [Text-formatting summary](https://developers.google.com/style/text-formatting)
- [Word list](https://developers.google.com/style/word-list)
