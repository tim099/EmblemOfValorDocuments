---
title: 故事資料 (RCG_StoryData) 說明
description: 一段「故事」的完整定義：起始事件、子故事字典、條件、隨機起點、Tag 篩選
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-en
---

> [!WARNING]
> Translation pending — this file needs an English translation.
The original zh-Hant content is included below for reference.


# 故事資料

> 程式類別名稱：`RCG_StoryData`

## 用途

**一段「故事」的完整劇本資料**。故事比 `RCG_MapEventData` 更複雜——包含多段子故事 (`SubStory`) 構成的對話樹，玩家做選擇後跳到不同子故事。例如「商人」可以是一個 Story：起始畫面 → 選擇 →「購買」/「離開」/「殺商人」 → 各自的子故事 → 結局。

繼承自 `RCG_Asset<RCG_StoryData>`，實作介面：`UCLI_ShortName`。

## 編輯器中的樣貌

```
RCG_StoryData: <ID>
    StoryData (m_StoryData)         ← 故事的元資料
        Tags / CanTriggerRepeatedly / Conditions
        IsRandomStart / RandomStartStories
    Story (StartEvent)              ← 起始 SubStory（key = "Start"）
    SubStory (m_EventDic)           ← 所有子故事 dictionary：{ id → RCG_OptionEventData }
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Tags** | 否 | 故事標籤（用於 StoryDropPool 篩選） |
| **CanTriggerRepeatedly** | — | 是否可重複觸發 |
| **Conditions** | 否 | 觸發條件（OR；空 = 無條件） |
| **IsRandomStart** | — | 是否從多個起點隨機選一個 |
| **RandomStartStories** | IsRandomStart=true | 隨機起點清單 + 權重 |
| **Story** | 是 | 主起始事件（key 固定為 `"Start"`） |
| **SubStory** | 否 | 所有子故事字典（key = state name，value = `RCG_OptionEventData`） |

每個 `RCG_OptionEventData`（子故事）內含對話 / 選項 / 結果效果，可指定下一個 SubStory 的 ID 形成樹狀結構。

## 行為說明

### 起點選擇 (`GetStartSubStory(iSubStory)`)
*   如果指定了 `iSubStory` → 用它。
*   否則若 `m_IsRandomStart` 且 `m_RandomStartStories` 非空 → 按權重隨機抽。
*   否則 fallback 到 `StartStateName = "Start"`。

### 觸發 (`StartStory(token, iSubStory, iIsRoot)`)
1. 用 `GetStartSubStory(iSubStory)` 決定起點。
2. 寫入 `RCG_DataService.Ins.StartStory(ID, iSubStory, iIsRoot)`（紀錄已觸發故事）。
3. 設 `s_CurStoryData = this`（給其他系統查詢當前故事）。
4. 取對應 SubStory 的 `RCG_OptionEventData.StartEvent(...)` 開始演繹。
5. 結束後清 `s_CurStoryData = null`。

### 條件判斷 (`StoryData.CheckCondition`)
*   `m_Conditions` 為空 → 永遠回 true。
*   非空 → `CheckConditions_OR`（OR 關係）。

## 注意事項

*   **`StartStateName = "Start"` 是 magic string**：起始事件 key 固定為這個；其他名稱不會被當起始。
*   **重複觸發旗標**：`CanTriggerRepeatedly` 預設 true（與 MapEventData 相反）；故事多半允許重複。
*   **`s_CurStoryData` 是全局靜態**：故事執行期間外部能查到當前故事；但**多個故事不能並行**（會互相覆蓋）。
*   **`m_EventDic` 自動補空項**：`GetSubStory(state)` 找不到 key 時會自動 add 一個空 `RCG_OptionEventData`，所以呼叫端不會 NRE，但會看到空白故事。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_StoryData.cs`
*   **繼承自**：`RCG_Asset<RCG_StoryData>`
*   **實作介面**：`UCLI_ShortName`
*   **AssetGroup**：`EditQuestSetting`
*   **常數**：`StartStateName = "Start"`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_StoryData` | StoryData | `StoryData`（巢狀） | 元資料：tags / conditions / random start |
| `m_EventDic` | SubStory | `Dictionary<string, RCG_OptionEventData>` | 子故事樹 |

`StoryData` 巢狀含 `m_Tags` / `m_CanTriggerRepeatedly` / `m_Conditions` / `m_IsRandomStart` / `m_RandomStartStories`。

### A.3 重要 Method

*   **`StartStory(token, iSubStory, iIsRoot)`** — 主觸發；async；含起點選擇 + DataService 紀錄 + s_CurStoryData 切換。
*   **`StartEvent` (property)** — `GetSubStory(StartStateName)`，起始事件捷徑。
*   **`GetSubStory(state)`** — 取子故事；缺則自動 add 空項。
*   **`StoryData.CheckCondition(data)`** — 觸發條件 OR 檢查。
*   **`StoryData.GetStartSubStory(iSubStory)`** — 起點選擇邏輯。
*   **`DefaultStory` (static)** — 預設故事 fallback。

### A.4 與其他系統的互動

*   **`RCG_OptionEventData`** — 每個 SubStory 的內容（對話、選項、結果）。
*   **`RCG_StoryDropPool`** — 隨機池來源。
*   **`RCG_DataService.Ins.StartStory`** — runtime 紀錄已觸發故事。
*   **`RCG_StoryGenData`** — Asset Entry 包裝。
*   **`RCG_GameManager.Random`** — 隨機起點抽取。

### A.5 已知議題

*   `s_CurStoryData` 不支援故事並行；極端情況需考慮中斷處理。
*   `// public class StoryState` 與 `m_OptionEventData` 註解掉，標示舊版單一事件結構轉成 dictionary 的歷史。
*   `DeserializeFromJson` 內舊版遷移邏輯（`m_EventDic[StartStateName] = m_OptionEventData`）已註解。