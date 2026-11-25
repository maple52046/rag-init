# Diátaxis 四類文件

PURPOSE: 整理 tutorials、how-to guides、reference、explanation 的目的、寫作方式與常見誤用。
APPLIES TO: 判斷一份文件應該怎麼寫、拆分混合型文件、設計文件模板與審查文件品質。
關鍵字: Diátaxis four types, tutorial, how-to guide, reference, explanation, documentation types.

> 本文件是依據 Diátaxis 官方四類文件頁面整理的摘要；非官方譯文。使用時請以原文為準。

---

## Tutorials：教學課程

Tutorial 是一堂課，目標是讓學習者在安全、可預期、可完成的經驗中取得技能與信心。

它服務的是「study」情境：使用者正在學，不是在完成真實工作。Tutorial 的成功不在於產出物本身，而在於學習者透過操作獲得熟悉感、基本能力與信心。

Tutorial 的寫法：

- 從一個可完成、具體、有意義的目標開始。
- 讓學習者儘早且持續看見可理解的結果。
- 每一步都要清楚說明要做什麼、預期會看到什麼。
- 用「we」建立導師與學習者一起走的關係。
- 嚴格減少背景解釋，只保留足以支撐當前操作的最小說明。
- 避免選項、分支、替代方案，讓路徑保持單線且可成功。
- 追求可靠性：學習者照做時應該能得到你承諾的結果。

Tutorial 不應該：

- 假設學習者已經知道工具、流程或常識。
- 把真實世界的複雜分支丟給初學者判斷。
- 把「你會學到」當作承諾；更好的方式是說明「我們會做出什麼」。
- 把 explanation 塞進操作流程，打斷學習者的注意力。

適合標題：

- `Create your first container image`
- `Build and run a minimal inference server`
- `Write your first knowledge article`

---

## How-to guides：任務指南

How-to guide 是為了幫助已具備基本能力的使用者完成一個真實任務或解決一個真實問題。

它服務的是「work」情境：使用者知道自己要達成什麼，也具備足夠能力理解指令與做判斷。How-to guide 不負責教會使用者整個領域，而是引導他把事情完成。

How-to guide 的寫法：

- 從使用者的真實目標命名與組織，而不是從工具功能命名。
- 只保留完成任務所需的行動與判斷。
- 允許條件式分支，因為真實工作常有不同情境。
- 以合理順序安排步驟，讓使用者的操作與思考保持 flow。
- 可以要求使用者自行接上前後文，不必像 tutorial 一樣完整鋪路。
- 必要的背景、完整選項清單或概念說明應連到 reference / explanation。

How-to guide 不應該：

- 寫成「按下按鈕會發生什麼」這種工具操作說明。
- 混入大量教學或概念討論。
- 試圖列出與主題相關的一切資訊。
- 假設只有一條完全受控的路徑；真實工作常需要 `if this, then that`。

適合標題：

- `How to configure package mirrors`
- `How to export a Docker image tarball`
- `How to troubleshoot deployment failures`

---

## Reference：參考資料

Reference 是對系統、API、命令、設定、限制與行為的中立描述。它服務的是工作中的查找需求：使用者需要準確、完整、可靠的事實。

Reference 的寫法：

- 描述事物本身，避免教學、討論與說服。
- 結構盡量反映被描述系統的結構，例如 module / class / method、command / subcommand / option。
- 使用一致、可預期的格式，讓讀者能快速掃描。
- 保持權威感、精確性與完整性。
- 可以提供簡短 example，但 example 是為了說明用法，不是展開成教學或解釋。
- 適當提供警告、限制、預設值、相容性與錯誤訊息。

Reference 不應該：

- 以使用者任務作為主軸；那是 how-to guide 的工作。
- 加入冗長背景與設計理由；那是 explanation 的工作。
- 只依賴自動產生內容而缺少必要的上下文、限制與使用規則。

適合標題：

- `Docker image tag naming reference`
- `Base image build arguments`
- `Supported environment variables`

---

## Explanation：解釋文章

Explanation 是幫助讀者理解脈絡、原因、背景、取捨與更大的圖像。它服務的是「study」情境，但不像 tutorial 那樣引導行動，而是讓讀者在離開當前操作後反思與理解。

Explanation 的寫法：

- 回答 `why`、`why not`、`how it fits together`。
- 提供歷史、設計原因、技術限制、替代方案與取捨。
- 可以承認觀點、判斷與偏好，因為理解常需要 perspective。
- 把概念連起來，協助讀者形成一張理解網。
- 明確界定範圍，避免無限展開或吸收其他文件類型。

Explanation 不應該：

- 變成步驟指南；需要操作時連到 how-to guide。
- 變成 facts list；需要查找資訊時連到 reference。
- 塞進 tutorial 或 how-to 中打斷當下操作。

適合標題：

- `Why image tags include the Python version`
- `About framework image provenance`
- `How Diátaxis differs from feature-based documentation`

---

## 最重要的邊界

Tutorial 與 how-to guide 都引導 action，但前者服務學習，後者服務工作。

Reference 與 explanation 都提供 cognition，但前者服務工作查找，後者服務學習理解。

當不確定時，不要先問「這篇看起來像哪一類」，而要問「讀者現在需要取得技能、完成工作、查找事實，還是理解原因」。判斷方法見 [`compass.md`](compass.md)。

---

## 來源

- [Tutorials](https://diataxis.fr/tutorials/)
- [How-to guides](https://diataxis.fr/how-to-guides/)
- [Reference](https://diataxis.fr/reference/)
- [Explanation](https://diataxis.fr/explanation/)
- [The difference between a tutorial and how-to guide](https://diataxis.fr/tutorials-how-to/)
- [The difference between reference and explanation](https://diataxis.fr/reference-explanation/)
