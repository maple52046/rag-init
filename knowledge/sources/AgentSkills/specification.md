# Agent Skills Specification

PURPOSE: 整理 Agent Skills 的格式規範：skill 目錄、`SKILL.md` frontmatter 欄位與限制、body、選用目錄、progressive disclosure、檔案引用與驗證，並記錄 `skills-ref` 驗證器依原始碼的實際行為。
APPLIES TO: 撰寫或檢查 `SKILL.md`、制定 skill 命名與 frontmatter 規則、設計 skill lint / CI 檢查。
關鍵字: SKILL.md, frontmatter, name, description, license, compatibility, metadata, allowed-tools, skills-ref validate, strictyaml.

> 來源：[Specification](https://agentskills.io/specification)，本地快照 [`source/specification.md`](source/specification.md)；驗證器行為依 [`skills-ref`](https://github.com/agentskills/agentskills/tree/83021e2be457a9b6679148719c1f6074f0ad975e/skills-ref) 原始碼（commit `83021e2`）。本文件為摘要，非官方譯文，以原文為準。標記 **[規範]** / **[建議]** 的定義見 [`README.md`](README.md#文件結構)。

---

## 目錄結構

skill 是一個至少含 `SKILL.md` 的目錄 **[規範]**：

```
skill-name/
├── SKILL.md          # 必要：metadata + 指示
├── scripts/          # 選用：可執行程式
├── references/       # 選用：文件
├── assets/           # 選用：範本、資源
└── ...               # 其他任意檔案或目錄
```

---

## `SKILL.md` 格式

`SKILL.md` 必須由 YAML frontmatter 開頭，後接 Markdown 內容 **[規範]**。

### Frontmatter 欄位

| 欄位 | 必要 | 限制 |
| --- | --- | --- |
| `name` | 是 | 最多 64 字元；只能是小寫字母、數字與 `-`；不可以 `-` 開頭或結尾 |
| `description` | 是 | 最多 1024 字元；不可為空；描述 skill 做什麼、何時使用 |
| `license` | 否 | 授權名稱，或指向 bundle 內授權檔的說明 |
| `compatibility` | 否 | 最多 500 字元；說明環境需求（目標產品、系統套件、網路存取等） |
| `metadata` | 否 | 任意 key-value 對應，用於額外 metadata（string key → string value） |
| `allowed-tools` | 否 | 以空白分隔的預先核准工具清單（Experimental） |

最小範例：

```markdown
---
name: skill-name
description: A description of what this skill does and when to use it.
---
```

含選用欄位的範例：

```markdown
---
name: pdf-processing
description: Extract PDF text, fill forms, merge files. Use when handling PDFs.
license: Apache-2.0
metadata:
  author: example-org
  version: "1.0"
---
```

### `name`

- **[規範]** 長度 1–64 字元。
- **[規範]** 只能含 unicode lowercase alphanumeric characters（頁面括號寫 `a-z`、`0-9`）與 `-`。原文同時寫了「unicode」與 `a-z` / `0-9`，字面有歧義；驗證器實際接受 Unicode 字母與數字，見下方驗證器一節。
- **[規範]** 不可以 `-` 開頭或結尾。
- **[規範]** 不可含連續的 `--`。
- **[規範]** 必須與上層目錄（skill 資料夾）名稱相同。

```yaml
name: pdf-processing      # 有效
name: data-analysis       # 有效
name: code-review         # 有效
name: PDF-Processing      # 無效：不可大寫
name: -pdf                # 無效：不可以 - 開頭
name: pdf--processing     # 無效：不可含連續 -
```

### `description`

- **[規範]** 長度 1–1024 字元（不可為空）。
- **[建議]** 同時描述 skill 做什麼與何時使用。
- **[建議]** 包含能幫助 agent 辨識相關任務的具體關鍵字。

```yaml
# 好
description: Extracts text and tables from PDF files, fills PDF forms, and merges multiple PDFs. Use when working with PDF documents or when the user mentions PDFs, forms, or document extraction.

# 差
description: Helps with PDFs.
```

description 的寫法與觸發測試見 [`optimizing-descriptions.md`](optimizing-descriptions.md)。

### `license`

- 指定套用於此 skill 的授權。
- **[建議]** 保持簡短：授權名稱，或 bundle 內授權檔的檔名。

```yaml
license: Proprietary. LICENSE.txt has complete terms
```

### `compatibility`

- **[規範]** 若提供，長度 1–500 字元。
- **[建議]** 只在 skill 有特定環境需求時才寫；可說明目標產品、所需系統套件、網路存取需求等。原文註明多數 skill 不需要此欄位。

```yaml
compatibility: Designed for Claude Code (or similar products)
compatibility: Requires git, docker, jq, and access to the internet
compatibility: Requires Python 3.14+ and uv
```

### `metadata`

- string key → string value 的 map。
- client 可用它存放 Agent Skills spec 未定義的額外屬性。
- **[建議]** key 名稱取得夠獨特，避免意外撞名。

```yaml
metadata:
  author: example-org
  version: "1.0"
```

### `allowed-tools`

- 以空白分隔、預先核准可執行的工具字串。
- Experimental：各 agent 實作對此欄位的支援程度不一。

```yaml
allowed-tools: Bash(git:*) Bash(jq:*) Read
```

### Body

frontmatter 之後的 Markdown body 是 skill 的指示，沒有格式限制，寫任何有助於 agent 有效完成任務的內容。

**[建議]** 段落：

- 逐步指示
- 輸入與輸出範例
- 常見 edge case

agent 一旦決定啟動 skill，會載入整份檔案；較長的 `SKILL.md` 內容應考慮拆到被引用的檔案。

---

## 選用目錄

`SKILL.md` 以外，skill 目錄可以含任何檔案與目錄。以下是組織常見內容的建議慣例 **[建議]**。

### `scripts/`

放 agent 可執行的程式碼。scripts 應：

- 自給自足，或清楚記載依賴。
- 提供有幫助的錯誤訊息。
- 妥善處理 edge case。

支援的語言取決於 agent 實作，常見有 Python、Bash、JavaScript。scripts 的設計細節見 [`using-scripts.md`](using-scripts.md)。

### `references/`

放 agent 需要時才讀的額外文件，例如：

- `REFERENCE.md`：詳細技術參考。
- `FORMS.md`：表單範本或結構化資料格式。
- 領域專屬檔案（`finance.md`、`legal.md` 等）。

每個 reference 檔保持聚焦；agent 按需載入，檔案越小越省 context。

### `assets/`

放靜態資源：

- 範本（文件範本、設定範本）
- 圖片（圖表、範例）
- 資料檔（lookup table、schema）

---

## Progressive disclosure

agent 漸進載入 skill，只在任務需要時才拉入更多細節。skill 應依此組織：

1. **Metadata**（約 100 tokens）：所有 skill 的 `name` 與 `description` 在啟動時載入。
2. **Instructions**（建議少於 5000 tokens）：skill 啟動時載入整份 `SKILL.md` body。
3. **Resources**（視需要）：`scripts/`、`references/`、`assets/` 等內的檔案只在需要時載入。

**[建議]** 主 `SKILL.md` 保持在 500 行以內，詳細參考資料移到獨立檔案。三層模型的整體說明見 [`README.md`](README.md#核心模型三層-progressive-disclosure)。

---

## 檔案引用

**[建議]** 引用 skill 內其他檔案時，使用相對於 skill 根目錄的路徑：

```markdown
See [the reference guide](references/REFERENCE.md) for details.

Run the extraction script:
scripts/extract.py
```

**[建議]** 檔案引用從 `SKILL.md` 起只深一層，避免深層巢狀的引用鏈。

---

## 驗證

用 [skills-ref](https://github.com/agentskills/agentskills/tree/main/skills-ref) 參考函式庫驗證 skill：

```bash
skills-ref validate ./my-skill
```

頁面說明它檢查 `SKILL.md` frontmatter 是否有效並符合所有命名慣例。

---

## skills-ref 驗證器實際行為（依原始碼）

本節依 commit `83021e2` 的 [`validator.py`](https://github.com/agentskills/agentskills/blob/83021e2be457a9b6679148719c1f6074f0ad975e/skills-ref/src/skills_ref/validator.py) 與 [`parser.py`](https://github.com/agentskills/agentskills/blob/83021e2be457a9b6679148719c1f6074f0ad975e/skills-ref/src/skills_ref/parser.py) 整理，並以該版本實際執行驗證過下列案例。這是比頁面文字更具體的行為描述，不是規範本身；`skills-ref` 標明僅供示範、非 production 用途，未來版本可能改變。

### 檢查流程

1. 路徑必須存在且為目錄；CLI 也接受直接指向 `SKILL.md` / `skill.md` 檔案的路徑（取其上層目錄）。
2. 依序找 `SKILL.md`、`skill.md`：優先大寫，但接受小寫 `skill.md`。兩者都沒有時回報 `Missing required file: SKILL.md`。
3. 解析 frontmatter（見下節），任何解析錯誤直接回報並停止。
4. 檢查欄位：頂層欄位白名單、`name`、`description`、`compatibility`。
5. 全部通過時 exit code 0 並輸出 `Valid skill: <path>`；否則錯誤列在 stderr，exit code 1。

### Frontmatter 解析

- 檔案內容必須以 `---` 這三個字元開頭（`content.startswith("---")`）。開頭有空行或 UTF-8 BOM 都會被拒絕：`SKILL.md must start with YAML frontmatter (---)`。
- 結束分隔是 `---` 在檔案中的下一次出現（`content.split("---", 2)`），不限定獨立一行。因此 frontmatter 值內出現 `---` 會讓 frontmatter 提前結束：`description: a --- b` 被解析成 `a`，且不報錯。
- 用 [strictyaml](https://hitchdev.com/strictyaml/) 解析，不帶 schema：
  - 所有 scalar 都解析為字串（`true`、`1.0`、`3` 都成為字串）。
  - 拒絕 flow style（`{a: b}`、`[a]`）、anchor / alias、重複 key。
  - 未加引號且含 `: ` 的值（例如 `description: Use this skill when: the user asks`）會解析失敗。用引號或 `>-` block scalar 可避免；block scalar 會被摺成單行。
- 頂層必須是 YAML mapping。
- `metadata` 若是 mapping，其 key 與 value 會被轉成字串。

### 欄位檢查

| 項目 | 頁面文字 | 驗證器實際行為 |
| --- | --- | --- |
| 頂層欄位 | 列出 6 個欄位 | 只允許 `name`、`description`、`license`、`allowed-tools`、`metadata`、`compatibility`；任何其他頂層 key 都報錯（`Unexpected fields in frontmatter: ...`）。client 專屬欄位（例如 client 實作指南提到的 `disable-model-invocation`）會讓驗證失敗 |
| `name` 字元集 | 小寫英數（`a-z`、`0-9`）與 `-` | 先 `strip()` 再做 Unicode NFKC 正規化；每個字元須通過 Python `str.isalnum()` 或為 `-`，且 `name == name.lower()`。因此接受 Unicode 小寫字母與沒有大小寫的文字（`café`、`技能` 皆通過），也接受非 ASCII 數字；大寫（含非拉丁大寫）、底線 `_`、空白會被拒絕 |
| `name` 長度 | 1–64 字元 | 以 NFKC 正規化後的字元數（code point）計 |
| `name` 與目錄 | 必須相同 | 目錄名也做 NFKC 正規化後比對；例如全形 `ｘ` 與目錄 `x` 視為相同 |
| `description` | 1–1024 字元、非空 | 必須為非空白字串；長度 ≤ 1024（code point）。不檢查內容是否寫了「何時使用」 |
| `compatibility` | 若提供則 1–500 字元 | 必須為字串且 ≤ 500；空字串可通過（不檢查下限） |
| `metadata` | string → string map | 不檢查型別；值為純字串（非 mapping）也會通過 |
| `license`、`allowed-tools` | 字串 | 完全不檢查（型別、格式都不驗） |

### 不檢查的項目

- body 內容（即使是空的）、行數、token 數。
- body 內引用的路徑是否存在、`scripts/` / `references/` / `assets/` 的內容。
- skill 目錄內是否有巢狀的 `SKILL.md`（`validate` 一次只驗一個目錄，不遞迴探索）。

### 其他 CLI 指令

- `skills-ref read-properties <path>`：解析 frontmatter 並輸出 JSON（`name`、`description` 會 `strip()`；另含 `license`、`compatibility`、`allowed-tools`、非空的 `metadata`）。不做完整驗證，只檢查 `name` 與 `description` 存在且非空。
- `skills-ref to-prompt <path>...`：產生 `<available_skills>` XML，每個 skill 含 `name`、`description`（HTML escape）與 `location`（`SKILL.md` 絕對路徑）。原始碼註解說明此 XML 格式是 Anthropic 為 Claude models 使用並推薦的格式，client 可依 model 自訂格式。catalog 格式的一般說明見 [`client-implementation.md`](client-implementation.md#step-3揭露可用-skill-給-model)。

套件資訊：`skills-ref` 0.1.0，需要 Python ≥ 3.11，依賴 `click` 與 `strictyaml`，Apache-2.0 授權。
PyPI 上的 `skills-ref` 0.1.1（2026-11 查證）提供的執行檔名稱是 `agentskills`，不是 `skills-ref`；不安裝直接執行用
`uvx --from skills-ref agentskills validate <dir>`。從上游原始碼安裝時，`pyproject.toml` 定義的指令仍是 `skills-ref`。
