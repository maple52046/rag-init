# Agent Skills 開放標準

PURPOSE: 整理 Agent Skills 開放標準（agentskills.io）的格式規範、撰寫指引與 client 實作指南，作為撰寫或審視 skill 規範時可引用的原始依據。
APPLIES TO: `SKILL.md` 格式、skill 目錄結構、description 撰寫與觸發、skill 評估、skill 內 scripts 設計、client 端的 skill 探索與載入行為。
關鍵字: Agent Skills, agentskills.io, SKILL.md, frontmatter, progressive disclosure, skills-ref, skill catalog, activation, .agents/skills.

> 本目錄依據 [Agent Skills](https://agentskills.io/) 官方網站整理為摘要與導入筆記；非官方譯文。使用時請以英文原文為準，原文快照見 [`source/`](source/home.md)。

---

## Agent Skills 是什麼

Agent Skills 是一種輕量、開放的格式，用來以專門知識與工作流程擴充 AI agent 的能力。一個 skill 就是一個含 `SKILL.md` 的資料夾：`SKILL.md` 內含 metadata（至少 `name` 與 `description`）與告訴 agent 如何執行某類任務的指示；資料夾也可以附帶 scripts、參考資料、範本與其他資源。

```
my-skill/
├── SKILL.md          # 必要：metadata + 指示
├── scripts/          # 選用：可執行程式
├── references/       # 選用：文件
├── assets/           # 選用：範本、資源
└── ...               # 其他任意檔案或目錄
```

官網列出的價值有三項：

- **Domain expertise**：把專門知識（從法務審查流程到資料分析 pipeline、簡報格式）封裝成可重用的指示與資源。
- **Repeatable workflows**：把多步驟任務變成一致、可稽核的程序。
- **Cross-product reuse**：寫一次，可在任何支援 skills 的 agent 使用。

這個格式最初由 [Anthropic](https://www.anthropic.com/) 開發，之後以開放標準釋出，並被越來越多 agent 產品採用；標準接受整個生態系的貢獻（討論在 [GitHub](https://github.com/agentskills/agentskills) 與 Discord）。

---

## 核心模型：三層 progressive disclosure

所有相容 agent 都採同一套三層載入策略：

| 層 | 載入內容 | 何時載入 | 預算 |
| --- | --- | --- | --- |
| 1. Catalog（Discovery） | 每個 skill 的 `name` + `description`（client 可另附 `SKILL.md` 路徑） | session 開始時，所有可用 skill | 每個 skill 約 50–100 tokens（specification 頁寫約 100 tokens） |
| 2. Instructions（Activation） | 整份 `SKILL.md` | 任務符合某 skill 的 description 時 | 建議 body 少於 5000 tokens；`SKILL.md` 少於 500 行 |
| 3. Resources（Execution） | `scripts/`、`references/`、`assets/` 內的檔案 | 指示引用到時，逐檔按需載入 | 視需要 |

完整指示只在任務需要時才載入，因此 agent 可以同時掛載許多 skill 而只佔用少量 context：裝了 20 個 skill 的 agent，不必預先付出 20 份完整指示的 token 成本。

官網各頁由此推出的後果：

- **觸發只靠 description**：agent 決定是否啟動 skill 時只看得到 catalog，description 承擔觸發的全部責任，必須同時說明「做什麼」與「何時用」；由此可推知，只寫在 body 裡的「何時使用」要等 skill 已啟動才會被讀到，對觸發沒有幫助。見 [`optimizing-descriptions.md`](optimizing-descriptions.md)。
- **body 的每個 token 都在競爭注意力**：啟動後整份 `SKILL.md` 與對話歷史、system context、其他已啟動 skill 共用 context window，所以只寫 agent 原本不知道的事。見 [`best-practices.md`](best-practices.md)。
- **細節拆到需要時才讀的檔案**：長內容移到 `references/` 等目錄，並在 `SKILL.md` 寫明何時讀哪個檔案；檔案引用只深一層。見 [`specification.md`](specification.md)。
- **簡單任務可能不觸發**：agent 通常只在自身能力不足時才查 skill；「讀這個 PDF」這類一步任務即使 description 完全符合也可能不啟動。

---

## Quickstart 範例

[Quickstart](https://agentskills.io/skill-creation/quickstart) 以 VS Code + GitHub Copilot 示範（同一個 skill 也可在 Claude Code、OpenAI Codex 等相容 agent 使用）。VS Code 預設在 `.agents/skills/` 找 skill，因此在專案建立 `.agents/skills/roll-dice/SKILL.md`：

````markdown
---
name: roll-dice
description: Roll dice using a random number generator. Use when asked to roll a die (d6, d20, etc.), roll dice, or generate a random dice roll.
---

To roll a die, use the following command that generates a random number from 1
to the given number of sides:

```bash
echo $((RANDOM % <sides> + 1))
```

```powershell
Get-Random -Minimum 1 -Maximum (<sides> + 1)
```

Replace `<sides>` with the number of sides on the die (e.g., 6 for a standard
die, 20 for a d20).
````

- `name`：skill 的短識別字，必須與資料夾名相同。
- `description`：告訴 agent 何時使用此 skill；agent 以此決定是否啟動。
- body：skill 啟動後 agent 依循的指示。

驗證方式：在 Copilot Chat 選 **Agent** 模式，輸入 `/skills` 確認 `roll-dice` 出現在清單，再問「Roll a d20」。agent 可能要求執行終端指令的權限。頁面註明 tool-use 可靠度因 model 而異；若 agent 沒有執行指令而自行回答，可換 model 再試。背後流程即上述三層：session 開始時掃描預設 skill 目錄並只讀 `name` / `description`（Discovery）、問題符合 description 時載入整份 `SKILL.md` body（Activation）、依 body 指示執行並代入骰子面數（Execution）。

---

## 文件結構

- [`specification.md`](specification.md)：格式規範（目錄、frontmatter 欄位與限制、body、選用目錄、progressive disclosure、檔案引用、驗證），以及依原始碼整理的 `skills-ref` 驗證器實際行為。
- [`best-practices.md`](best-practices.md)：skill 撰寫指引：素材來源、context 使用、控制力校準、高價值寫作模式。
- [`optimizing-descriptions.md`](optimizing-descriptions.md)：description 寫法與觸發率 eval、train / validation 優化迴圈。
- [`evaluating-skills.md`](evaluating-skills.md)：輸出品質 eval：測試案例、with / without skill 對照、assertions、評分、彙整與迭代。
- [`using-scripts.md`](using-scripts.md)：一次性指令、自帶依賴的 scripts、為 agent 設計的 script 介面。
- [`client-implementation.md`](client-implementation.md)：client 端如何探索、解析、揭露、啟動與管理 skill，以及 Client Showcase 清單。

標記慣例（各檔通用）：

- **[規範]**：specification 以 must / 硬性限制表述的規則。`skills-ref validate` 實際會擋哪些，見 [`specification.md`](specification.md#skills-ref-驗證器實際行為依原始碼)。
- **[建議]**：specification 的 should / recommendation，或撰寫指引、client 實作指南中的建議做法。
- **[現況]**：原文對既有 client 實作的描述（例如「多數實作如何做」），不是規範要求。

建議閱讀順序：先讀本頁與 `specification.md`；撰寫 skill 時讀 `best-practices.md`、`using-scripts.md`；調整觸發讀 `optimizing-descriptions.md`；驗證品質讀 `evaluating-skills.md`；需要知道 skill 會被 client 如何探索與載入時讀 `client-implementation.md`。

---

## 來源與快照

擷取時間：2026-11-24（UTC+8），自各頁的 `.md` 端點取得（頁面索引見 <https://agentskills.io/llms.txt>）。`source/` 內檔案為原樣快照，不做修改。

| 頁面 | URL | 本地快照 |
| --- | --- | --- |
| Agent Skills Overview | <https://agentskills.io/home> | [`source/home.md`](source/home.md) |
| Specification | <https://agentskills.io/specification> | [`source/specification.md`](source/specification.md) |
| Client Showcase | <https://agentskills.io/clients> | [`source/clients.jsx`](source/clients.jsx) |
| Quickstart | <https://agentskills.io/skill-creation/quickstart> | [`source/skill-creation/quickstart.md`](source/skill-creation/quickstart.md) |
| Best practices for skill creators | <https://agentskills.io/skill-creation/best-practices> | [`source/skill-creation/best-practices.md`](source/skill-creation/best-practices.md) |
| Optimizing skill descriptions | <https://agentskills.io/skill-creation/optimizing-descriptions> | [`source/skill-creation/optimizing-descriptions.md`](source/skill-creation/optimizing-descriptions.md) |
| Evaluating skill output quality | <https://agentskills.io/skill-creation/evaluating-skills> | [`source/skill-creation/evaluating-skills.md`](source/skill-creation/evaluating-skills.md) |
| Using scripts in skills | <https://agentskills.io/skill-creation/using-scripts> | [`source/skill-creation/using-scripts.md`](source/skill-creation/using-scripts.md) |
| How to add skills support to your agent | <https://agentskills.io/client-implementation/adding-skills-support> | [`source/client-implementation/adding-skills-support.md`](source/client-implementation/adding-skills-support.md) |

- Client Showcase 頁由 JavaScript 渲染，`.md` 端點只有標題，因此改存網站原始碼中的資料檔 `docs/snippets/clients.jsx`。
- 上游 repository：<https://github.com/agentskills/agentskills>，commit [`83021e2`](https://github.com/agentskills/agentskills/tree/83021e2be457a9b6679148719c1f6074f0ad975e)（`83021e2be457a9b6679148719c1f6074f0ad975e`，2026-11-05 03:47 UTC+8）。網站原始碼在 `docs/`（MDX），參考驗證器在 `skills-ref/`。`clients.jsx` 與 `skills-ref` 行為描述取自此 commit。

---

## 授權與署名

- 網站文件（`docs/`）以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 授權，全文見 [`source/LICENSE`](source/LICENSE)。本目錄的快照與摘要屬 CC BY 4.0 素材的重製與改作，署名：Agent Skills project（<https://agentskills.io/>，<https://github.com/agentskills/agentskills>）。摘要為整理與翻譯，已改動原文。
- `skills-ref` 參考函式庫以 Apache-2.0 授權（上游 `skills-ref/LICENSE`）。本目錄只描述其行為，未收錄其程式碼。
- `skills-ref` 的 README 明載它僅供示範（"intended for demonstration purposes only. It is not meant to be used in production."），不是 production 工具；引用它的行為時應視為格式規範的參考實作，而非完整的合規檢查器。
