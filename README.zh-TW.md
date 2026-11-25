# rag-init

給 AI coding agent 讀寫的知識庫起手包。適用於 RAG 背後的語料庫、放在 repository 裡的團隊 wiki，或由 agent
持續維護的筆記資料夾。

[English](README.md)

內容：

- **`create-knowledge-rules`**：一個 skill。它會訪談你的團隊怎麼產生和使用知識，再為你的 agent 寫出知識管理規則，
  涵蓋讀取、蒸餾、撰寫與更新。你只要描述情境，方法論由 agent 挑選。
- **`git-commit`**：一個可獨立運作的 skill，撰寫符合 Conventional Commits 的 commit 訊息，並安全地 commit。
- **來源封存區** [`knowledge/docs/`](knowledge/docs/README.md)：規則所依據的方法論來源，包括 Diátaxis、ADR、
  Zettelkasten、ISO 30401、Google 開發者文件風格指南、The Good Docs Project，以及 Agent Skills 標準。

兩個 skill 都遵循 [Agent Skills](https://agentskills.io/) 開放標準，可在 Cursor、Claude Code、OpenAI Codex、
Google Antigravity 與 Gemini CLI 使用。

## 快速開始

1. 把 skill 複製到你的專案（或使用者層級的 skills 目錄，見[各工具支援](#各工具支援)）：

   ```sh
   mkdir -p .agents/skills
   cp -R path/to/rag-init/.agents/skills/create-knowledge-rules .agents/skills/
   cp -R path/to/rag-init/.agents/skills/git-commit .agents/skills/
   ```

   使用 Claude Code 時，再把同樣的資料夾複製到 `.claude/skills/`。請複製 `.agents/skills/` 裡的實體資料夾，
   不要複製本 repository `.claude/skills/` 裡的薄 wrapper。
2. 對 agent 說「幫這個 repository 建立知識管理規則」，或執行 `/create-knowledge-rules`（Codex 用
   `$create-knowledge-rules`）。
3. 回答問題。每一題都附建議答案，可以直接採用。動筆之前，agent 會先給你一份情境摘要和預計採用的規則，請你確認。

你也可以直接用下列任一工具開啟本 repository，skill 會自動被探索到。

## `create-knowledge-rules` 會產出什麼

- **一份規則文件**，預設為 `docs/knowledge-governance.md`，涵蓋知識的讀取、蒸餾、撰寫與更新。每條規則都註明出自
  哪個方法論；文件最後列出被排除的規則、排除原因，以及什麼情況下值得加回來。
- **`AGENTS.md` 裡的常駐段落**，位於 `<!-- knowledge-rules:start -->` 與 `<!-- knowledge-rules:end -->` 之間，
  只放讀取規則和寫入前的檢查。agent 只有在寫入知識時才讀完整的規則文件。

訪談涵蓋：知識庫放哪類內容、誰會讀、怎麼用、知識從哪裡來、撰寫與回顧習慣、語言和存放位置、token 預算，以及
agent 該怎麼讀取知識。token 預算有三個等級：

| 等級 | 適用 | 常駐段落 | 規則文件 |
| --- | --- | --- | --- |
| 精簡（Lean） | 個人或小型知識庫 | ≤ 14 行、≤ 250 tokens | ≤ 80 行、≤ 1,200 tokens |
| 標準（Standard，預設） | 中等規模的團隊知識庫 | ≤ 22 行、≤ 400 tokens | ≤ 250 行、≤ 3,500 tokens |
| 完整（Full） | 多人維護或內容量大的知識庫 | ≤ 32 行、≤ 550 tokens | ≤ 600 行、≤ 9,000 tokens |

「完整」等級的完整產出範例在
[`assets/examples/`](.agents/skills/create-knowledge-rules/assets/examples/knowledge-governance.example.md)。
再次執行這個 skill 時，會更新既有規則，不會多出第二份。

## `git-commit` 做什麼

`/git-commit [--auto-add] [--all] [--push] [--date <when>]`（Codex 用 `$git-commit`）：沒帶 `--auto-add` 或
`--all` 時只 commit 你已 staged 的檔案；沒帶 `--push` 時不 push；疑似 secrets 的檔案不會被 commit；訊息最後附上
agent 的署名 trailer。同時提供 POSIX shell 和 PowerShell 的指令。詳見 [`SKILL.md`](.agents/skills/git-commit/SKILL.md)。

## 各工具支援

skill 以實體資料夾放在 `.agents/skills/`，只有一層、不使用 symbolic link，所以在 Linux、macOS 和 Windows 上
clone 下來的行為都一樣。

| 工具 | 本 repository 的 skills | 本 repository 的規則 | 在自己的專案使用 |
| --- | --- | --- | --- |
| [Cursor](https://cursor.com/docs/context/skills) | `.agents/skills/` | `AGENTS.md` | 複製到 `.agents/skills/` 或 `.cursor/skills/`；所有專案共用則放 `~/.cursor/skills/` |
| [Claude Code](https://code.claude.com/docs/en/skills) | `.claude/skills/`（薄 wrapper） | `CLAUDE.md`，內容匯入 `AGENTS.md` | 複製到 `.claude/skills/`；所有專案共用則放 `~/.claude/skills/` |
| [OpenAI Codex](https://developers.openai.com/codex/skills) | `.agents/skills/` | `AGENTS.md` | 複製到 `.agents/skills/`；所有專案共用則放 `~/.agents/skills/` |
| [Google Antigravity](https://antigravity.google/docs/skills/)（App 與 CLI） | `.agents/skills/` | `AGENTS.md` | 複製到 `.agents/skills/`；所有專案共用則放 `~/.gemini/config/skills/` |
| [Gemini CLI](https://geminicli.com/docs/cli/skills/) | `.agents/skills/` | 依下方設定後讀 `AGENTS.md` | 複製到 `.agents/skills/`；所有專案共用則放 `~/.agents/skills/` |

說明：

- **Cursor** 也會讀 `.claude/skills/`。同一個 skill 兩邊都有時只會列出一次（以 Cursor CLI 2026.11.05 確認）。
- **Claude Code** 不讀 `.agents/skills/`，所以本 repository 在 `.claude/skills/` 為每個 skill 放一個指向實體 skill
  的 wrapper。`CLAUDE.md` 的內容是 `@AGENTS.md`；這裡用匯入而不用 symbolic link，因為 Git for Windows 預設設定下
  clone 會讓 symbolic link 失效。
- **Antigravity** 只掃描 `.agents/skills/` 下一層，每個 skill 資料夾要直接放在它底下。
- **Gemini CLI** 預設讀 `GEMINI.md`。要讓它讀 `AGENTS.md`，在專案的 `.gemini/settings.json`（或
  `~/.gemini/settings.json`）加入：

  ```json
  {
    "context": {
      "fileName": ["AGENTS.md", "GEMINI.md"]
    }
  }
  ```

  2026 年 6 月 18 日起，Gemini CLI 不再服務 Google 個人帳號；Google 的後繼工具是
  [Antigravity CLI](https://antigravity.google/docs/cli/gcli-migration/)，它不需額外設定就會讀 `AGENTS.md` 和
  `.agents/skills/`。

## Repository 結構

```text
rag-init/
├── README.md, README.zh-TW.md
├── LICENSE                         # MIT，適用本 repository 自有內容
├── THIRD_PARTY_NOTICES.md          # 逐目錄列出上游授權
├── AGENTS.md                       # 本 repository 的維護規則
├── CLAUDE.md                       # @AGENTS.md
├── .agents/skills/
│   ├── create-knowledge-rules/     # SKILL.md、references/、assets/（範本與範例）
│   └── git-commit/                 # SKILL.md、references/conventional-commits-1.0.0.md
├── .claude/skills/<name>/SKILL.md  # 指向 .agents/skills/<name>/ 的 Claude Code wrapper
└── knowledge/docs/                 # 方法論來源封存區
```

## 授權

本 repository 自有內容採 MIT 授權（[`LICENSE`](LICENSE)）。來源封存區保留各自的上游授權，其中部分目錄是
CC BY-SA 4.0 而非 MIT，詳見 [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md)。Knowledge-Centered Service（KCS）
是方法論的一部分，但因授權不允許散布改作而沒有收錄，封存區改為連到原文。

`knowledge/docs/` 裡的筆記多為非官方的繁體中文摘要；上游快照維持原文語言。
