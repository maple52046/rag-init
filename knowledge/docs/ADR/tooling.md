# ADR Tooling

PURPOSE: 整理 adr.github.io 的 decision capturing tools 類型，協助評估是否需要 ADR CLI、UI、rendering、lint 或架構管理工具。
APPLIES TO: ADR tooling selection, docs-as-code, MADR tools, Nygard ADR tools, architecture documentation automation.
關鍵字: ADR tooling, adr-tools, MADR, adr-log, Log4brains, Backstage ADR plugin, markdownlint, docs-as-code.

> 本文件整理自 [Decision Capturing Tools](https://adr.github.io/adr-tooling/) 與 MADR tooling 建議；非官方譯文。工具清單會變動，選用前應自行確認維護狀態與成熟度。

---

## 先決定流程，再決定工具

ADR 工具應支援團隊流程，而不是替代流程。

導入工具前先確認：

- 團隊採用哪個模板：MADR、Nygard、Y-Statement 或自訂格式。
- ADR 存放位置是否和程式碼同 repo。
- 是否需要 proposal / accepted / rejected / deprecated / superseded 生命週期。
- 是否需要生成 index 或 decision log。
- 是否需要在 developer portal 中搜尋與瀏覽。
- 是否需要 lint / CI 檢查格式一致性。

若團隊剛開始導入 ADR，純 Markdown + code review 往往已足夠。

---

## 建立與維護決策檔案

adr.github.io 將工具分成可支援任意模板、MADR 模板、Nygard 模板等類型。

任意模板工具可能支援多種格式，例如：

- ADG：用 Go 寫的 CLI，可建模、管理與重用 architectural decisions，支援 Nygard、MADR basic、QOC 等選項。
- dotnet-adr：跨平台 .NET global tool，用於建立與管理 ADR。
- ReflectRally：協作式 web application，支援建立、討論與維護 ADR。
- adr.zone：web-based ADR generator，支援 Nygard、MADR、Y-Statement、ISO/IEC/IEEE 42010-inspired 等格式。

選用這類工具時，重點是確認它是否能與團隊既有 repo、review 與 CI 流程整合。

---

## MADR 工具

MADR ecosystem 中常見工具類型：

- 生成或更新 ADR index。
- 以表單或 UI 編輯 ADR。
- VS Code extension。
- Backstage plugin，用於 developer portal 中跨 repo 搜尋與瀏覽 ADR。
- CLI 建立與更新 ADR。
- 渲染 ADR 成網站。
- 支援 proposal / acceptance / rejection / deprecation / superseding lifecycle。

adr.github.io 列出的 MADR 相關工具包含 adr-log、ADR Manager、ADR Manager VS Code Extension、Backstage ADR plugin、Hugo Markdown ADR Tools、Log4brains、pyadr 等。

---

## Nygard 工具

Nygard template 常見工具是 `adr-tools`，以 bash scripts 管理 Nygard 格式 ADR。

適合情境：

- 團隊採用簡潔的 `title / status / context / decision / consequences` 格式。
- 希望用 CLI 快速建立新 ADR。
- 不需要 MADR 的完整 options / pros / cons 結構。

---

## Rendering 與 portal

當 ADR 數量增加後，只靠檔案列表可能不夠。

可考慮：

- 產生 `index.md`。
- 用 static site generator 渲染 decision log。
- 在 developer portal 中搜尋 ADR。
- 跨多 repo 彙整 ADR。

工具例子包括 Log4brains、adr-viewer、Backstage ADR plugin 等。

---

## Lint 與 CI

MADR 頁面指出，ADR 用 Markdown 書寫時容易出現格式不一致，因此可用 markdownlint 或 CI 檢查。

Lint 可檢查：

- 標題層級。
- 清單格式。
- 檔名格式。
- 必填章節是否存在。
- 連結是否有效。

Lint 不會保證決策品質；它只能保證格式一致。決策品質仍需要 review。

---

## 接近程式碼與架構管理的工具

adr.github.io 也列出一些不是純 ADR 產生器、但與架構決策管理相關的工具：

- Embedded Architectural Decision Records：將分散式 decision log 透過 annotations 嵌入 Java code。
- ArchUnit：用 unit tests 檢查 architecture rules。
- docToolchain：docs-as-code 的軟體架構文件自動化工具。
- Structurizr：以 C4 model 視覺化、文件化與探索軟體架構。

這些工具提醒我們：ADR 不只是一堆 Markdown。重要決策可以與架構規則、圖、文件網站與測試一起形成可維護的 architecture knowledge system。

---

## 選型建議

最小導入：

- `docs/decisions/`。
- Markdown template。
- Pull request review。
- 手動 index 或簡單 README。

中等規模：

- CLI 建立新 ADR。
- markdownlint / CI。
- 自動產生 index。
- 明確狀態與 superseding convention。

大型或多 repo：

- Developer portal 或搜尋工具。
- 跨 repo ADR aggregation。
- rendering site。
- 與架構圖、測試或治理流程整合。

---

## 來源

- [Decision Capturing Tools](https://adr.github.io/adr-tooling/)
- [Markdown Architectural Decision Records](https://adr.github.io/madr/)
