# 優化 Skill Description

PURPOSE: 整理 agentskills.io 對 `description` 的寫法原則與觸發率測試方法：觸發機制、eval query 設計、多次執行與 trigger rate、train / validation 切分與優化迴圈。
APPLIES TO: 撰寫或改寫 skill 的 `description`、診斷 skill 不觸發或誤觸發、建立觸發 eval。
關鍵字: skill description, triggering, trigger rate, eval queries, should_trigger, near-miss, train/validation split, overfitting.

> 來源：[Optimizing skill descriptions](https://agentskills.io/skill-creation/optimizing-descriptions)，本地快照 [`source/skill-creation/optimizing-descriptions.md`](source/skill-creation/optimizing-descriptions.md)。本文件為摘要，非官方譯文，以原文為準。`description` 的硬性限制見 [`specification.md`](specification.md#description)；本頁其餘內容屬 **[建議]**。

---

## 觸發如何運作

skill 只有被啟動才有用。`SKILL.md` frontmatter 的 `description` 是 agent 決定是否為某任務載入 skill 的主要機制：描述不足就該觸發時不觸發；描述過寬就不該觸發時觸發。

agent 啟動時只載入每個 skill 的 `name` 與 `description`；使用者任務符合某 description 時，才把整份 `SKILL.md` 讀進 context 並依其指示行事（三層模型見 [`README.md`](README.md#核心模型三層-progressive-disclosure)）。因此 **description 承擔觸發的全部責任**：description 沒傳達 skill 何時有用，agent 就不知道要用它。

重要細節：agent 通常只在任務需要超出自身能力的知識或能力時才查 skill。「read this PDF」這種簡單的一步請求，即使 description 完全符合也可能不觸發 PDF skill，因為 agent 用基本工具就能處理。涉及專門知識的任務（不熟的 API、領域專屬流程、不常見的格式）才是好 description 能決定成敗的地方。

---

## 撰寫有效的 description

- **祈使語氣**：把 description 寫成給 agent 的指示：「Use this skill when...」而不是「This skill does...」。agent 在決定是否行動，就告訴它何時行動。
- **聚焦使用者意圖，不寫實作**：描述使用者想達成什麼，不描述 skill 的內部機制；agent 是拿使用者的請求去比對。
- **寧可積極（pushy）**：明列 skill 適用的情境，包括使用者沒直接點名領域的情況：「even if they don't explicitly mention 'CSV' or 'analysis.'」
- **保持精簡**：通常幾句話到一小段，足以涵蓋範圍，又不會在多個 skill 間膨脹 context。specification 硬性上限為 1024 字元。

---

## 設計觸發 eval query

測試觸發需要一組 eval query：真實感的使用者 prompt，並標註應不應該觸發 skill。

```json
[
  { "query": "I've got a spreadsheet in ~/data/q4_results.xlsx with revenue in col C and expenses in col D — can you add a profit margin column and highlight anything under 10%?", "should_trigger": true },
  { "query": "whats the quickest way to convert this json file to yaml", "should_trigger": false }
]
```

（原文檔名為 `eval_queries.json`。）目標約 20 筆：8–10 筆應觸發、8–10 筆不應觸發。

### 應觸發的 query

測試 description 是否涵蓋 skill 的範圍。沿以下面向變化：

- **措辭**：正式、口語、含錯字或縮寫。
- **明確度**：有的直接點名領域（「analyze this CSV」），有的只描述需求（「my boss wants a chart from this data file」）。
- **細節量**：簡短 prompt（「analyze my sales CSV and make a chart」）混合帶檔案路徑、欄位名與背景的長訊息。
- **複雜度**：改變步驟數與決策點；單步任務與多步工作流程並列，測試相關任務被埋在較長流程中時 agent 能否辨識 skill 適用。

最有用的應觸發 query 是「skill 會有幫助、但光看 query 看不出關聯」的案例。這正是 description 措辭決定成敗之處；若 query 已直接要求 skill 所做的事，任何合理的 description 都會觸發。

### 不應觸發的 query

最有價值的反例是 **near-miss**：和 skill 共享關鍵字或概念，但實際需要別的東西。它們測的是 description 是否精確，而不只是夠寬。以 CSV 分析 skill 為例：

- 弱反例：`"Write a fibonacci function"`（明顯無關，測不出東西）、`"What's the weather today?"`（沒有關鍵字重疊，太容易）。
- 強反例：`"I need to update the formulas in my Excel budget spreadsheet"`（共享 spreadsheet 與 data 概念，但需要的是 Excel 編輯而非 CSV 分析）、`"can you write a python script that reads a csv and uploads each row to our postgres database"`（涉及 CSV，但任務是 database ETL 而非分析）。

### 真實感

真實 prompt 帶有通用測試 query 缺少的脈絡。加入：

- 檔案路徑（`~/Downloads/report_final_v2.xlsx`）
- 個人脈絡（`"my manager asked me to..."`）
- 具體細節（欄位名、公司名、資料值）
- 口語、縮寫與偶發錯字

---

## 測試 description 是否觸發

基本做法：在裝有 skill 的 agent 上逐筆執行 query，觀察 agent 是否調用 skill。先確認 skill 已註冊且可被探索；做法因 client 而異（skills 目錄、設定檔或 CLI flag）。多數 client 有某種可觀測性（執行 log、tool call 歷史、verbose 輸出）可看出 run 中查了哪些 skill。**判定依據：agent 載入了該 skill 的 `SKILL.md` 即為觸發**；沒查就直接做事即為未觸發。

query 判定為 pass 的條件：

- `should_trigger` 為 `true` 且 skill 被調用；或
- `should_trigger` 為 `false` 且 skill 未被調用。

### 多次執行

model 行為不具確定性，同一 query 可能這次觸發、下次不觸發。每筆 query 跑多次（3 次是合理起點），計算 **trigger rate**：skill 被調用的執行比例。應觸發 query 的 trigger rate 高於門檻才算 pass（0.5 是合理預設）；不應觸發 query 低於門檻才算 pass。

20 筆 × 3 次 = 60 次調用，需要寫成腳本。原文的通用結構如下；`check_triggered` 內的 `claude` 調用與偵測邏輯要換成所用 client 提供的方式（此例以 Claude Code 的 JSON 輸出偵測 `Skill` tool call）：

```bash
#!/bin/bash
QUERIES_FILE="${1:?Usage: $0 <queries.json>}"
SKILL_NAME="my-skill"
RUNS=3

# This example uses Claude Code's JSON output to check for Skill tool calls.
# Replace this function with detection logic for your agent client.
# Should return 0 (success) if the skill was invoked, 1 otherwise.
check_triggered() {
  local query="$1"
  claude -p "$query" --output-format json 2>/dev/null \
    | jq -e --arg skill "$SKILL_NAME" \
      'any(.messages[].content[]; .type == "tool_use" and .name == "Skill" and .input.skill == $skill)' \
      > /dev/null 2>&1
}

count=$(jq length "$QUERIES_FILE")
for i in $(seq 0 $((count - 1))); do
  query=$(jq -r ".[$i].query" "$QUERIES_FILE")
  should_trigger=$(jq -r ".[$i].should_trigger" "$QUERIES_FILE")
  triggers=0

  for run in $(seq 1 $RUNS); do
    check_triggered "$query" && triggers=$((triggers + 1))
  done

  jq -n \
    --arg query "$query" \
    --argjson should_trigger "$should_trigger" \
    --argjson triggers "$triggers" \
    --argjson runs "$RUNS" \
    '{query: $query, should_trigger: $should_trigger, triggers: $triggers, runs: $runs, trigger_rate: ($triggers / $runs)}'
done | jq -s '.'
```

若 client 支援，可在結果明朗時提早中止 run（agent 已查了 skill，或已開始不用 skill 做事），能大幅降低跑完整組 eval 的時間與成本。

---

## 以 train / validation 切分避免過擬合

若對全部 query 優化 description，容易過擬合：description 對這些特定措辭有效，換新措辭就失敗。解法是切分 query：

- **Train set（約 60%）**：用來找出失敗並指導修改。
- **Validation set（約 40%）**：保留下來，只用來檢查修改是否泛化。

兩組都要按比例含應觸發與不應觸發的 query，不要把正例全放在同一組。隨機打散，之後各輪固定不變，才能公平比較。用上述腳本時，可把 query 分成 `train_queries.json` 與 `validation_queries.json` 分別執行。

---

## 優化迴圈

1. **評估**：在 train 與 validation 兩組上評估目前的 description。train 結果指導修改；validation 結果告訴你修改是否泛化。
2. **找出 train set 的失敗**：哪些應觸發的沒觸發？哪些不應觸發的觸發了？只用 train set 的失敗指導修改；無論自己改或請 LLM 改，都不要把 validation 結果帶進修改過程。
3. **修改 description**，著重泛化：
   - 應觸發的失敗：description 可能太窄。放寬範圍或補充 skill 何時有用。
   - 不應觸發卻觸發：description 可能太寬。補充 skill **不做**什麼，或釐清它與相鄰能力的邊界。
   - 不要把失敗 query 的特定關鍵字加進去（那是過擬合）；找出這些 query 代表的一般類別或概念來處理。
   - 多輪後仍卡住，改用結構上不同的寫法，而非微調；不同框架或句型可能突破微調做不到的地方。
   - 確認仍在 1024 字元上限內；description 在優化過程中容易越寫越長。
4. **重複** 1–3，直到 train set 全部通過或不再有明顯進步。
5. **選最佳版本**：依 validation pass rate（validation set 中通過的比例）選。最佳 description 不一定是最後一版；較早的版本可能比後來對 train set 過擬合的版本有更高的 validation pass rate。

通常五輪就夠。若表現沒有改善，問題可能在 query 本身（太容易、太難或標錯），而非 description。

[`skill-creator`](https://github.com/anthropics/skills/tree/main/skills/skill-creator) skill 可端到端自動化此迴圈：切分 eval set、平行評估 trigger rate、用 Claude 提出 description 改進，並產生可即時觀看的 HTML 報告。

---

## 套用結果

選定最佳 description 後：

1. 更新 `SKILL.md` frontmatter 的 `description`。
2. 確認 description 在 1024 字元上限內。
3. 確認觸發符合預期：手動試幾個 prompt 做快速檢查；更嚴謹的做法是另寫 5–10 筆新 query（混合應觸發與不應觸發）跑 eval 腳本。這些 query 從未參與優化，能誠實檢驗 description 是否泛化。

Before / After：

```yaml
# Before
description: Process CSV files.

# After
description: >
  Analyze CSV and tabular data files — compute summary statistics,
  add derived columns, generate charts, and clean messy data. Use this
  skill when the user has a CSV, TSV, or Excel file and wants to
  explore, transform, or visualize the data, even if they don't
  explicitly mention "CSV" or "analysis."
```

改進後的 description 對「做什麼」更具體（summary stats、derived columns、charts、cleaning），對「何時適用」更寬（CSV、TSV、Excel；即使沒出現明確關鍵字）。

觸發可靠之後，下一步是評估輸出品質，見 [`evaluating-skills.md`](evaluating-skills.md)。
