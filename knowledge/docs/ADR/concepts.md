# ADR 核心概念

PURPOSE: 整理 ADR 的基本概念與使用邊界，協助判斷什麼決策該記錄、如何定義粒度，以及 decision log 如何維護。
APPLIES TO: architecture decision records, decision history, rationale documentation, architectural knowledge management.
關鍵字: ADR concepts, architectural decision, decision log, rationale, status, consequences, superseded.

> 本文件整理自 adr.github.io 首頁與 MADR overview；非官方譯文。

---

## 一份 ADR 只記錄一個決策

ADR 應捕捉單一 decision。若一份 ADR 同時記錄多個互相獨立的決策，後續會難以：

- review。
- supersede。
- 追蹤實作狀態。
- 理解某個後果來自哪個選擇。
- 在新情境中重用或反駁該決策。

好的粒度通常是：一個明確問題、一組可比較選項、一個被選方案，以及它帶來的後果。

---

## ADR 記錄「為什麼」，不只是「做了什麼」

程式碼、設定檔與部署描述通常能告訴我們系統現在是什麼樣子，但很少能完整回答為什麼。

ADR 應保留：

- 決策當時的 context。
- 要解決的 problem statement。
- 影響決策的 drivers / forces。
- 曾考慮過的 options。
- 選擇某方案的 rationale。
- 被接受的 trade-offs 與 consequences。
- 未來如何確認實作是否符合決策。

這些資訊讓後來的人能判斷：原本假設是否仍成立、決策是否需要更新、或是否應該被新的 ADR supersede。

---

## Decision log

一組 ADR 形成專案的 decision log。Decision log 是架構知識的一部分，應該和系統一起維護。

Decision log 的用途：

- 保存決策歷史。
- 提供 onboarding context。
- 支援架構 review。
- 幫助避免重複討論已決事項。
- 讓設計演進有可追蹤軌跡。

Decision log 應能被搜尋、瀏覽與排序。小專案可以用 `docs/decisions/` 目錄與 Markdown；大型專案可能需要 index、分類、portal 或 tooling。

---

## 狀態與生命週期

ADR 通常需要 status。常見狀態：

- **Proposed**：提出中，尚未接受。
- **Accepted**：已接受，作為目前設計依據。
- **Rejected**：明確不採用。
- **Deprecated**：曾經有效，但不再建議使用。
- **Superseded**：被另一份 ADR 取代。

狀態的重要性在於保留歷史，而不是刪掉舊決策。當決策改變時，通常應新增 ADR 並標註 supersedes / superseded by，讓讀者理解設計演進。

---

## ADR 與 Architecture Knowledge Management

ADR 屬於 Architectural Knowledge Management（AKM）的一部分。它把架構知識從人的記憶、會議討論與聊天紀錄中提煉成可維護文件。

但 ADR 不必只限於傳統「大架構」決策。MADR 頁面提醒，不要把 architecture 一詞解讀得過窄。只要某個 decision 對系統設計、品質或未來維護有重要影響，就值得用結構化方式捕捉。

---

## 與其他文件的邊界

ADR 不取代：

- API reference。
- 操作手冊。
- 架構圖。
- runbook。
- implementation design doc。

ADR 應連結這些文件，但它自己的主題是 decision rationale。若內容變成大量實作細節或操作步驟，應拆到對應文件，ADR 只保留與決策理由直接相關的部分。

---

## 來源

- [Architectural Decision Records](https://adr.github.io/)
- [Markdown Architectural Decision Records](https://adr.github.io/madr/)
