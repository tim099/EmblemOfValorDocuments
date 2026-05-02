---
title: 故事掉落池 (RCG_StoryDropPool) 說明
description: 定義「在某情境下會抽到哪些故事 (RCG_StoryData)」的資料；劇情段落、隨機故事插入點來源
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 故事掉落池

> 程式類別名稱：`RCG_StoryDropPool`

## 用途

定義「**這個情境下可能抽到哪些故事段落**」。與 `RCG_QuestDropPool` 不同，這裡掉的是 `RCG_StoryData`（純劇情、對話、過場），不含戰鬥或選擇。例如「進入新章節時播放的開場白」「夜晚休息時的旅途見聞」。

繼承自 `RCG_Asset<RCG_StoryDropPool>`，實作介面：`UCL.Core.UCLI_ShortName`。

## 編輯器中的樣貌

```
RCG_StoryDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ DropPool / MixDropPools / FilterDropData
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **DropType** | 是 | `DropPool` / `MixPool` / `FilterDrop` |
| **DropPool** | DropType=DropPool | 故事清單 + 權重 |
| **MixDropPools** | DropType=MixPool | 引用其他池子並指定權重 |
| **FilterDropData** | DropType=FilterDrop | 用內部 `DropFilter` 篩 |

## 行為說明

與其他 Drop Pool 同骨架；無 unlock / skill 等 runtime 篩選。

## 注意事項

*   結構與 `RCG_QuestDropPool` 幾乎相同，差異只在掉落目標型別與篩選 tag 種類。
*   **DropPool 模式下**反序列化會自動清理失效 ID（依 `iData.Exist()`）。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_StoryDropPool.cs`
*   **繼承自**：`RCG_Asset<RCG_StoryDropPool>`
*   **AssetGroup**：`EditDropSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_DropPool` | 掉落池 | `RCG_CommonDropSetting<RCG_StoryGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_FilterDropData` | 條件式 | `FilterDropData` | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `EDropType` enum | 預設 `DropPool` |

### A.3 重要 Method

*   **`GetDrops(int)`** / **`GetDropsWithFilterFunc(int, Func)`** — 抽 N 個故事。
*   **`GetDropRate(...)`** — 標準化權重表。

### A.4 與其他系統的互動

*   **`RCG_StoryData`** — 掉落目標。
*   **`RCG_StoryGenData`** / **`RCG_StoryDropPoolGenData`** — Asset Entry 包裝。
