# ADR Templates

PURPOSE: 整理 adr.github.io 提到的主要 ADR 模板，協助團隊選擇適合的決策記錄格式。
APPLIES TO: ADR template selection, MADR, Nygard ADR, Y-Statement, decision record format.
關鍵字: ADR templates, MADR, Nygard ADR, Y-Statement, ISO 42010, pros and cons.

> 本文件整理自 [ADR Templates](https://adr.github.io/adr-templates/) 與 [MADR](https://adr.github.io/madr/)；非官方譯文。

---

## 模板選擇原則

ADR 模板不是目的。模板的價值在於讓團隊穩定捕捉同一類關鍵資訊：

- 決策背景。
- 問題陳述。
- 選項。
- 選擇理由。
- 取捨與後果。
- 狀態與後續確認方式。

越複雜、越多人協作、越需要 review 的決策，越適合使用較完整的模板。越小、越輕量的決策，越適合簡短格式。

---

## MADR

MADR 是 Markdown Architectural Decision Records。它用 Markdown 以結構化方式記錄重要決策。

MADR 強調 considered options 與 pros / cons，因為理解「為什麼選了 A」通常也需要知道「為什麼沒有選 B / C」。

典型 MADR 內容：

- Context and Problem Statement。
- Decision Drivers。
- Considered Options。
- Decision Outcome。
- Consequences。
- Confirmation。
- Pros and Cons of the Options。
- More Information。

MADR 也可包含 metadata，例如 status、date、decision-makers、consulted、informed。這讓它適合多人協作、需要 review 或需要追蹤實作確認的決策。

適合使用 MADR 的情境：

- 決策選項多，且取捨需要透明。
- 團隊需要 review / proposal / accepted 等流程。
- 決策可能需要確認實作是否符合。
- 希望用 Markdown 與 docs-as-code 維護。

---

## Nygard ADR

Nygard ADR 來自 Michael Nygard 的 `Documenting Architecture Decisions`，是早期流行並影響廣泛的 ADR 格式。

典型結構：

- Title。
- Status。
- Context。
- Decision。
- Consequences。

它比 MADR 簡潔，適合希望低摩擦開始記錄決策的團隊。

適合使用 Nygard ADR 的情境：

- 團隊剛開始導入 ADR。
- 希望模板短、容易填寫。
- 決策不需要完整選項矩陣。
- 重點是保存 context、decision、consequences。

風險是：若沒有明確記錄 considered options，後來的人可能仍然不知道其他方案為何被排除。

---

## Y-Statement

Y-Statement 是非常輕量的決策敘述格式。

短版：

```text
In the context of <context>, facing <concern>, we decided for <option>
to achieve <quality>, accepting <downside>.
```

長版：

```text
In the context of <context>, facing <concern>, we decided for <option>
and neglected <alternative>, to achieve <quality>, accepting <downside>,
because <rationale>.
```

Y-Statement 適合用來快速捕捉決策核心：情境、面對的問題、選擇、目標品質、接受的代價與理由。

適合使用 Y-Statement 的情境：

- 需要在討論中快速記錄。
- 決策簡短但仍需保留 trade-off。
- 想把決策濃縮成一段可讀敘述。
- 可作為較完整 ADR 的摘要。

---

## Other ADR templates

adr.github.io 也提到其他 ADR 格式，例如 ISO/IEC/IEEE 42010:2011 附錄中的 architecture decision information items，以及社群整理的大量模板。

選用其他模板時，應檢查它是否至少回答：

- 這個決策解決什麼問題？
- 當時有哪些重要背景與限制？
- 考慮過哪些選項？
- 為什麼選這個？
- 接受了哪些正面與負面後果？
- 決策狀態是什麼？
- 何時需要 revisiting 或 superseding？

---

## 選型建議

若團隊沒有既有慣例：

- 從 Nygard ADR 開始，最容易養成習慣。
- 若決策經常需要比較多個選項，改用 MADR。
- 若只需要快速捕捉會議中的決策，可用 Y-Statement 作為草稿，再視需要擴充成 ADR。

無論選哪個模板，都應保持一個原則：ADR 要讓未來讀者理解決策理由與後果，而不是只填完表格。

---

## 來源

- [ADR Templates](https://adr.github.io/adr-templates/)
- [Markdown Architectural Decision Records](https://adr.github.io/madr/)
