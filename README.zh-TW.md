# rag-init

給 AI coding agent 讀寫的知識庫起手包。適用於 RAG 背後的語料庫、放在 repository 裡的團隊 wiki，或由 agent
持續維護的筆記資料夾。

[English](README.md)

內容：

- **可直接使用的知識庫**：`AGENTS.md` 和 [`docs/knowledge-governance.md`](docs/knowledge-governance.md) 放了一套
  預設規則，規定 agent 怎麼讀取、蒸餾、撰寫與更新知識；[`knowledge/`](knowledge/README.md) 放了起步用的 index、
  taxonomy 和 glossary。
- **`create-knowledge-rules`**：一個 skill。它會訪談你的團隊怎麼產生和使用知識，再把上述規則改寫成適合你的版本。
  你只要描述情境，方法論由 agent 挑選。
- **`git-commit`**：一個可獨立運作的 skill，撰寫符合 Conventional Commits 的 commit 訊息，並安全地 commit。
- **方法論來源** [`knowledge/sources/`](knowledge/sources/README.md)：Diátaxis、ADR、Zettelkasten、ISO 30401、
  Google 開發者文件風格指南、The Good Docs Project 和 Agent Skills 標準，附閱讀筆記。

兩個 skill 都遵循 [Agent Skills](https://agentskills.io/) 開放標準，可在 Cursor、Claude Code、OpenAI Codex、
Google Antigravity 與 Gemini CLI 使用。

## 三種使用方式

### 1. 用這個 template 建立新的知識庫

在 GitHub 上按 **Use this template**，就會得到一個包含這些檔案、沒有 commit 歷史的新 repository，而且可以設成私有。
接著：

1. 用你的 agent 開啟新 repository，執行 `/create-knowledge-rules`（Codex 用 `$create-knowledge-rules`）。回答問題
   （每題都附建議答案），agent 會依你的情境改寫預設規則。你也可以先沿用預設規則，直接開始寫。
2. 把 `README.md` 和 `README.zh-TW.md` 換成你自己的內容，並刪除 `CONTRIBUTING.md`，那份是給 rag-init 本身用的。
3. `LICENSE` 只涵蓋 rag-init 自己的檔案。你自己的內容請自選授權；只要保留 `knowledge/sources/` 裡的資料夾，就要
   一併保留 `THIRD_PARTY_NOTICES.md` 和其中的授權檔。
4. 不需要方法論來源的話，刪除 `knowledge/sources/`（或其中不要的資料夾），並刪掉 `THIRD_PARTY_NOTICES.md` 裡對應的列。

從 template 建立的 repository 收不到 rag-init 之後的更新。需要更新的話，可以改用 fork（公開 repository 的 fork
會維持公開），或把 rag-init 加為第二個 remote 再 merge；也可以只用下面第二種方式更新 skill。

### 2. 把 skill 加進既有專案

把 skill 資料夾複製到你的專案，或複製到使用者層級的 skills 目錄讓所有專案共用（見[各工具支援](#各工具支援)）：

```sh
mkdir -p .agents/skills
cp -R path/to/rag-init/.agents/skills/create-knowledge-rules .agents/skills/
cp -R path/to/rag-init/.agents/skills/git-commit .agents/skills/
```

使用 Claude Code 時，再把同樣的資料夾複製到 `.claude/skills/`。請複製 `.agents/skills/` 裡的實體資料夾，不要複製
本 repository `.claude/skills/` 裡的薄 wrapper。接著對 agent 說「幫這個 repository 建立知識管理規則」，或執行
`/create-knowledge-rules`。

### 3. 用命令列工具起手

會在獨立的 repository `rag-init-cli` 提供一個命令列工具，負責鋪設專案並交棒給你的 agent，目前規劃中。

## `create-knowledge-rules` 會產出什麼

- **一份規則文件**，預設為 `docs/knowledge-governance.md`，涵蓋知識的讀取、蒸餾、撰寫與更新。每條規則都註明出自
  哪個方法論；文件最後列出被排除的規則、排除原因，以及什麼情況下值得加回來。
- **`AGENTS.md` 裡的常駐段落**，位於 `<!-- knowledge-rules:start -->` 與 `<!-- knowledge-rules:end -->` 之間，
  只放讀取規則和寫入前的檢查。agent 只有在寫入知識時才讀完整的規則文件。

訪談涵蓋：知識庫放哪類內容、誰會讀、怎麼用、知識從哪裡來、撰寫與回顧習慣、語言和存放位置、token 預算，以及
agent 該怎麼讀取知識。token 預算有三個等級，本 repository 的預設規則採「標準」：

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
├── README.md, README.zh-TW.md      # 介紹 rag-init；在你自己的 repository 請換掉
├── CONTRIBUTING.md                 # 修改 rag-init 本身時的規則
├── LICENSE                         # MIT，適用 rag-init 自有內容
├── THIRD_PARTY_NOTICES.md          # 逐目錄列出上游授權
├── AGENTS.md                       # 預設的常駐知識規則
├── CLAUDE.md                       # @AGENTS.md
├── docs/knowledge-governance.md    # 預設規則文件（標準等級）
├── knowledge/
│   ├── README.md, taxonomy.md, glossary.md
│   └── sources/                    # 方法論來源與閱讀筆記
├── .agents/skills/
│   ├── create-knowledge-rules/     # SKILL.md、references/、assets/（範本與範例）
│   └── git-commit/                 # SKILL.md、references/conventional-commits-1.0.0.md
└── .claude/skills/<name>/SKILL.md  # 指向 .agents/skills/<name>/ 的 Claude Code wrapper
```

## 授權

rag-init 自有內容採 MIT 授權（[`LICENSE`](LICENSE)）。方法論來源保留各自的上游授權，其中部分目錄是 CC BY-SA 4.0
而非 MIT，詳見 [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md)。Knowledge-Centered Service（KCS）是方法論的
一部分，但因授權不允許散布改作而沒有收錄，來源目錄改為連到原文。

`knowledge/sources/` 裡的筆記多為非官方的繁體中文摘要；上游快照維持原文語言。
