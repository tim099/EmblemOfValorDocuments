---
title: RCG_StoryData 說明
description: <!-- TODO: 一句話功能摘要 -->
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# RCG_StoryData

> 程式類別名稱：`RCG_StoryData`

## 用途

<!-- TODO: 描述這個 Asset 在遊戲裡負責什麼、什麼情境會用、舉 1-2 個範例。 -->

繼承自 `RCG_Asset<RCG_StoryData>`，實作介面：`UCLI_ShortName`

## 編輯器中的樣貌

```
<!-- TODO: 描繪此 Asset 在編輯器內的版面 -->
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Tags** | — | <!-- TODO: 說明欄位用途 --> |
| **CanTriggerRepeatedly** | — | <!-- TODO: 說明欄位用途 --> |
| **Conditions** | — | <!-- TODO: 說明欄位用途 --> |
| **IsRandomStart** | — | <!-- TODO: 說明欄位用途 --> |
| **RandomStartStories** | — | <!-- TODO: 說明欄位用途 --> |
| **StoryData** | — | <!-- TODO: 說明欄位用途 --> |
| **EventDic** | — | <!-- TODO: 說明欄位用途 --> |

## 行為說明

<!-- TODO: 戰鬥 / 載入 / 解鎖時的觸發時機與順序。 -->

## 注意事項

<!-- TODO: 常見的設計反模式 / 容易踩到的坑。 -->

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊

*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_StoryData.cs`
*   **繼承自**：`RCG_Asset<RCG_StoryData>, UCLI_ShortName`
*   **實作介面**：`UCLI_ShortName`

### A.2 欄位對照（自動產生，需人工複核）

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_Tags` | Tags | `List<RCG_EventTagGenData>` | `Tags` | |
| `m_CanTriggerRepeatedly` | CanTriggerRepeatedly | `bool` | `CanTriggerRepeatedly` | |
| `m_Conditions` | Conditions | `List<RCG_Condition>` | `Conditions` | |
| `m_IsRandomStart` | IsRandomStart | `bool` | `IsRandomStart` | |
| `m_RandomStartStories` | RandomStartStories | `List<SubStoryWeight>` | `RandomStartStories` | UCL.Core.PA.Conditional(nameof(m_IsRandomStart), false, true) |
| `m_StoryData` | StoryData | `StoryData` | `StoryData` | |
| `m_EventDic` | EventDic | `Dictionary<string, RCG_OptionEventData>` | `EventDic` | |

### A.3 重要 Method 摘要

<!-- TODO: 補上影響行為的關鍵 method（OnGUI / Preview / 序列化覆寫等）。 -->

### A.4 與其他系統的互動

<!-- TODO: 列出依賴 / 被依賴的類別與系統。 -->

### A.5 已知議題（選填）

<!-- TODO: TODO/FIXME 摘錄、待重構點。 -->
