# Diátaxis 與資訊架構

PURPOSE: 整理如何用 Diátaxis 組織文件集，並處理大型文件、landing page、長列表與多維度使用者需求。
APPLIES TO: 文件目錄規劃、knowledge base 分類、產品文件重構、大型文件集導覽設計。
關鍵字: Diátaxis map, documentation architecture, landing page, contents page, complex hierarchies.

> 本文件整理自 Diátaxis map 與 complex hierarchies；非官方譯文。

---

## Map 的用途

Diátaxis map 的價值不是提供四個固定目錄名稱，而是描述四種文件需求之間的關係。

四類文件各自有不同目的：

- Tutorial：引入、教育、帶領，回答「你可以教我做一次嗎？」
- How-to guide：引導完成任務，回答「我要如何做到？」
- Reference：陳述、描述、告知，回答「這是什麼？有哪些事實？」
- Explanation：解釋、澄清、討論，回答「為什麼？它如何連到更大的圖像？」

清楚的文件架構會同時幫助讀者與作者：

- 讀者知道某個位置會提供哪種幫助。
- 作者知道該用哪種語氣、結構與內容邊界。
- 維護者更容易看出缺漏、重複與錯置。

---

## 不要只照產品功能分類

如果沒有清楚的文件架構，作者常會自然地照產品功能或系統模組分類。這在 reference 中通常合理，因為 reference 應反映被描述機器本身的結構。

但若整個文件集都只照產品功能分類，會造成問題：

- Tutorial 會被拆碎，無法形成完整學習經驗。
- How-to guide 會變成工具操作清單，而不是使用者任務。
- Explanation 會散落在功能頁中，無法完整討論設計理由。
- 使用者想完成跨功能任務時，需要自己拼接多處內容。

Diátaxis 的原則是：文件架構要從使用者需求出發，再視需要映射到產品結構。

---

## Landing page 與目錄頁

Home page 與各區 landing page 不應只是連結清單。它們應該提供簡短 overview，讓讀者理解：

- 這一區服務什麼需求。
- 這一區包含哪些內容群組。
- 讀者應該從哪裡開始。
- 不同內容之間如何選擇。

長列表很難閱讀。若列表超過少數項目，除非它有自然排序（例如字母、數字、版本），否則應考慮分組。Diátaxis 原文提到七個項目左右通常是舒適上限；超過時應思考更好的分群方式。

好的 landing page 通常包含：

- 一段說明本區用途的文字。
- 少量有意義的分組標題。
- 每組一段簡短脈絡。
- 每組底下的文件連結。

---

## 大型文件集

大型文件集仍可使用 Diátaxis，但不必只有四個頂層目錄。可以在四類文件底下再加一層分組，例如：

- How-to guides
- Installation guides
- Local installation
- Docker
- Virtual machine
- Deployment and scaling
- Deploy an instance
- Scale an instance

也可以先按使用者類型、部署環境或產品形態分組，再在各組內套用 Diátaxis。關鍵不是外觀看起來是否剛好四格，而是每份內容是否保持自己的目的與寫法。

---

## 多維度問題

複雜文件常同時面對多個結構：

- 使用者類型不同：end users、developers、contributors。
- 使用環境不同：cloud A、cloud B、on-prem。
- 產品在不同情境下幾乎像不同產品。
- 某些內容共享，某些內容必須分開。

Diátaxis 對這類問題的提醒是：先問使用者眼中的產品是什麼，而不是產品團隊眼中的產品是什麼。

如果不同使用者群面對的任務、語言、限制與工作流完全不同，那就應該讓文件結構反映這些差異。若某些內容可共享，則集中成共同 reference 或 explanation，其他文件用連結指向它。

---

## 實作原則

設計資訊架構時遵守幾個原則：

- 讓文件複雜度符合真實需求；必要的複雜不是錯誤。
- 即使結構複雜，也要讓導覽模式穩定且可預期。
- 不要為了遵守四分類而犧牲使用者路徑。
- 不要把不同文件型態混進同一頁，只因為它們屬於同一功能。
- 共享知識應集中在一處，其他地方連結過去，避免重複與漂移。

---

## 來源

- [The map](https://diataxis.fr/map/)
- [Diátaxis in complex hierarchies](https://diataxis.fr/complex-hierarchies/)
- [The compass](https://diataxis.fr/compass/)
