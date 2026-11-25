# Architectural Decision Records 知識庫

PURPOSE: 整理 Architectural Decision Records（ADR）的核心定義、使用目的與實務入口，作為記錄架構與重要設計決策的共同語言。
APPLIES TO: architecture decision making, design decision records, decision log, software architecture documentation, docs-as-code.
關鍵字: ADR, Architectural Decision Record, Architectural Decision, ASR, decision log, rationale, trade-offs, consequences.

> 本知識庫依據 [Architectural Decision Records](https://adr.github.io/) 與其 templates / practices / tooling 頁面整理為摘要與導入筆記；非官方譯文。使用時請以原文為準。

---

## ADR 是什麼

ADR（Architectural Decision Record）是用來記錄單一 architecture decision 及其 rationale 的文件。

相關術語：

- **Architectural Decision（AD）**：針對架構上重要的功能或非功能需求所做的、有理由的設計選擇。
- **Architecturally Significant Requirement（ASR）**：會對系統架構或品質產生可衡量影響的需求。
- **Architectural Decision Record（ADR）**：捕捉一個 AD、其理由、取捨與後果的記錄。
- **Decision log**：專案中一系列 ADR 的集合。

簡單說，ADR 讓後來的人知道：當時面對什麼問題、考慮過哪些選項、為什麼選了這個方案、接受了哪些後果。

---

## ADR 解決什麼問題

軟體專案常留下「最後做了什麼」，但沒有留下「為什麼這樣做」。當團隊成員變動、系統演進或決策被挑戰時，缺少 decision history 會造成重複討論與錯誤回滾。

ADR 的價值：

- 保留決策理由，而不只是最終設計。
- 讓 trade-offs 與 consequences 可見。
- 讓新成員能理解系統為何長成現在這樣。
- 支援 agile / incremental engineering：決策可逐步提出、接受、替換或廢棄。
- 讓架構知識可以以 docs-as-code 方式和程式碼一起 version。

ADR 原本屬於 Architectural Knowledge Management（AKM），但也可延伸成「Any Decision Record」，用來記錄其他重要設計或產品決策。

---

## 什麼時候需要 ADR

適合寫 ADR 的情境：

- 選擇技術棧、框架、資料庫、通訊協定或部署架構。
- 影響系統品質屬性，例如 performance、security、reliability、operability、maintainability。
- 決策有多個合理選項，且取捨會影響未來維護。
- 團隊未來可能會問「為什麼不是另一個方案」。
- 決策需要被 review、被確認實作、或將來可能 supersede。

不一定需要 ADR 的情境：

- 純實作細節，沒有長期架構或品質影響。
- 沒有實質選項的例行決定。
- 可以直接從 code 或 config 清楚看出的短期變更。

---

## 文件結構

- [`concepts.md`](concepts.md)：ADR 的基本概念、決策粒度、狀態與 decision log。
- [`templates.md`](templates.md)：MADR、Nygard ADR、Y-Statement 與其他模板的差異。
- [`practices.md`](practices.md)：建立、審查與完成 ADR 的實務要點。
- [`tooling.md`](tooling.md)：ADR tooling 類型與選型注意事項。

建議閱讀順序：

1. 先讀本頁，理解 ADR 要保存的是決策脈絡，不只是文件格式。
2. 讀 `concepts.md`，建立決策粒度與 decision log 觀念。
3. 讀 `templates.md`，選擇適合團隊的模板。
4. 讀 `practices.md`，把 ADR 納入日常決策流程。
5. 讀 `tooling.md`，評估是否需要 CLI、portal、lint 或渲染工具。

---

## 來源

- [Architectural Decision Records](https://adr.github.io/)
- [ADR Templates](https://adr.github.io/adr-templates/)
- [AD Practices](https://adr.github.io/ad-practices/)
- [Decision Capturing Tools](https://adr.github.io/adr-tooling/)
- [Markdown Architectural Decision Records](https://adr.github.io/madr/)
