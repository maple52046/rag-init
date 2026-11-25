# Diátaxis Compass

PURPOSE: 提供判斷文件內容歸屬的最小決策工具，協助把混合內容拆回 tutorials、how-to guides、reference、explanation。
APPLIES TO: 文件審查、文件重構、撰寫前定位、判斷段落是否放錯位置。
關鍵字: Diátaxis compass, action, cognition, acquisition, application, documentation classification.

> 本文件整理自 Diátaxis 的 compass 與 foundations。Compass 不是目錄模板，而是判斷工具。

---

## 兩個問題

判斷任何一段內容時，只問兩個問題。

第一個問題：這段內容是在服務 **action** 還是 **cognition**？

- **Action**：引導使用者做事、操作、執行步驟、做判斷。
- **Cognition**：提供事實、概念、脈絡、原因、關係與理解。

第二個問題：這段內容是在服務 **acquisition** 還是 **application**？

- **Acquisition**：使用者正在學習、取得技能、形成理解。
- **Application**：使用者正在工作、應用既有技能完成任務。

交叉結果：

- Action + Acquisition = **Tutorial**
- Action + Application = **How-to guide**
- Cognition + Application = **Reference**
- Cognition + Acquisition = **Explanation**

---

## 用法

Compass 可以用在不同粒度：

- 一整個文件：判斷它的主型態。
- 一個章節：判斷它是否偏離文件主目的。
- 一段文字：判斷它是否應該移到其他文件類型。
- 一句話：判斷語氣是否在教學、指令、描述或解釋之間混淆。

當直覺不可靠時，尤其應使用 compass。許多文件看起來像 tutorial，其實使用者是在工作；許多 reference 寫著寫著變成 explanation；許多 how-to guide 被工具功能清單偽裝。

---

## 判斷提示

若內容主要是在回答：

- 「你可以帶我做一次嗎？」通常是 tutorial。
- 「我要怎麼完成這件事？」通常是 how-to guide。
- 「這個東西有哪些選項、限制、預設值、行為？」通常是 reference。
- 「為什麼是這樣？它和其他概念如何連在一起？」通常是 explanation。

若讀者狀態是：

- 新手正在建立信心與基本能力：tutorial。
- 已具備基本能力、正在解任務：how-to guide。
- 正在工作中查正確事實：reference。
- 離開操作現場、想補理解：explanation。

---

## 典型移動規則

在 tutorial 中發現長篇原因、歷史、架構討論：

- 移到 explanation。
- Tutorial 只保留一句足以支撐當下操作的最小說明，並連過去。

在 how-to guide 中發現完整參數表、選項清單、API 欄位定義：

- 移到 reference。
- How-to guide 只保留該任務需要的選項與判斷。

在 reference 中發現「為什麼要這樣設計」「什麼情境該選 A 或 B」：

- 原因與取捨移到 explanation。
- 實際選擇流程若是任務，移到 how-to guide。

在 explanation 中發現逐步操作：

- 移到 how-to guide 或 tutorial。
- Explanation 可以保留概念性描述，說明步驟背後的原因。

---

## 審查問題

審查文件時可以逐段問：

- 這段是在要求讀者做事，還是在讓讀者知道/理解？
- 讀者是在學習，還是在工作？
- 這段內容是否打斷了讀者當下的節奏？
- 若它是必要資訊，是否應該以連結指向另一類文件，而不是塞在本文？
- 標題是否反映使用者需求，而不是只反映工具或功能名稱？

---

## 來源

- [The compass](https://diataxis.fr/compass/)
- [Foundations](https://diataxis.fr/foundations/)
- [The map](https://diataxis.fr/map/)
