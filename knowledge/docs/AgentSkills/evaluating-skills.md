# 評估 Skill 輸出品質

PURPOSE: 整理 agentskills.io 以 eval 驅動迭代來測試 skill 輸出品質的方法：測試案例、with / without skill 對照執行、assertions、評分、彙整、模式分析、人工審查與迭代迴圈。
APPLIES TO: 為 skill 建立輸出品質 eval、比較 skill 新舊版本、判斷 skill 是否真的增加價值。
關鍵字: skill evals, evals.json, with_skill, without_skill, baseline, assertions, grading.json, benchmark.json, feedback.json, blind comparison, iteration.

> 來源：[Evaluating skill output quality](https://agentskills.io/skill-creation/evaluating-skills)，本地快照 [`source/skill-creation/evaluating-skills.md`](source/skill-creation/evaluating-skills.md)。本文件為摘要，非官方譯文，以原文為準。本頁內容全屬 **[建議]**；檔名與 JSON 結構是原文示範的慣例，不是 specification 的一部分。

---

## 為什麼需要 eval

skill 寫好、試了一個 prompt、看起來能用；但它是否可靠——跨不同 prompt、在 edge case、比沒有 skill 更好？結構化 eval 回答這些問題，並提供系統化改善 skill 的回饋迴圈。觸發面的測試另見 [`optimizing-descriptions.md`](optimizing-descriptions.md)；本頁測的是 skill 啟動後的輸出。

---

## 設計測試案例

一個測試案例有三部分：

- **Prompt**：真實感的使用者訊息，真的會有人這樣打。
- **Expected output**：以人類可讀的方式描述成功長什麼樣。
- **Input files**（選用）：skill 需要處理的檔案。

存在 skill 目錄內的 `evals/evals.json`：

```json
{
  "skill_name": "csv-analyzer",
  "evals": [
    {
      "id": 1,
      "prompt": "I have a CSV of monthly sales data in data/sales_2025.csv. Can you find the top 3 months by revenue and make a bar chart?",
      "expected_output": "A bar chart image showing the top 3 months by revenue, with labeled axes and values.",
      "files": ["evals/files/sales_2025.csv"]
    },
    {
      "id": 2,
      "prompt": "there's a csv in my downloads called customers.csv, some rows have missing emails — can you clean it up and tell me how many were missing?",
      "expected_output": "A cleaned CSV with missing emails handled, plus a count of how many were missing.",
      "files": ["evals/files/customers.csv"]
    }
  ]
}
```

撰寫測試 prompt 的要點：

- **先寫 2–3 個案例**：看到第一輪結果前不要過度投入，之後再擴充。
- **讓 prompt 有變化**：不同措辭、細節量與正式程度；有的口語（「hey can you clean up this csv」），有的精確（「Parse the CSV at data/input.csv, drop rows where column B is null, and write the result to data/output.csv」）。
- **涵蓋 edge case**：至少一個測邊界條件的 prompt：壞掉的輸入、不尋常的請求，或 skill 指示可能模稜兩可的情境。
- **用真實脈絡**：真實使用者會提到檔案路徑、欄位名與個人脈絡；「process this data」太模糊，測不出有用的東西。

此時先不要定義具體的 pass / fail 檢查，只寫 prompt 與 expected output；詳細檢查（assertions）等看過第一輪輸出再加。

---

## 執行 eval

核心模式：每個測試案例跑兩次，一次 **with skill**、一次 **without skill**（或用前一版 skill），以取得對照 baseline。

### 工作區結構

eval 結果放在與 skill 目錄並列的 workspace 目錄。每跑完整個 eval 迴圈一次就有自己的 `iteration-N/`；其中每個測試案例一個 eval 目錄，內含 `with_skill/` 與 `without_skill/`：

```
csv-analyzer/
├── SKILL.md
└── evals/
    └── evals.json
csv-analyzer-workspace/
└── iteration-1/
    ├── eval-top-months-chart/
    │   ├── with_skill/
    │   │   ├── outputs/       # Files produced by the run
    │   │   ├── timing.json    # Tokens and duration
    │   │   └── grading.json   # Assertion results
    │   └── without_skill/
    │       ├── outputs/
    │       ├── timing.json
    │       └── grading.json
    ├── eval-clean-missing-emails/
    │   ├── with_skill/
    │   │   ├── outputs/
    │   │   ├── timing.json
    │   │   └── grading.json
    │   └── without_skill/
    │       ├── outputs/
    │       ├── timing.json
    │       └── grading.json
    └── benchmark.json         # Aggregated statistics
```

主要手寫的檔案是 `evals/evals.json`；其他 JSON（`grading.json`、`timing.json`、`benchmark.json`）在 eval 過程中由 agent、scripts 或你產生。

### 啟動各次執行

每次 eval run 都從乾淨的 context 開始，不帶前次執行或 skill 開發過程的殘留狀態，確保 agent 只依 `SKILL.md` 行事。支援 subagent 的環境（例如 Claude Code）天然有這種隔離：每個子任務從頭開始；沒有 subagent 時，每次 run 用獨立 session。

每次 run 提供：skill 路徑（baseline 則不提供）、測試 prompt、輸入檔、輸出目錄。單次 with-skill run 的指示範例：

```
Execute this task:
- Skill path: /path/to/csv-analyzer
- Task: I have a CSV of monthly sales data in data/sales_2025.csv.
  Can you find the top 3 months by revenue and make a bar chart?
- Input files: evals/files/sales_2025.csv
- Save outputs to: csv-analyzer-workspace/iteration-1/eval-top-months-chart/with_skill/outputs/
```

baseline 用同一 prompt、不給 skill 路徑，輸出到 `without_skill/outputs/`。

改進既有 skill 時，用前一版當 baseline：編輯前先 snapshot（`cp -r <skill-path> <workspace>/skill-snapshot/`），baseline run 指向 snapshot，輸出到 `old_skill/outputs/` 而非 `without_skill/`。

### 記錄 timing

timing 資料讓你比較 skill 相對 baseline 花了多少時間與 tokens：大幅提升品質但 token 用量變三倍的 skill，和又好又省的 skill 是不同的取捨。每次 run 完成時記錄 token 數與時長：

```json
{
  "total_tokens": 84852,
  "duration_ms": 23332
}
```

在 Claude Code 中，subagent 任務結束時的 task completion notification 含 `total_tokens` 與 `duration_ms`；要立即存下，它們不會被保存在其他地方。

---

## 撰寫 assertions

assertion 是對輸出應包含或達成什麼的可驗證陳述。看過第一輪輸出後再加；skill 跑過之前，你往往不知道「好」長什麼樣。

- 好的 assertion：`"The output file is valid JSON"`（可程式驗證）、`"The bar chart has labeled axes"`（具體可觀察）、`"The report includes at least 3 recommendations"`（可計數）。
- 弱的 assertion：`"The output is good"`（太模糊無法評分）、`"The output uses exactly the phrase 'Total Revenue: $X'"`（太脆，措辭不同的正確輸出會失敗）。

不是每件事都需要 assertion。文風、視覺設計、輸出「感覺對不對」很難拆成 pass / fail，留給人工審查；assertion 只用於能客觀檢查的事。在 `evals/evals.json` 各案例加上 assertions：

```json
{
  "skill_name": "csv-analyzer",
  "evals": [
    {
      "id": 1,
      "prompt": "I have a CSV of monthly sales data in data/sales_2025.csv. Can you find the top 3 months by revenue and make a bar chart?",
      "expected_output": "A bar chart image showing the top 3 months by revenue, with labeled axes and values.",
      "files": ["evals/files/sales_2025.csv"],
      "assertions": [
        "The output includes a bar chart image file",
        "The chart shows exactly 3 months",
        "Both axes are labeled",
        "The chart title or caption mentions revenue"
      ]
    }
  ]
}
```

---

## 評分

評分是拿每個 assertion 對照實際輸出，記錄 **PASS** 或 **FAIL** 並附具體證據；證據要引用或指向輸出內容，不只是陳述意見。

最簡單的做法是把輸出與 assertions 交給 LLM 逐一評估。能用程式檢查的 assertion（合法 JSON、列數正確、檔案存在且尺寸符合）用驗證 script：機械檢查上 script 比 LLM 判斷可靠，而且可跨輪重用。

```json
{
  "assertion_results": [
    {
      "text": "The output includes a bar chart image file",
      "passed": true,
      "evidence": "Found chart.png (45KB) in outputs directory"
    },
    {
      "text": "The chart shows exactly 3 months",
      "passed": true,
      "evidence": "Chart displays bars for March, July, and November"
    },
    {
      "text": "Both axes are labeled",
      "passed": false,
      "evidence": "Y-axis is labeled 'Revenue ($)' but X-axis has no label"
    },
    {
      "text": "The chart title or caption mentions revenue",
      "passed": true,
      "evidence": "Chart title reads 'Top 3 Months by Revenue'"
    }
  ],
  "summary": {
    "passed": 3,
    "failed": 1,
    "total": 4,
    "pass_rate": 0.75
  }
}
```

（以上為 `grading.json`。）評分原則：

- **PASS 需要具體證據**，不要給模糊空間。assertion 要求「includes a summary」，而輸出只有標題為「Summary」的一句空話，就是 FAIL：標籤在，實質不在。
- **也要審視 assertion 本身**：評分時留意太容易（不論 skill 品質都過）、太難（輸出很好也不過）或無法驗證（光看輸出無法檢查）的 assertion，下一輪修掉。

比較兩個 skill 版本時可用**盲測**：把兩份輸出交給不知道來源的 LLM judge，依它自己的評分標準評整體品質（組織、格式、可用性、完成度），避免偏向「應該比較好」的版本。這補足 assertion 評分：兩份輸出可能都通過所有 assertion，整體品質卻差很多。

---

## 彙整結果

一輪所有 run 評分完後，依組態計算統計並存成 `benchmark.json`，與 eval 目錄放在一起（例如 `csv-analyzer-workspace/iteration-1/benchmark.json`）：

```json
{
  "run_summary": {
    "with_skill": {
      "pass_rate": { "mean": 0.83, "stddev": 0.06 },
      "time_seconds": { "mean": 45.0, "stddev": 12.0 },
      "tokens": { "mean": 3800, "stddev": 400 }
    },
    "without_skill": {
      "pass_rate": { "mean": 0.33, "stddev": 0.10 },
      "time_seconds": { "mean": 32.0, "stddev": 8.0 },
      "tokens": { "mean": 2100, "stddev": 300 }
    },
    "delta": {
      "pass_rate": 0.50,
      "time_seconds": 13.0,
      "tokens": 1700
    }
  }
}
```

`delta` 同時告訴你 skill 的代價（更多時間、更多 tokens）與收益（更高 pass rate）。多 13 秒換 50 個百分點的 pass rate 大概值得；token 用量加倍只換 2 個百分點則可能不值得。

`stddev` 只有在每個 eval 多次執行時才有意義。初期只有 2–3 個案例且各跑一次時，看原始通過數與 delta；擴充測試集並多次執行後，統計量才開始有用。

---

## 分析模式

總體統計可能掩蓋重要模式。算完 benchmark 後：

- **移除或替換兩種組態都必過的 assertion**：它們不提供資訊（model 沒有 skill 也做得到），只會灌高 with-skill pass rate。
- **調查兩種組態都必敗的 assertion**：可能 assertion 壞了（要求 model 做不到的事）、案例太難、或檢查錯對象；下一輪前修掉。
- **研究有 skill 才過、沒有就失敗的 assertion**：這是 skill 明確增加價值之處，要弄清楚**為什麼**：是哪些指示或 scripts 起了作用？
- **結果在多次執行間不一致時收緊指示**：同一 eval 時過時不過（benchmark 上 `stddev` 高），可能 eval 本身不穩（對 model 隨機性敏感），或 skill 指示模稜兩可到 model 每次解讀不同。加範例或更具體的指引來減少歧義。
- **檢查時間與 token 的離群值**：某個 eval 花了其他的 3 倍時間，就讀它的執行 transcript（model 在該次 run 的完整紀錄）找瓶頸。

---

## 人工審查

assertion 評分與模式分析能抓到很多問題，但只檢查你想得到要寫的 assertion。人工 reviewer 帶來新視角：抓到沒預料到的問題、發現技術上正確卻沒抓到重點的輸出、或難以表達成 pass / fail 的問題。每個案例都把實際輸出與評分一起看。

每個案例記錄具體回饋，存在 workspace（例如與 eval 目錄並列的 `feedback.json`）：

```json
{
  "eval-top-months-chart": "The chart is missing axis labels and the months are in alphabetical order instead of chronological.",
  "eval-clean-missing-emails": ""
}
```

「The chart is missing axis labels」可行動；「looks bad」不可行動。空字串代表輸出沒問題，該案例通過審查。迭代時聚焦在有具體意見的案例。

---

## 迭代 skill

評分與審查後有三種訊號：

- **失敗的 assertion**：指出具體缺口，例如缺步驟、指示不清、skill 沒處理到的情況。
- **人工回饋**：指出較廣的品質問題，例如做法錯誤、輸出結構不佳、技術上正確但沒幫助。
- **執行 transcript**：揭露**為什麼**出錯。agent 忽略某條指示，可能是指示模稜兩可；agent 在無效步驟上花時間，那些指示可能需要簡化或移除。

把三種訊號連同目前的 `SKILL.md` 交給 LLM 請它提出修改，最有效率；LLM 能綜合失敗 assertion、reviewer 意見與 transcript 行為間的模式。prompt 時附上這些準則：

- **從回饋泛化**：skill 會用在許多不同 prompt 上，不只測試案例；修正要廣泛處理根本問題，不要為特定範例加窄補丁。
- **保持精簡**：較少、較好的指示常勝過窮舉規則。transcript 顯示白工（不必要的驗證、不需要的中間輸出）就移除那些指示；加了更多規則 pass rate 仍停滯，skill 可能過度約束，試著移除指示看結果是否持平或變好。
- **解釋原因**：基於理由的指示（「Do X because Y tends to cause Z」）比僵硬指令（「ALWAYS do X, NEVER do Y」）有效；model 理解目的時更可靠地遵循。
- **Bundle 重複工作**：每次測試 run 都各自寫了類似的 helper script（圖表產生器、資料 parser），就把 script 放進 skill 的 `scripts/`；見 [`using-scripts.md`](using-scripts.md)。

### 迴圈

1. 把 eval 訊號與目前的 `SKILL.md` 交給 LLM，請它提出改進。
2. 審查並套用修改。
3. 在新的 `iteration-<N+1>/` 目錄重跑所有測試案例。
4. 評分並彙整新結果。
5. 人工審查。重複。

停止條件：對結果滿意、人工回饋持續為空、或各輪之間不再有明顯進步。

[`skill-creator`](https://github.com/anthropics/skills/tree/main/skills/skill-creator) skill 自動化了此流程的大部分：執行 eval、評分 assertion、彙整 benchmark，並呈現結果供人工審查。
