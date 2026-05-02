---
title: 道具掉落池 (RCG_ItemDropPool) 說明
description: 定義一組「會掉哪些道具、各自權重」的資料；事件獎勵、寶箱、商店補貨用它抽道具
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-en
---

> [!WARNING]
> Translation pending — this file needs an English translation.
The original zh-Hant content is included below for reference.


# 道具掉落池

> 程式類別名稱：`RCG_ItemDropPool`

## 用途

定義「**這個池子會掉哪些消耗道具、各自的權重**」。寶箱、事件獎勵、商人補貨從這裡抽道具（藥水、捲軸、食材⋯）。與 `RCG_CardDropPool` 相同骨架，只是抽的對象換成 `RCG_ItemData`。

繼承自 `RCG_Asset<RCG_ItemDropPool>`，實作介面：`UCL.Core.UCLI_ShortName`。

## 編輯器中的樣貌

```
RCG_ItemDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    Name(多國語言)
    ▼ DropPool / MixDropPools / FilterDropData  ← 視 DropType 顯示
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **DropType** | 是 | `DropPool`（直接列）/ `MixPool`（混合別的池）/ `FilterDrop`（標籤條件篩） |
| **Name** | 否 | 顯示名稱（多語系）。空白時 fallback 到第一個 drop 名稱再 fallback 到 ID |
| **DropPool** | DropType=DropPool | 道具清單 + 權重（`RCG_CommonDropSetting<RCG_ItemGenData>`） |
| **MixDropPools** | DropType=MixPool | 引用其他池子並指定權重 |
| **FilterDropData** | DropType=FilterDrop | 用內部 `DropFilter`（FilterType: Tag / RarityTag / Operator）動態篩 |

## 行為說明

### 三種模式
*   **DropPool**：手動列出每個道具 + 權重。
*   **MixPool**：把多個既有池子按權重合併。最多遞迴 10 層。
*   **FilterDrop**：條件式（道具標籤、稀有度），支援 AND / OR / NOT。沒條件 = 全道具庫均等。

### 隱性篩選（runtime）
*   **未解鎖**（`UnlockData.m_LockedItems.CheckLocked`）：跳過。
被剔除後剩餘道具**重新標準化權重**。

### 預覽
編輯器內按 `ShowDetail` 可即時看當前掉落率列表。

## 注意事項

*   **DropPool 模式下** 反序列化時會自動移除不存在的道具 ID（檢查 `iData.Exist()`）。
*   **FilterDrop 內部 `DropFilter`** 與 `RCG_CardDropPool` 使用的 `CardDropFilter` 不是同一個型別 — 道具不需要 SkillTag、EquipmentType 等卡牌專屬篩選。
*   **MixPool 循環**會被 `iLayer > 10` 截斷。
*   **道具未解鎖**會直接從 runtime 池中剔除，編輯器預覽不一定會反映此 runtime 邏輯。

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊

*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_ItemDropPool.cs`
*   **繼承自**：`RCG_Asset<RCG_ItemDropPool>`
*   **實作介面**：`UCL.Core.UCLI_ShortName`
*   **AssetGroup**：`EditDropSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_Name` | 顯示名稱 | `RCG_LocalizeData` | `Name` | |
| `m_DropPool` | 掉落池 | `RCG_CommonDropSetting<RCG_ItemGenData>` | — | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | — | `Conditional(MixPool)` |
| `m_FilterDropData` | 條件式 | `FilterDropData` = `FilterDropDataBase<RCG_ItemGenData, RCG_ItemData, DropFilter>` | — | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `EDropType` enum | `DropType` | |

### A.3 重要 Method 摘要

*   **`GetDropItems(int)`** → 主要入口；自動套用 unlock 篩選後抽 N 個。
*   **`GetDropItemsWithFilterFunc(int, Func)`** → 自訂篩選版。
*   **`GetDropRate(CheckDropConditionData, int)`** → 標準化後權重表。
*   **`DeserializeFromJson`** → 清理失效 ID。

### A.4 與其他系統的互動

*   **`RCG_ItemData`** — 池子掉的目標型別。
*   **`RCG_ItemGenData`** — Asset Entry 包裝。
*   **`RCG_ItemDropPoolGenData`** — 其他 Asset 引用此池子時的型別。
*   **`RCG_DataService.Ins.m_UnlockData.m_LockedItems`** — unlock 篩選來源。
*   **`FilterDropDataBase<TGenData, TData, TFilter>`** — 通用條件篩 base class。

### A.5 已知議題

*   程式註解 `// 清理不存在的道具 QWQ` — 標示 deserialize 階段的清理流程。
*   遞迴上限 10 層（`iLayer > 10`）。