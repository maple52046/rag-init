# Diátaxis 技術文件框架

PURPOSE: 整理 Diátaxis 的核心觀念，作為規劃、重整與審視技術文件時的分類與判斷準則。
APPLIES TO: 技術文件資訊架構、文件類型分類、文件重構、知識庫治理、README / how-to / reference / explanation 的拆分。
關鍵字: Diátaxis, documentation, tutorials, how-to guides, reference, explanation, compass, information architecture.

> 本知識庫依據 [Diátaxis](https://diataxis.fr/) 官方網站整理為摘要與導入筆記；非官方譯文。使用時請以原文為準。

---

## Diátaxis 是什麼

Diátaxis 是一套技術文件寫作與資訊架構方法。它主張文件應圍繞使用者的需求組織，而不是只照產品功能、團隊邊界或作者習慣堆疊。

Diátaxis 將文件需求分成四類：

- **Tutorials**：教學課程，服務「學習與技能取得」。
- **How-to guides**：任務指南，服務「完成真實工作」。
- **Reference**：參考資料，服務「工作時查找正確資訊」。
- **Explanation**：解釋文章，服務「理解脈絡、原因與設計取捨」。

這四類不是任意清單，而是由兩個維度交叉而成：

- 文件是在引導 **action**，還是在提供 **cognition**。
- 使用者是在 **acquisition of skill**（學習/取得技能），還是在 **application of skill**（工作/應用技能）。

---

## 核心判斷

Diátaxis 最常見的價值不是「把目錄硬分成四格」，而是幫助作者在寫每一頁、每一節、甚至每一句時問：

- 這段內容服務的是哪一種使用者需求？
- 它現在的語氣、結構與放置位置，是否符合那個需求？
- 是否把教學、任務、參考、解釋混在一起，導致讀者在錯誤的情境讀到錯誤形態的內容？

文件品質常見問題來自邊界模糊：

- 在 tutorial 裡塞太多背景解釋，打斷學習者正在操作的注意力。
- 把 how-to guide 寫成工具功能清單，而不是使用者真實任務。
- 在 reference 裡加入過多教學或討論，降低查找速度與權威感。
- 把 explanation 散落到各處，讓理解脈絡無法完整發展。

---

## 文件結構

- [`four-types.md`](four-types.md)：四類文件的目的、寫法、適用情境與常見錯誤。
- [`compass.md`](compass.md)：用 action/cognition 與 acquisition/application 判斷內容歸屬。
- [`workflow.md`](workflow.md)：如何用 Diátaxis 逐步改善既有文件，不以大規模重寫為前提。
- [`information-architecture.md`](information-architecture.md)：如何用 Diátaxis 組織文件、處理大型或多維度文件集。
- [`quality.md`](quality.md)：Diátaxis 對文件品質的定位：功能品質與深層品質。

建議閱讀順序：

1. 先讀本頁，建立四類文件與兩個維度的概念。
2. 再讀 `compass.md`，學會判斷任一內容應歸到哪一類。
3. 讀 `four-types.md`，掌握每一類文件的寫作準則。
4. 需要整理既有文件時，讀 `workflow.md` 與 `information-architecture.md`。
5. 需要討論文件品質時，讀 `quality.md`。

---

## 來源

- [Diátaxis](https://diataxis.fr/)
- [Start here - Diátaxis in five minutes](https://diataxis.fr/start-here/)
- [Foundations](https://diataxis.fr/foundations/)
