# Skill 撰寫最佳實踐

PURPOSE: 整理 agentskills.io 對 skill 作者的撰寫指引：從真實經驗取材、以實際執行修正、節省 context、依任務脆弱度校準控制力，以及高價值的指示寫作模式。
APPLIES TO: 撰寫新 skill、改寫既有 skill 的 body、審查 skill 內容的範圍與詳細度。
關鍵字: skill best practices, real expertise, context budget, coherent unit, calibrating control, gotchas, output template, checklist, validation loop, plan-validate-execute.

> 來源：[Best practices for skill creators](https://agentskills.io/skill-creation/best-practices)，本地快照 [`source/skill-creation/best-practices.md`](source/skill-creation/best-practices.md)。本文件為摘要，非官方譯文，以原文為準。本頁內容全屬 **[建議]**。

---

## 從真實專業出發

常見陷阱是請 LLM 在沒有領域脈絡的情況下生成 skill，只靠它的通用訓練知識。結果是含糊、通用的程序（「妥善處理錯誤」「遵循驗證的最佳實踐」），而不是讓 skill 有價值的具體 API 模式、edge case 與專案慣例。有效的 skill 建立在真實專業上，關鍵是把領域專屬脈絡餵進建立過程。

### 從實作任務萃取

和 agent 在對話中完成一個真實任務，過程中提供脈絡、糾正與偏好，再把可重用的模式抽成 skill。注意：

- **有效的步驟**：促成成功的動作序列。
- **你做過的糾正**：你導正 agent 做法的地方（例如「用 library X 不要用 Y」「檢查 edge case Z」）。
- **輸入 / 輸出格式**：資料進來與出去的樣子。
- **你提供的脈絡**：agent 原本不知道的專案事實、慣例或限制。

### 從既有專案產物合成

已有一批知識時，可以交給 LLM 合成 skill。用團隊真實的事故報告與 runbook 合成的 data-pipeline skill，會勝過用通用「data engineering best practices」文章合成的版本，因為它捕捉到的是**你的** schema、失敗模式與復原程序。重點是專案專屬素材，不是通用參考。好的素材包括：

- 內部文件、runbook、style guide
- API spec、schema、設定檔
- code review 意見與 issue tracker（反映反覆出現的顧慮與 reviewer 期待）
- 版控歷史，特別是 patch 與 fix（從實際改動看出模式）
- 真實失敗案例與其解法

---

## 以實際執行修正

skill 初稿通常需要修。拿 skill 跑真實任務，把**所有**結果（不只失敗）回饋到建立過程，問：哪裡誤觸發？漏了什麼？什麼可以刪？即使只做一輪「執行 → 修訂」品質也會明顯提升；複雜領域常需要多輪。

讀 agent 的執行軌跡，不只看最終輸出。agent 在無效步驟上浪費時間，常見原因是：

- 指令太模糊：agent 試了好幾種做法才找到可行的。
- 指令不適用於當下任務：agent 仍照做。
- 給了太多選項卻沒有清楚的預設值。

更結構化的迭代（測試案例、assertions、評分）見 [`evaluating-skills.md`](evaluating-skills.md)。

---

## 明智地使用 context

skill 啟動後，整份 `SKILL.md` body 會與對話歷史、system context、其他已啟動 skill 一起載入 context window。skill 的每個 token 都在和 window 內其他內容競爭 agent 的注意力。

### 只加 agent 缺的，省略它已知的

聚焦在沒有 skill 時 agent **不會**知道的事：專案慣例、領域程序、不明顯的 edge case、該用的特定工具或 API。不需要解釋 PDF 是什麼、HTTP 怎麼運作、database migration 做什麼。

````markdown
<!-- Too verbose — the agent already knows what PDFs are -->
## Extract PDF text

PDF (Portable Document Format) files are a common file format that contains
text, images, and other content. To extract text from a PDF, you'll need to
use a library. pdfplumber is recommended because it handles most cases well.

<!-- Better — jumps straight to what the agent wouldn't know on its own -->
## Extract PDF text

Use pdfplumber for text extraction. For scanned documents, fall back to
pdf2image with pytesseract.

```python
import pdfplumber

with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
````

對每段內容問：「沒有這句指示，agent 會做錯嗎？」答案為否就刪；不確定就測。如果 agent 不靠 skill 就能把整個任務做好，這個 skill 可能沒有增加價值；系統化測試方法見 [`evaluating-skills.md`](evaluating-skills.md)。

### 設計內聚的單元

決定 skill 涵蓋範圍，就像決定一個函式該做什麼：要封裝一個能與其他 skill 良好組合的內聚工作單元。

- 範圍太窄：一個任務要載入多個 skill，有額外負擔，指令也可能互相衝突。
- 範圍太寬：難以精準啟動。

例如「查詢資料庫並格式化結果」可以是一個內聚單元；再涵蓋資料庫管理，大概就做太多了。

### 詳細度適中

過度完整的 skill 弊大於利：agent 難以從中挑出相關部分，還可能被不適用於當下任務的指令帶往無效路徑。簡潔的逐步指引加一個可運作的範例，通常勝過窮舉式文件。發現自己在涵蓋每個 edge case 時，想想多數是否交給 agent 自己判斷更好。

### 以 progressive disclosure 組織大型 skill

specification 建議 `SKILL.md` 少於 500 行、5,000 tokens，只放每次執行都需要的核心指示（見 [`specification.md`](specification.md#progressive-disclosure)）。skill 確實需要更多內容時，把詳細參考資料移到 `references/` 等目錄的獨立檔案。

關鍵是告訴 agent **何時**載入哪個檔案。「API 回傳非 200 狀態碼時讀 `references/api-errors.md`」比籠統的「詳見 references/」有用得多，讓 agent 按需而非預先載入 context。

---

## 校準控制力

skill 的各部分不需要同樣的指令強度。讓指令的具體程度配合任務的脆弱度，多數 skill 兩者混用，各部分獨立校準。

### 具體程度配合脆弱度

**給 agent 自由**：多種做法都可行、任務容忍差異時。彈性指示附上**原因**往往比僵硬指令有效；理解指令目的的 agent 能依情境做更好的決定。code review skill 可以只描述要看什麼，不規定確切步驟：

```markdown
## Code review process

1. Check all database queries for SQL injection (use parameterized queries)
2. Verify authentication checks on every endpoint
3. Look for race conditions in concurrent code paths
4. Confirm error messages don't leak internal details
```

**給精確指令**：操作脆弱、一致性重要、或必須遵循特定順序時：

````markdown
## Database migration

Run exactly this sequence:

```bash
python scripts/migrate.py --verify --backup
```

Do not modify the command or add additional flags.
````

### 給預設，不給選單

多種工具或做法都可行時，選一個預設，替代方案簡短帶過，不要把它們並列為同等選項：

````markdown
<!-- Too many options -->
You can use pypdf, pdfplumber, PyMuPDF, or pdf2image...

<!-- Clear default with escape hatch -->
Use pdfplumber for text extraction:

```python
import pdfplumber
```

For scanned PDFs requiring OCR, use pdf2image with pytesseract instead.
````

### 教程序，不教宣告

skill 應該教 agent **如何處理**一類問題，而不是針對某個實例**產出什麼**：

```markdown
<!-- Specific answer — only useful for this exact task -->
Join the `orders` table to `customers` on `customer_id`, filter where
`region = 'EMEA'`, and sum the `amount` column.

<!-- Reusable method — works for any analytical query -->
1. Read the schema from `references/schema.yaml` to find relevant tables
2. Join tables using the `_id` foreign key convention
3. Apply any filters from the user's request as WHERE clauses
4. Aggregate numeric columns as needed and format as a markdown table
```

這不表示 skill 不能有具體細節：輸出格式範本、「never output PII」這類限制、工具專屬指示都有價值。重點是**做法**要能泛化，即使個別細節是具體的。

---

## 有效指示的寫作模式

以下是可重用的內容組織技巧，不是每個 skill 都需要全部，挑適合任務的用。

### Gotchas 段落

許多 skill 中價值最高的內容是 gotchas 清單：違反合理假設的環境專屬事實。它們不是通用建議（「妥善處理錯誤」），而是對 agent 若沒被告知就會犯的錯所做的具體糾正：

```markdown
## Gotchas

- The `users` table uses soft deletes. Queries must include
  `WHERE deleted_at IS NULL` or results will include deactivated accounts.
- The user ID is `user_id` in the database, `uid` in the auth service,
  and `accountId` in the billing API. All three refer to the same value.
- The `/health` endpoint returns 200 as long as the web server is running,
  even if the database connection is down. Use `/ready` to check full
  service health.
```

gotchas 放在 `SKILL.md` 本體，讓 agent 在遇到情況前就讀到。放在獨立 reference 檔也可以，前提是告訴 agent 何時載入；但對不明顯的問題，agent 可能認不出觸發時機。

每當 agent 犯了需要你糾正的錯，就把糾正加進 gotchas。這是迭代改善 skill 最直接的方法之一。

### 輸出格式範本

需要特定輸出格式時提供範本。這比用文字描述格式可靠，因為 agent 很擅長比對具體結構。短範本可內嵌在 `SKILL.md`；較長的、或只在特定情況需要的範本，放在 `assets/` 並從 `SKILL.md` 引用，讓它只在需要時載入。

````markdown
## Report structure

Use this template, adapting sections as needed for the specific analysis:

```markdown
# [Analysis Title]

## Executive summary
[One-paragraph overview of key findings]

## Key findings
- Finding 1 with supporting data
- Finding 2 with supporting data

## Recommendations
1. Specific actionable recommendation
2. Specific actionable recommendation
```
````

### 多步驟工作流程的 checklist

明確的 checklist 幫助 agent 追蹤進度、避免跳步，特別是步驟之間有依賴或驗證關卡時：

```markdown
## Form processing workflow

Progress:
- [ ] Step 1: Analyze the form (run `scripts/analyze_form.py`)
- [ ] Step 2: Create field mapping (edit `fields.json`)
- [ ] Step 3: Validate mapping (run `scripts/validate_fields.py`)
- [ ] Step 4: Fill the form (run `scripts/fill_form.py`)
- [ ] Step 5: Verify output (run `scripts/verify_output.py`)
```

### Validation loop

要求 agent 在往下走之前驗證自己的工作：做完 → 跑驗證器（script、參考 checklist 或自我檢查）→ 修正 → 重複直到通過。

```markdown
## Editing workflow

1. Make your edits
2. Run validation: `python scripts/validate.py output/`
3. If validation fails:
   - Review the error message
   - Fix the issues
   - Run validation again
4. Only proceed when validation passes
```

參考文件本身也可以當「驗證器」：要求 agent 定稿前對照該文件檢查。

### Plan-validate-execute

批次或破壞性操作：讓 agent 先以結構化格式產出中間計畫，對照 source of truth 驗證，通過後才執行。

```markdown
## PDF form filling

1. Extract form fields: `python scripts/analyze_form.py input.pdf` → `form_fields.json`
   (lists every field name, type, and whether it's required)
2. Create `field_values.json` mapping each field name to its intended value
3. Validate: `python scripts/validate_fields.py form_fields.json field_values.json`
   (checks that every field name exists in the form, types are compatible, and
   required fields aren't missing)
4. If validation fails, revise `field_values.json` and re-validate
5. Fill the form: `python scripts/fill_form.py input.pdf field_values.json output.pdf`
```

關鍵是第 3 步：以 script 對照 source of truth（`form_fields.json`）檢查計畫（`field_values.json`）。像「Field 'signature_date' not found — available fields: customer_name, order_total, signature_date_signed」這樣的錯誤，提供 agent 足夠資訊自我修正。

### Bundle 可重用的 scripts

迭代 skill 時比對各測試案例的執行軌跡。若 agent 每次都各自重新發明同樣的邏輯（畫圖、解析特定格式、驗證輸出），就該寫一次測過的 script 放進 `scripts/`。scripts 的設計見 [`using-scripts.md`](using-scripts.md)。

---

## 下一步

- 輸出品質的測試、評分與迭代：[`evaluating-skills.md`](evaluating-skills.md)。
- 讓 description 在正確的 prompt 上觸發：[`optimizing-descriptions.md`](optimizing-descriptions.md)。
