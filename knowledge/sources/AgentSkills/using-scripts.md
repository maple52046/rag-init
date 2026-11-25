# 在 Skill 中使用 Scripts

PURPOSE: 整理 agentskills.io 對 skill 內執行指令與 bundle scripts 的指引：一次性指令與 runner、從 `SKILL.md` 引用 scripts、自帶依賴的 scripts，以及為 agent 設計的 script 介面。
APPLIES TO: 在 skill 中寫指令或 `scripts/`、選擇 runner 與鎖版本、設計會被 agent 呼叫的 CLI 介面。
關鍵字: skill scripts, uvx, pipx, npx, bunx, deno run, go run, PEP 723, inline dependencies, non-interactive, --help, structured output, exit codes, --dry-run.

> 來源：[Using scripts in skills](https://agentskills.io/skill-creation/using-scripts)，本地快照 [`source/skill-creation/using-scripts.md`](source/skill-creation/using-scripts.md)。本文件為摘要，非官方譯文，以原文為準。除特別標示外，本頁內容屬 **[建議]**。

---

## 一次性指令

既有套件已能完成需要的工作時，可直接在 `SKILL.md` 指示中引用，不需要 `scripts/` 目錄。許多生態系提供在執行時自動解析依賴的工具：

| Runner | 範例 | 備註 |
| --- | --- | --- |
| [`uvx`](https://docs.astral.sh/uv/guides/tools/) | `uvx ruff@0.8.0 check .`<br>`uvx black@24.10.0 .` | 隨 uv 提供，不隨 Python 附帶，需另外安裝。快，積極快取，重複執行近乎即時 |
| [`pipx`](https://pipx.pypa.io/) | `pipx run 'black==24.10.0' .`<br>`pipx run 'ruff==0.8.0' check .` | 不隨 Python 附帶，需另外安裝；可從 OS 套件管理器取得（`apt install pipx`、`brew install pipx`）。成熟的替代方案；`uvx` 已成標準推薦，`pipx` 在 OS 套件管理器上更普及 |
| [`npx`](https://docs.npmjs.com/cli/commands/npx) | `npx eslint@9 --fix .`<br>`npx create-vite@6 my-app` | 隨 npm（隨 Node.js）附帶。按需下載、執行並快取。用 `npx package@version` 鎖版本 |
| [`bunx`](https://bun.sh/docs/cli/bunx) | `bunx eslint@9 --fix .`<br>`bunx create-vite@6 my-app` | 隨 Bun 附帶，在 Bun 環境可直接取代 `npx`；只適用於使用者環境是 Bun 而非 Node.js 時 |
| [`deno run`](https://docs.deno.com/runtime/reference/cli/run/) | `deno run npm:create-vite@6 my-app`<br>`deno run --allow-read npm:eslint@9 -- --fix .` | 隨 Deno 附帶。存取檔案系統 / 網路需要權限 flag（`--allow-read` 等）；用 `--` 分隔 Deno flag 與工具自己的 flag |
| [`go run`](https://pkg.go.dev/cmd/go#hdr-Compile_and_run_Go_program) | `go run golang.org/x/tools/cmd/goimports@v0.28.0 .`<br>`go run github.com/golangci/golangci-lint/cmd/golangci-lint@v1.62.0 run` | 內建於 `go` 指令。鎖版本或用 `@latest` 讓指令明確 |

在 skill 中使用一次性指令的要點：

- **鎖版本**（例如 `npx eslint@9.0.0`），讓指令長期行為一致。
- **寫明前置需求**：在 `SKILL.md` 寫「Requires Node.js 18+」之類的需求，不要假設 agent 環境已具備。runtime 等級的需求使用 frontmatter 的 [`compatibility`](specification.md#compatibility) 欄位。
- **複雜指令改成 script**：帶幾個 flag 呼叫工具時一次性指令很好用；指令複雜到難以一次寫對時，`scripts/` 內測過的 script 更可靠。

---

## 從 `SKILL.md` 引用 scripts

引用 bundle 內檔案時，使用**相對於 skill 目錄根的路徑**；agent 會自動解析，不需要絕對路徑。先在 `SKILL.md` 列出可用的 scripts，讓 agent 知道它們存在：

```markdown
## Available scripts

- **`scripts/validate.sh`** — Validates configuration files
- **`scripts/process.py`** — Processes input data
```

再指示 agent 執行：

````markdown
## Workflow

1. Run the validation script:
   ```bash
   bash scripts/validate.sh "$INPUT_FILE"
   ```

2. Process the results:
   ```bash
   python3 scripts/process.py --input results.json
   ```
````

同一相對路徑慣例也適用於 `references/*.md` 等支援檔：code block 內的 script 執行路徑相對於 **skill 目錄根**，因為 agent 從那裡執行指令。

---

## 自帶依賴的 scripts

需要可重用邏輯時，在 `scripts/` 放一個以 inline 方式宣告自身依賴的 script，agent 一行指令就能執行，不需要另外的 manifest 檔或安裝步驟。

### Python（PEP 723）

[PEP 723](https://peps.python.org/pep-0723/) 定義 inline script metadata 的標準格式，在 `# ///` 標記內以 TOML 宣告依賴：

```python
# /// script
# dependencies = [
#   "beautifulsoup4",
# ]
# ///

from bs4 import BeautifulSoup

html = '<html><body><h1>Welcome</h1><p class="info">This is a test.</p></body></html>'
print(BeautifulSoup(html, "html.parser").select_one("p.info").get_text())
```

以 [uv](https://docs.astral.sh/uv/) 執行（推薦）：`uv run scripts/extract.py`。`uv run` 建立隔離環境、安裝宣告的依賴並執行 script；`pipx run scripts/extract.py` 也支援 PEP 723。

- 用 [PEP 508](https://peps.python.org/pep-0508/) specifier 鎖版本：`"beautifulsoup4>=4.12,<5"`。
- 用 `requires-python` 限制 Python 版本。
- 用 `uv lock --script` 建立 lockfile 以完全重現。

### Deno

Deno 的 `npm:` 與 `jsr:` import specifier 讓每個 script 預設自給自足：

```typescript
#!/usr/bin/env -S deno run

import * as cheerio from "npm:cheerio@1.0.0";

const html = `<html><body><h1>Welcome</h1><p class="info">This is a test.</p></body></html>`;
const $ = cheerio.load(html);
console.log($("p.info").text());
```

執行：`deno run scripts/extract.ts`。

- npm 套件用 `npm:`，Deno 原生套件用 `jsr:`。
- 版本 specifier 依 semver：`@1.0.0`（精確）、`@^1.0.0`（相容）。
- 依賴全域快取；用 `--reload` 強制重新抓取。
- 帶 native addon（node-gyp）的套件可能無法運作；附預建 binary 的套件最合適。

### Bun

找不到 `node_modules` 目錄時，Bun 會在執行時自動安裝缺少的套件。在 import 路徑直接鎖版本：

```typescript
#!/usr/bin/env bun

import * as cheerio from "cheerio@1.0.0";

const html = `<html><body><h1>Welcome</h1><p class="info">This is a test.</p></body></html>`;
const $ = cheerio.load(html);
console.log($("p.info").text());
```

執行：`bun run scripts/extract.ts`。

- 不需要 `package.json` 或 `node_modules`；原生支援 TypeScript。
- 套件全域快取；第一次執行下載，之後近乎即時。
- 若目錄樹往上任何一層存在 `node_modules`，自動安裝會停用，Bun 退回標準 Node.js 解析。

### Ruby

Ruby 2.6 起內建 Bundler。用 `bundler/inline` 直接在 script 內宣告 gem：

```ruby
require 'bundler/inline'

gemfile do
  source 'https://rubygems.org'
  gem 'nokogiri'
end

html = '<html><body><h1>Welcome</h1><p class="info">This is a test.</p></body></html>'
doc = Nokogiri::HTML(html)
puts doc.at_css('p.info').text
```

執行：`ruby scripts/extract.rb`。

- 明確鎖版本（`gem 'nokogiri', '~> 1.16'`），因為沒有 lockfile。
- 工作目錄中既有的 `Gemfile` 或 `BUNDLE_GEMFILE` 環境變數可能造成干擾。

---

## 為 agent 使用設計 scripts

agent 執行 script 時，讀 stdout 與 stderr 來決定下一步。幾個設計選擇能讓 scripts 對 agent 好用得多。

### 避免互動式提示（硬性要求）

這是 agent 執行環境的硬性限制：agent 在非互動 shell 中運作，無法回應 TTY prompt、密碼對話框或確認選單。等待互動輸入的 script 會無限期卡住。所有輸入改走命令列 flag、環境變數或 stdin：

```
# Bad: hangs waiting for input
$ python scripts/deploy.py
Target environment: _

# Good: clear error with guidance
$ python scripts/deploy.py
Error: --env is required. Options: development, staging, production.
Usage: python scripts/deploy.py --env staging --tag v1.2.3
```

### 以 `--help` 記載用法

`--help` 輸出是 agent 學習 script 介面的主要途徑。包含簡短描述、可用 flag 與用法範例：

```
Usage: scripts/process.py [OPTIONS] INPUT_FILE

Process input data and produce a summary report.

Options:
  --format FORMAT    Output format: json, csv, table (default: json)
  --output FILE      Write output to FILE instead of stdout
  --verbose          Print progress to stderr

Examples:
  scripts/process.py data.csv
  scripts/process.py --format csv --output report.csv data.csv
```

保持簡短：這段輸出會和 agent 正在處理的其他內容一起進入 context window。

### 寫有幫助的錯誤訊息

agent 收到錯誤時，訊息直接決定它下一次怎麼試。不透明的「Error: invalid input」浪費一輪。要說明哪裡錯、預期什麼、該試什麼：

```
Error: --format must be one of: json, csv, table.
       Received: "xml"
```

### 使用結構化輸出

優先用結構化格式（JSON、CSV、TSV）而非自由文字。結構化格式可同時被 agent 與標準工具（`jq`、`cut`、`awk`）使用，讓 script 能組合進 pipeline：

```
# Whitespace-aligned — hard to parse programmatically
NAME          STATUS    CREATED
my-service    running   2025-01-15

# Delimited — unambiguous field boundaries
{"name": "my-service", "status": "running", "created": "2025-01-15"}
```

**分離資料與診斷**：結構化資料送 stdout，進度訊息、警告與其他診斷送 stderr。agent 可擷取乾淨可解析的輸出，同時在需要時取得診斷資訊。

### 其他考量

- **Idempotency**：agent 可能重試指令。「不存在才建立」比「建立、重複就失敗」安全。
- **輸入約束**：遇到模稜兩可的輸入，以清楚錯誤拒絕，不要猜。盡量使用 enum 與封閉集合。
- **Dry-run 支援**：破壞性或有狀態的操作提供 `--dry-run`，讓 agent 預覽將發生什麼。
- **有意義的 exit code**：不同失敗類型（找不到、參數錯誤、認證失敗）用不同 exit code，並寫進 `--help`，讓 agent 知道每個碼的意思。
- **安全預設**：考慮破壞性操作是否應要求明確確認 flag（`--confirm`、`--force`）或其他符合風險等級的防護。
- **可預期的輸出大小**：許多 agent harness 會自動截斷超過門檻（例如 10–30K 字元）的 tool 輸出，可能遺失關鍵資訊。若 script 可能產生大量輸出，預設輸出摘要或合理上限，並支援 `--offset` 之類的 flag 讓 agent 需要時再要更多；若輸出很大又不適合分頁，要求 agent 傳 `--output` 指定輸出檔，或傳 `-` 明確選擇輸出到 stdout。

何時該把重複邏輯寫成 bundle script，見 [`best-practices.md`](best-practices.md#bundle-可重用的-scripts)。
