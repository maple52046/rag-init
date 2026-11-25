# ADR 捕捉與審查實務

PURPOSE: 整理 ADR practices 頁面的實務重點，協助團隊把 decision capturing 放進日常架構與設計流程。
APPLIES TO: ADR creation, ADR review, decision readiness, decision done, architecture governance.
關鍵字: ADR practices, definition of ready, definition of done, ADR review, decision drivers, alternatives.

> 本文件整理自 [AD Practices](https://adr.github.io/ad-practices/) 與 adr.github.io 指向的實務主題；非官方譯文。

---

## ADR 應納入決策流程，而不是事後補文

ADR 的價值在於捕捉 decision making，而不只是記錄 decision result。

若只在決策很久之後補寫，常會遺失：

- 當時的 constraints。
- 被排除方案。
- 重要的反對意見。
- 團隊為何接受某些代價。
- 決策當下的不確定性。

實務上，ADR 可以從 proposed 狀態開始，在討論與 review 中逐步成熟，最後變成 accepted、rejected 或其他狀態。

---

## Definition of Ready for ADR

在開始正式決策前，應確認問題是否已足夠清楚。

可檢查：

- 問題是否具備 architecture / design significance。
- context 是否足以讓 reviewer 理解限制。
- 決策範圍是否清楚，不與其他決策混在一起。
- 是否列出主要 decision drivers。
- 是否至少有可比較的候選方案。
- 是否知道誰需要參與、諮詢或被通知。

若這些條件不足，ADR 可能應先停留在草稿或 problem framing，而不是急著接受。

---

## 好的 ADR 應包含選項與取捨

adr.github.io 的 template 討論特別強調 considered options 與 pros / cons 的重要性。

好的 ADR 不只說：

- 我們選了 X。

還要說：

- 面對什麼 context 與 concern。
- X 與其他選項相比解決了什麼。
- 為了 X 接受了哪些 downside。
- 哪些品質屬性被改善，哪些被犧牲。
- 哪些假設若改變，決策可能需要重看。

這讓 ADR 能支援後續演進，而不是變成不可挑戰的宣告。

---

## Review ADR

ADR review 不只是文字校稿，而是檢查決策品質。

Review 時可問：

- Problem statement 是否清楚？
- Decision drivers 是否在同一抽象層級，避免把 criteria 混得太雜？
- Options 是否足以代表實際選擇空間？
- Rationale 是否能解釋為什麼選這個，而不是只描述偏好？
- Consequences 是否包含正面與負面？
- 是否說明 implementation / confirmation plan？
- 是否需要連到其他 ADR、issue、design doc 或 evidence？

Review 的目標不是讓 ADR 很長，而是讓未來讀者能理解決策的必要資訊。

---

## Definition of Done for ADR

一份 ADR 何時算完成，取決於團隊流程，但可用幾個面向判斷：

- **Evidence**：是否有足夠證據支持決策。
- **Criteria and alternatives**：是否說明判斷標準與替代方案。
- **Agreement**：必要參與者是否已達成共識或明確記錄異議。
- **Documentation**：決策、理由與後果是否已清楚記錄。
- **Realization / review plan**：是否知道如何實作、驗證或何時回顧。

若 ADR 已 accepted 但沒有任何實作確認方式，可能會變成只存在文件中的願望。

---

## 從 Architectural Decision 到 Any Decision

AD Practices 頁面提到，有些文章主張把 ADR 擴展成 design decisions 或 Any Decision Record。

這種擴展是合理的，但要小心不要讓 decision log 被低價值決策淹沒。

可用問題判斷是否值得記錄：

- 這個決策未來是否可能被問「為什麼」？
- 它是否影響品質屬性、維護成本、團隊流程或外部介面？
- 它是否有多個合理方案？
- 它是否需要跨團隊共享或被後續工作遵守？

若答案多數為是，就值得用 ADR 或 ADR-like 格式記錄。

---

## 常見反模式

避免這些寫法：

- 只寫最終決定，不寫 context。
- 只寫優點，不寫 downside。
- 把方案比較寫成事後合理化。
- 混入大量 implementation details，掩蓋真正決策。
- 決策改變時直接修改舊 ADR，抹掉歷史。
- ADR 沒有狀態，讀者不知道是否仍有效。
- 所有大小事都寫 ADR，導致 decision log 雜訊過高。

---

## 來源

- [AD Practices](https://adr.github.io/ad-practices/)
- [ADR Templates](https://adr.github.io/adr-templates/)
- [Markdown Architectural Decision Records](https://adr.github.io/madr/)
