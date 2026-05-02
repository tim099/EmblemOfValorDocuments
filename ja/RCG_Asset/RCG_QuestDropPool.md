---
title: 任務/事件掉落池 (RCG_QuestDropPool) 說明
description: 定義「在某情境下會觸發哪些事件 (RCG_QuestData)」的資料；地圖節點、隨機事件來源
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-ja
---

> [!WARNING]
> 翻訳待機中 — このファイルは日本語翻訳が必要です。
参考用に zh-Hant 原文を以下に掲載しています。


# 任務/事件掉落池

> 程式類別名稱：`RCG_QuestDropPool`

## 用途

定義「**這個情境下可能觸發哪些事件 (Quest)**」。例如「中型城鎮」「森林神祕節點」「夜晚遭遇」分別是不同池，池內列出可能的 `RCG_QuestData`（每個 Quest 是一段事件腳本：對話、選擇、結果）+ 權重。

繼承自 `RCG_Asset<RCG_QuestDropPool>`，實作介面：`UCL.Core.UCLI_ShortName`。

## 編輯器中的樣貌

```
RCG_QuestDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ DropPool / MixDropPools / FilterDropData
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **DropType** | 是 | `DropPool` / `MixPool` / `FilterDrop` |
| **DropPool** | DropType=DropPool | 事件清單 + 權重 |
| **MixDropPools** | DropType=MixPool | 引用其他池子並指定權重 |
| **FilterDropData** | DropType=FilterDrop | 用內部 `DropFilter`（FilterType: Tag / Operator）篩 |

## 行為說明

### 三種模式
與其他 Drop Pool 同。FilterDrop 只支援「事件標籤」單一維度。

### 隱性篩選
本類別**沒有 unlock 篩選**——預設 filter 永遠 return true（程式內有註解掉的「不連續觸發相同事件」邏輯，目前未啟用）。

### 預覽
按 `ShowDetail` 即時看當前掉落率。

## 注意事項

*   **`enum DropType` 不是 `EDropType`**：與 BattleSetDropPool / ItemDropPool 等命名不一致；這是歷史遺留差異。
*   **無 Name 欄位**：`GetShortName()` 直接取第一個 drop 的名稱。
*   **沒有自動清理失效 ID 的 `DeserializeFromJson` override**——壞 ID 不會自動消失，需要手動修。

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊

*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_QuestDropPool.cs`
*   **繼承自**：`RCG_Asset<RCG_QuestDropPool>`
*   **實作介面**：`UCL.Core.UCLI_ShortName`
*   **AssetGroup**：`EditDropSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_DropPool` | 掉落池 | `RCG_CommonDropSetting<RCG_QuestGenData>` | — | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | — | `Conditional(MixPool)` |
| `m_FilterDropData` | 條件式 | `FilterDropData` = `FilterDropDataBase<RCG_QuestGenData, RCG_QuestData, DropFilter>` | — | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `DropType` enum | `DropType` | 預設 `DropPool` |

### A.3 重要 Method 摘要

*   **`GetDrops(int, CheckDropConditionData)`** → 主要入口。
*   **`GetDropsWithFilterFunc(int, Func)`** → 自訂篩選版。
*   **`GetDropRate(CheckDropConditionData, int)`** → 預設 filter 永遠 true（程式註解掉「不連續觸發相同事件」邏輯）。

### A.4 與其他系統的互動

*   **`RCG_QuestData`** — 池子掉的目標型別。
*   **`RCG_QuestGenData`** / **`RCG_QuestDropPoolGenData`** — Asset Entry 包裝；後者預設 ID = `"NormalDrop"`。
*   **`RCG_EventTagGenData`** — FilterDrop 條件用的事件標籤型別。

### A.5 已知議題

*   程式內有註解掉的「不連續觸發相同事件」(`triggeredStory.LastElement()`) 邏輯——若需此行為要解註解。
*   無 `DeserializeFromJson` override，壞 ID 不會自動清理。