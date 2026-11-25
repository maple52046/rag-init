# Diátaxis 與文件品質

PURPOSE: 整理 Diátaxis 對文件品質的定位，區分 functional quality 與 deep quality，避免把分類框架誤當成品質保證。
APPLIES TO: 文件品質審查、技術文件治理、文件重構優先順序、documentation review。
關鍵字: Diátaxis quality, functional quality, deep quality, documentation quality, flow, user needs.

> 本文件整理自 Diátaxis 的 documentation quality 討論；非官方譯文。

---

## Diátaxis 是品質方法，但不是完整品質保證

Diátaxis 可以提升文件品質，因為它讓文件更貼近使用者在不同情境下的需求。它能幫助作者避免把不同目的的內容混在一起，進而改善 flow、可用性與讀者感受。

但 Diátaxis 不能取代基本技術品質。文件仍必須準確、完整、一致、可驗證。分類正確但內容錯誤，仍然是壞文件。

---

## Functional quality

Functional quality 是文件作為工具必須滿足的基本標準，例如：

- Accuracy：內容是否符合實際系統。
- Completeness：必要資訊是否齊全。
- Consistency：術語、格式、行為描述是否一致。
- Usefulness：是否真的能幫使用者完成目的。
- Precision：描述是否精確、不含歧義。

這些品質面向彼此獨立。文件可能準確但不完整，也可能完整但難用。它們多半可以透過客觀方式檢查，例如對照程式碼、版本、CLI 輸出、API schema、實際操作結果。

Diátaxis 本身不能保證 functional quality；它不能讓錯誤的參數變正確，也不能自動補齊缺失資訊。

---

## Deep quality

Deep quality 是文件使用起來是否符合人的需求與節奏，例如：

- 使用起來順。
- 有 flow。
- 能預期讀者下一步需要什麼。
- 不打斷讀者當下目的。
- 結構與語氣讓人覺得自然、可信、好用。

這些品質通常無法像參數正確性一樣量測，只能透過判斷、觀察與使用者經驗辨識。它們與人的狀態有關：讀者是在學習、工作、查找，還是反思。

Diátaxis 主要幫助的是 deep quality。它透過四種文件需求，讓內容、語氣、架構與讀者狀態對齊。

---

## 兩種品質的關係

Functional quality 是 deep quality 的前提。

文件可以準確、完整、一致，但仍然使用起來笨重、不順、令人疲憊。這代表它具備 functional quality，但缺少 deep quality。

反過來，文件不可能在錯誤、不完整、不一致的情況下真正具備 deep quality。讀者一旦遇到錯誤事實或不可靠步驟，信任與 flow 就會被破壞。

因此文件審查應同時問：

- 這份文件是否正確、完整、一致？
- 這份文件是否以正確形態服務正確需求？

---

## Diátaxis 如何暴露 functional quality 問題

雖然 Diátaxis 不能直接創造 functional quality，但它常能讓問題變明顯。

例子：

- Reference 若照系統結構排列，缺少的 API、參數或限制會更容易看出。
- Tutorial 移除多餘解釋後，可能暴露某一步其實缺少明確預期結果。
- How-to guide 聚焦任務後，可能發現原文件只有工具功能，缺少真實情境判斷。
- Explanation 從其他文件抽出後，可能發現設計理由互相矛盾或缺乏來源。

這些問題仍需要透過技術驗證、補資料、測試與審查解決。Diátaxis 的作用是讓它們浮現。

---

## Diátaxis 如何創造 deep quality

Diátaxis 能改善 deep quality 的方式：

- 讓文件類型對應讀者需求，避免在錯誤情境使用錯誤語氣。
- 保護讀者 flow，例如不在操作指南中插入長篇背景討論。
- 讓 tutorial 專注於安全、成功、具體的學習經驗。
- 讓 how-to guide 專注於真實任務與合理順序。
- 讓 reference 保持穩定、可預期、可查找。
- 讓 explanation 有空間完整處理原因、取捨與脈絡。

這些不是公式，而是條件。套用 Diátaxis 不保證文件一定優秀，但它能建立優秀文件更可能出現的結構。

---

## 品質審查問題

審查一份文件時，可以分兩層問。

Functional quality：

- 是否符合目前版本與實際行為？
- 是否有缺失的前提、限制、錯誤處理或預期輸出？
- 是否使用一致術語與格式？
- 是否有可驗證來源或實際測試依據？

Deep quality：

- 讀者此刻最可能是學習、工作、查找，還是理解？
- 文件形態是否與這個狀態對齊？
- 是否有不必要的跳轉、打斷、長篇旁枝？
- 是否讓讀者自然知道下一步去哪裡？
- 是否把不同文件類型混在一起，削弱了各自效果？

---

## 來源

- [Towards a theory of quality in documentation](https://diataxis.fr/quality/)
- [The map](https://diataxis.fr/map/)
- [The compass](https://diataxis.fr/compass/)
