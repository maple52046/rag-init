# Client 端的 Skills 支援

PURPOSE: 整理 agentskills.io 給 agent / 開發工具實作者的 skills 支援指南：探索、解析、揭露、啟動與 context 管理的完整生命週期，以及 Client Showcase 收錄的 client 清單。
APPLIES TO: 理解 client 會在哪裡找到 skill、如何解析與載入 `SKILL.md`、model 實際收到什麼內容；據此決定 skill 的放置位置與寫法，或實作 skills 支援。
關鍵字: skills client implementation, skill discovery, .agents/skills, .claude/skills, name collision, trust, lenient validation, skill catalog, available_skills, activate_skill, slash command, context compaction, subagent, Client Showcase.

> 來源：[How to add skills support to your agent](https://agentskills.io/client-implementation/adding-skills-support)，本地快照 [`source/client-implementation/adding-skills-support.md`](source/client-implementation/adding-skills-support.md)；client 清單來自 [Client Showcase](https://agentskills.io/clients) 的資料檔 [`source/clients.jsx`](source/clients.jsx)。本文件為摘要，非官方譯文，以原文為準。

這份指南是寫給 client 實作者的 how-to：大部分內容是 **[建議]** 做法，不是 specification 的要求；原文描述既有實作現況之處標為 **[現況]**。specification 只定義 skill 目錄**裡面**有什麼，不規定 skill 目錄放在哪裡。從 skill 作者的角度讀，這些建議說明了 skill 在各 client 可能被如何探索與載入，但個別 client 的實際行為仍要查它自己的文件（見文末清單的 docs 連結）。

---

## 整體架構

不論 agent 架構為何，核心整合都相同；實作細節取決於兩個因素：

- **skill 放在哪裡**：在本機執行的 agent 可以掃描使用者檔案系統的 skill 目錄；雲端或 sandbox 中的 agent 需要替代的探索機制（API、遠端 registry 或打包的 assets）。
- **model 如何取得 skill 內容**：model 有讀檔能力時可直接讀 `SKILL.md`；否則由 client 提供專用工具，或以程式把 skill 內容注入 prompt。

不必支援每種情境，選符合自己 agent 的路徑即可。

### 核心原則：progressive disclosure

所有相容 agent 都採三層載入策略：

| 層 | 載入內容 | 時機 | Token 成本 |
| --- | --- | --- | --- |
| 1. Catalog | name + description | session 開始 | 每個 skill 約 50–100 tokens |
| 2. Instructions | 整份 `SKILL.md` body | skill 啟動時 | 建議少於 5000 tokens |
| 3. Resources | scripts、references、assets | 指示引用到時 | 不定 |

model 從一開始就看到 catalog，知道有哪些 skill；判斷某 skill 相關時載入完整指示；指示引用支援檔時再逐一按需載入。裝了 20 個 skill 的 agent 不必預先付出 20 份完整指示的 token 成本，只付實際用到的。

---

## Step 1：探索 skill

session 啟動時找出所有可用 skill 並載入其 metadata。

### 掃描位置

多數本機 agent 至少掃兩個 scope **[現況]**：

- **Project-level**（相對於工作目錄）：專案或 repository 專屬的 skill。
- **User-level**（相對於 home 目錄）：該使用者在所有專案都可用的 skill。

也可以有其他 scope，例如管理者部署的組織層 skill，或隨 agent 內建的 skill。

每個 scope 內，**[建議]** 同時掃 client 專屬目錄與 `.agents/skills/` 慣例：

| Scope | 路徑 | 用途 |
| --- | --- | --- |
| Project | `<project>/.<your-client>/skills/` | client 原生位置 |
| Project | `<project>/.agents/skills/` | 跨 client 互通 |
| User | `~/.<your-client>/skills/` | client 原生位置 |
| User | `~/.agents/skills/` | 跨 client 互通 |

`.agents/skills/` 已成為廣泛採用的跨 client 共享慣例 **[現況]**。掃描它代表其他相容 client 安裝的 skill 會自動對你的 client 可見，反之亦然。

其他位置 **[現況]**：部分實作為了務實相容也掃 `.claude/skills/`（project-level 與 user-level 皆有），因為許多既有 skill 安裝在那裡。其他附加位置包括往上到 git root 的祖先目錄（適合 monorepo）、[XDG](https://specifications.freedesktop.org/basedir-spec/latest/) 設定目錄，以及使用者自訂路徑。

### 掃描對象

在每個 skills 目錄內，找**含有名稱恰為 `SKILL.md` 之檔案的子目錄**：

```
~/.agents/skills/
├── pdf-processing/
│   ├── SKILL.md          ← discovered
│   └── scripts/
│       └── extract.py
├── data-analysis/
│   └── SKILL.md          ← discovered
└── README.md             ← ignored (not a skill directory)
```

實務掃描規則 **[建議]**：

- 跳過不會含 skill 的目錄，例如 `.git/`、`node_modules/`。
- 可選擇遵循 `.gitignore`，避免掃描 build 產物。
- 設合理上限（例如最大深度 4–6 層、最多 2000 個目錄），防止在大型目錄樹中失控掃描。

後果（推論，原文未明說）：深度上限意味著掃描會往下遞迴尋找含 `SKILL.md` 的子目錄，而不只看 skills 目錄的第一層。因此放在某個 skill 內部更深處、名稱恰為 `SKILL.md` 的檔案（例如供複製用的範本）在這類 client 上也可能被當成獨立 skill 探索並載入。

### 處理撞名

兩個 skill 的 `name` 相同時，套用確定性的優先規則。

- **[現況]** 既有實作的普遍慣例：**project-level skill 覆蓋 user-level skill**。
- **[建議]** 同一 scope 內（例如 `<project>/.agents/skills/` 與 `<project>/.<your-client>/skills/` 都有 `code-review`），取先找到或後找到皆可，但要選定一種並保持一致；發生撞名時記錄警告，讓使用者知道有 skill 被遮蔽。

### 信任

project-level skill 來自正在處理的 repository，可能不受信任（例如剛 clone 的開源專案）。**[建議]** 以信任檢查把關 project-level skill 的載入：只有使用者把專案資料夾標為信任時才載入，避免不受信任的 repository 悄悄把指示注入 agent 的 context。

### 雲端與 sandbox 中的 agent

agent 在 container 或遠端伺服器執行時，無法存取使用者本機檔案系統，探索方式依 scope 而不同：

- **Project-level skill** 通常最簡單：agent 操作 clone 下來的 repository 時（即使在 sandbox 內），project-level skill 隨程式碼一起到位，可從 repo 目錄樹掃描。
- **User-level 與 organization-level skill** 不存在於 sandbox，需要從外部來源供應，例如 clone 一個設定 repository、在 agent 設定中接受 skill URL 或套件、或讓使用者透過 web UI 上傳 skill 目錄。
- **內建 skill** 可打包成 agent 部署產物中的靜態 assets，每個 session 都可用，不需外部抓取。

skill 到位後，其餘生命週期（解析、揭露、啟動）都相同。

---

## Step 2：解析 `SKILL.md`

對每個探索到的 `SKILL.md` 取出 metadata 與 body。

### 取出 frontmatter

`SKILL.md` 有兩部分：`---` 分隔符之間的 YAML frontmatter，以及結束分隔符後的 Markdown body。解析步驟：

1. 找檔案開頭的 `---` 與其後的結束 `---`。
2. 解析兩者之間的 YAML，取出 `name` 與 `description`（必要）以及任何選用欄位。
3. 結束 `---` 之後的所有內容（trim 後）就是 skill 的 body。

欄位與限制見 [`specification.md`](specification.md#skillmd-格式)。

### 處理格式錯誤的 YAML

為其他 client 撰寫的 skill 檔可能含技術上不合法、但其 parser 恰好接受的 YAML。最常見的是未加引號且含冒號的值：

```yaml
# Technically invalid YAML — the colon breaks parsing
description: Use this skill when: the user asks about PDFs
```

**[建議]** 提供 fallback：把這類值包上引號或轉成 YAML block scalar 後重試。以極低成本提升跨 client 相容性。（`skills-ref` 驗證器遇到這種寫法會直接解析失敗，見 [`specification.md`](specification.md#frontmatter-解析)。）

### 寬鬆驗證

**[建議]** 有問題時警告，但盡可能仍載入 skill：

| 狀況 | 處理 |
| --- | --- |
| `name` 與上層目錄名不符 | 警告，照常載入 |
| `name` 超過 64 字元 | 警告，照常載入 |
| `description` 缺漏或為空 | 略過該 skill（description 是揭露的必要條件），記錄錯誤 |
| YAML 完全無法解析 | 略過該 skill，記錄錯誤 |

記錄診斷資訊以便呈現給使用者（debug 指令、log 檔或 UI），但不要因外觀問題阻擋載入。原文註明：specification 對 `name` 定義了嚴格限制（與上層目錄相符、字元集、最大長度），上述寬鬆做法是刻意放寬，以提升與其他 client 所寫 skill 的相容性。所以「某 client 載得起來」不等於「符合 specification」。

### 要儲存什麼

每筆 skill 紀錄至少需要三個欄位，存在以 `name` 為 key 的記憶體 map 中以利啟動時查找：

| 欄位 | 說明 |
| --- | --- |
| `name` | 來自 frontmatter |
| `description` | 來自 frontmatter |
| `location` | `SKILL.md` 的絕對路徑 |

body 可以在探索時就存起來，也可以在啟動時從 `location` 讀取：存起來啟動較快；啟動時讀取總體記憶體較省，且能讀到兩次啟動之間對 skill 檔的修改。skill 的 **base directory**（`location` 的上層目錄）之後用來解析相對路徑與列舉 bundle 資源，需要時從 `location` 推導。

---

## Step 3：揭露可用 skill 給 model

告訴 model 有哪些 skill，但不載入完整內容（progressive disclosure 第 1 層）。

### 建立 skill catalog

每個 skill 放入 `name`、`description`，以及選用的 `location`（`SKILL.md` 路徑），格式可依技術棧選 XML、JSON 或條列：

```xml
<available_skills>
  <skill>
    <name>pdf-processing</name>
    <description>Extract PDF text, fill forms, merge files. Use when handling PDFs.</description>
    <location>/home/user/.agents/skills/pdf-processing/SKILL.md</location>
  </skill>
  <skill>
    <name>data-analysis</name>
    <description>Analyze datasets, generate charts, and create summary reports.</description>
    <location>/home/user/project/.agents/skills/data-analysis/SKILL.md</location>
  </skill>
</available_skills>
```

`location` 有兩個用途：讓讀檔式啟動成為可能，並給 model 解析 skill body 中相對引用（例如 `scripts/evaluate.py`）的基準路徑。若專用啟動工具的回傳結果會提供 skill 目錄路徑（見下方「結構化包裝」），可在 catalog 省略 `location`；否則要保留。每個 skill 約為 catalog 增加 50–100 tokens，裝了數十個 skill 也仍精簡。`skills-ref to-prompt` 可產生這種 XML，見 [`specification.md`](specification.md#其他-cli-指令)。

### catalog 放在哪裡

兩種常見做法，都可行：

- **System prompt 區段**：在 system prompt 中加一個有標籤的區段，前面附簡短的使用說明。最簡單，適用於任何有讀檔工具的 model。
- **工具描述**：把 catalog 嵌入專用 skill 啟動工具的描述中。system prompt 保持乾淨，並自然地把探索與啟動綁在一起。

system prompt 較簡單、相容性較廣；有專用啟動工具時，嵌入工具描述較乾淨。

### 行為指示

在 catalog 旁附一小段指示，告訴 model 如何及何時使用 skill；措辭依支援的啟動機制而定。

model 以讀檔啟動 skill 時：

```
The following skills provide specialized instructions for specific tasks.
When a task matches a skill's description, use your file-read tool to load
the SKILL.md at the listed location before proceeding.
When a skill references relative paths, resolve them against the skill's
directory (the parent of SKILL.md) and use absolute paths in tool calls.
```

model 以專用工具啟動 skill 時：

```
The following skills provide specialized instructions for specific tasks.
When a task matches a skill's description, call the activate_skill tool
with the skill's name to load its full instructions.
```

保持簡短：目的是告訴 model 有 skill 存在以及如何載入，詳細指示由載入後的 skill 內容提供。

### 過濾

有些 skill 應排除在 catalog 外，常見原因：

- 使用者在設定中停用了該 skill。
- 權限系統拒絕存取該 skill。
- skill 選擇不讓 model 自動啟動（例如透過 `disable-model-invocation` 旗標）。

**[建議]** 被過濾的 skill **從 catalog 完全隱藏**，而不是列出後在啟動時阻擋，避免 model 浪費回合嘗試載入用不了的 skill。注意 `disable-model-invocation` 這類 client 專屬旗標不在 specification 的 frontmatter 欄位中，`skills-ref` 會把它當成未知欄位報錯（見 [`specification.md`](specification.md#欄位檢查)）。

### 沒有可用 skill 時

完全不探索到 skill 時，整個省略 catalog 與行為指示。不要顯示空的 `<available_skills/>` 區塊，也不要註冊沒有有效選項的 skill 工具，否則會讓 model 困惑。

---

## Step 4：啟動 skill

model 或使用者選定 skill 時，把完整指示送進對話 context（progressive disclosure 第 2 層）。

### Model 驅動的啟動

**[現況]** 多數實作依賴 model 自己的判斷作為啟動機制，而不是在 harness 端實作觸發比對或關鍵字偵測：model 讀 catalog、判斷某 skill 與當前任務相關，然後載入它。兩種實作模式：

- **讀檔啟動**：model 以標準讀檔工具讀取 catalog 中的 `SKILL.md` 路徑，以 tool result 收到檔案內容。不需要特別基礎設施；model 有檔案存取權時最簡單。
- **專用工具啟動**：註冊一個工具（例如 `activate_skill`），輸入 skill 名稱、回傳內容。model 無法直接讀檔時必須採用；能讀檔時也可選用。相較直接讀檔的優點：
  - 控制回傳內容，例如移除或保留 YAML frontmatter。
  - 以結構化標籤包裝內容，方便 context 管理時辨識。
  - 在指示旁列出 bundle 資源（例如 `references/*`）。
  - 執行權限檢查或請求使用者同意。
  - 追蹤啟動以供分析。

**[建議]** 使用專用啟動工具時，把 `name` 參數限制在有效 skill 名稱集合內（例如在 tool schema 中用 enum），避免 model 幻想出不存在的 skill 名稱。沒有可用 skill 時，完全不要註冊此工具。

### 使用者顯式啟動

使用者也應能直接啟動 skill，不必等 model 判斷。**[現況]** 最常見的是由 harness 攔截的 **slash command 或 mention 語法**（`/skill-name` 或 `$skill-name`）。具體語法由各 client 決定；重點是 harness 負責查找與注入，model 不需要自己採取啟動動作就收到 skill 內容。輸入時列出可用 skill 的 autocomplete 元件可讓此功能更易被發現。

### Model 收到什麼

skill 啟動時 model 收到的內容有兩種選擇，實務上都可行：

- **整份檔案**：model 看到含 YAML frontmatter 的整份 `SKILL.md`。讀檔啟動自然就是這樣；專用工具也可這樣做。frontmatter 可能含啟動時有用的欄位，例如 `compatibility` 註明的環境需求可影響 model 如何執行指示。
- **只有 body（移除 frontmatter）**：harness 解析並移除 YAML frontmatter，只回傳 Markdown 指示。**[現況]** 在有專用啟動工具的既有實作中，多數採用此做法：探索時取出 `name` 與 `description` 後就剝除 frontmatter。

後果（推論）：在採「只有 body」的 client 上，skill 啟動後 model 看不到 frontmatter；`compatibility` 等 frontmatter 內的資訊若執行時需要，body 中也要寫到。

### 結構化包裝

**[建議]** 使用專用啟動工具時，以識別用標籤包裝 skill 內容，例如：

```xml
<skill_content name="pdf-processing">
# PDF Processing

## When to use this skill
Use this skill when the user needs to work with PDF files...

[rest of SKILL.md body]

Skill directory: /home/user/.agents/skills/pdf-processing
Relative paths in this skill are relative to the skill directory.

<skill_resources>
  <file>scripts/extract.py</file>
  <file>scripts/merge.py</file>
  <file>references/pdf-spec-summary.md</file>
</skill_resources>
</skill_content>
```

好處：

- model 能清楚區分 skill 指示與其他對話內容。
- harness 能在 context compaction 時辨識 skill 內容（見 Step 5）。
- bundle 資源呈現給 model，但不預先載入。

### 列出 bundle 資源

專用啟動工具回傳 skill 內容時，可以一併列舉 skill 目錄中的支援檔（scripts、references、assets），但**不應預先讀取它們**；model 在 skill 指示引用到時，再用讀檔工具按需載入特定檔案。skill 目錄很大時，考慮限制列舉數量並註明清單可能不完整。

### 權限 allowlist

若 agent 有管控檔案存取的權限系統，**[建議]** 把 skill 目錄加入 allowlist，讓 model 讀取 bundle 資源時不觸發使用者確認。否則每次引用 bundle 內的 script 或 reference 都會跳出權限對話框，打斷含有 `SKILL.md` 以外資源的 skill 的流程。

---

## Step 5：隨時間管理 skill context

skill 指示進入對話 context 後，要讓它在整個 session 中持續有效。

### 保護 skill 內容免於 context compaction

若 agent 在 context window 滿時截斷或摘要較舊訊息，**[建議]** **把 skill 內容排除在修剪之外**。skill 指示是持久的行為指引；在對話中途遺失它們會悄悄降低 agent 表現而沒有任何可見錯誤：model 繼續運作，但少了 skill 提供的專門指示。常見做法：

- 把 skill 工具輸出標為受保護，讓修剪演算法跳過。
- 用 Step 4 的結構化標籤辨識 skill 內容，在 compaction 時保留。

### 啟動去重

**[建議]** 追蹤當前 session 已啟動的 skill。model（或使用者）嘗試載入已在 context 中的 skill 時，可略過重新注入，避免同一份指示在對話中出現多次。

### Subagent 委派（選用）

進階模式，只有部分 client 支援 **[現況]**：不把 skill 指示注入主對話，而是在**獨立的 subagent session** 中執行 skill。subagent 收到 skill 指示、執行任務，並把工作摘要回傳主對話。適合 skill 工作流程複雜到值得一個專注的獨立 session 時。

---

## Client Showcase

[Client Showcase](https://agentskills.io/clients) 列出支援 Agent Skills 格式的 agent 產品。下表依 commit `83021e2` 的 `docs/snippets/clients.jsx` 原始順序列出全部 46 個（網站提供隨機與字母排序檢視）；「Skills 文件」為資料檔中的 `instructionsUrl`。收錄於此清單代表該產品宣稱支援，各 client 實際支援的探索位置、啟動方式與 frontmatter 欄位仍以其文件為準。

| # | Client | Skills 文件 |
| --- | --- | --- |
| 1 | [Junie](https://junie.jetbrains.com/) | [docs](https://junie.jetbrains.com/docs/agent-skills.html) |
| 2 | [ZeroClaw](https://www.zeroclawlabs.ai/) | [docs](https://docs.zeroclawlabs.ai/master/en/tools/skills.html) |
| 3 | [Gemini CLI](https://geminicli.com) | [docs](https://geminicli.com/docs/cli/skills/) |
| 4 | [Autohand Code CLI](https://autohand.ai/) | [docs](https://autohand.ai/docs/working-with-autohand-code/agent-skills.html) |
| 5 | [OpenCode](https://opencode.ai/) | [docs](https://opencode.ai/docs/skills/) |
| 6 | [OpenHands](https://openhands.dev/) | [docs](https://docs.openhands.dev/overview/skills) |
| 7 | [Mux](https://mux.coder.com/) | [docs](https://mux.coder.com/agent-skills) |
| 8 | [Cursor](https://cursor.com/) | [docs](https://cursor.com/docs/context/skills) |
| 9 | [Amp](https://ampcode.com/) | [docs](https://ampcode.com/manual#agent-skills) |
| 10 | [Letta](https://www.letta.com/) | [docs](https://docs.letta.com/letta-code/skills/) |
| 11 | [Firebender](https://firebender.com/) | [docs](https://docs.firebender.com/multi-agent/skills) |
| 12 | [Goose](https://block.github.io/goose/) | [docs](https://block.github.io/goose/docs/guides/context-engineering/using-skills/) |
| 13 | [GitHub Copilot](https://github.com/) | [docs](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) |
| 14 | [VS Code](https://code.visualstudio.com/) | [docs](https://code.visualstudio.com/docs/copilot/customization/agent-skills) |
| 15 | [Claude Code](https://claude.ai/code) | [docs](https://code.claude.com/docs/en/skills) |
| 16 | [Claude](https://claude.ai/) | [docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) |
| 17 | [ChatGPT & Codex](https://chatgpt.com/codex/) | [docs](https://developers.openai.com/codex/skills/) |
| 18 | [Piebald](https://piebald.ai) | — |
| 19 | [Factory](https://factory.ai/) | [docs](https://docs.factory.ai/cli/configuration/skills) |
| 20 | [pi](https://shittycodingagent.ai/) | [docs](https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/docs/skills.md) |
| 21 | [Databricks Genie Code](https://databricks.com/) | [docs](https://docs.databricks.com/aws/en/assistant/skills) |
| 22 | [Agentman](https://agentman.ai/) | [docs](https://agentman.ai/agentskills) |
| 23 | [TRAE](https://trae.ai/) | [docs](https://www.trae.ai/blog/trae_tutorial_0115) |
| 24 | [Spring AI](https://docs.spring.io/spring-ai/reference) | [docs](https://spring.io/blog/2026/01/13/spring-ai-generic-agent-skills/) |
| 25 | [Roo Code](https://roocode.com) | [docs](https://docs.roocode.com/features/skills) |
| 26 | [Mistral AI Vibe](https://github.com/mistralai/mistral-vibe) | [docs](https://github.com/mistralai/mistral-vibe) |
| 27 | [Command Code](https://commandcode.ai/) | [docs](https://commandcode.ai/docs/skills) |
| 28 | [Ona](https://ona.com) | [docs](https://ona.com/docs/ona/agents-md#skills-for-repository-specific-workflows) |
| 29 | [VT Code](https://github.com/vinhnx/vtcode) | [docs](https://github.com/vinhnx/vtcode/blob/main/docs/skills/SKILLS_GUIDE.md) |
| 30 | [Qodo](https://www.qodo.ai/) | [docs](https://www.qodo.ai/blog/how-i-use-qodos-agent-skills-to-auto-fix-issues-in-pull-requests/) |
| 31 | [Laravel Boost](https://github.com/laravel/boost) | [docs](https://laravel.com/docs/12.x/boost#agent-skills) |
| 32 | [Emdash](https://emdash.sh) | [docs](https://docs.emdash.sh/skills) |
| 33 | [Snowflake Cortex Code](https://docs.snowflake.com/en/user-guide/cortex-code/cortex-code) | [docs](https://docs.snowflake.com/en/user-guide/cortex-code/extensibility#extensibility-skills) |
| 34 | [Kiro](https://kiro.dev/) | [docs](https://kiro.dev/docs/skills/) |
| 35 | [Workshop](https://workshop.ai/) | [docs](https://docs.workshop.ai/core-concepts/working-with-the-agent#create-your-own-agents) |
| 36 | [Google AI Edge Gallery](https://github.com/google-ai-edge/gallery) | [docs](https://github.com/google-ai-edge/gallery/tree/main/skills) |
| 37 | [nanobot](https://nanobot.wiki/) | [docs](https://nanobot.wiki/docs/0.1.5/use-nanobot/skills) |
| 38 | [fast-agent](https://fast-agent.ai/) | [docs](https://fast-agent.ai/agents/skills/) |
| 39 | [bub](https://bub.build/) | [docs](https://bub.build/docs/build/skills/) |
| 40 | [Tabnine](https://www.tabnine.com/) | [docs](https://docs.tabnine.com/main/getting-started/tabnine-cli/features/agent-skills) |
| 41 | [Vita](https://www.vita-ai.net) | [docs](https://www.vita-ai.net/docs/features/agent-skills) |
| 42 | [Superconductor](https://superconductor.com/) | [docs](https://superconductor.com/docs/project/mcp-and-skills) |
| 43 | [Deep Code](https://deepcode.vegamo.cn/en) | [docs](https://deepcode.vegamo.cn/en/docs/configuration/agent-skills) |
| 44 | [Pulumi Neo](https://www.pulumi.com/product/neo/) | [docs](https://www.pulumi.com/docs/ai/skills/) |
| 45 | [Hermes Agent](https://hermes-agent.nousresearch.com/) | [docs](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills) |
| 46 | [OpenClaw](https://openclaw.ai/) | [docs](https://docs.openclaw.ai/tools/skills) |
