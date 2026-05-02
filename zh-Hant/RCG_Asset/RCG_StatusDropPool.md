---
title: 狀態效果掉落池 (RCG_StatusDropPool) 說明
description: 定義「會掉哪些狀態效果」的資料；隨機 Buff/Debuff、神祕祝福等情境用的池
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 狀態效果掉落池

> 程式類別名稱：`RCG_StatusDropPool`

## 用途

定義「**這個情境下可能抽到哪些狀態效果**」。例如「神壇給予的隨機祝福」「詛咒寶箱的隨機 debuff」「特殊事件的群體 buff」都從這裡抽 `RCG_CustomStatusData`。

繼承自 `RCG_Asset<RCG_StatusDropPool>`，實作介面：`UCL.Core.UCLI_ShortName`。

## 編輯器中的樣貌

```
RCG_StatusDropPool: <ID>
    DropType  ▾ DropPool / MixPool      ← 沒有 FilterDrop
    ▼ DropPool / MixDropPools
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **DropType** | 是 | `DropPool` / `MixPool`（**只有兩種模式**，沒有 FilterDrop） |
| **DropPool** | DropType=DropPool | 狀態清單 + 權重 |
| **MixDropPools** | DropType=MixPool | 混合其他池子 |

## 行為說明

與其他 Drop Pool 骨架類似，但**只有兩種模式**：直接列、混合別的池。沒有「依條件動態篩」這個選項。

## 注意事項

*   **沒有 FilterDrop 模式**：要做動態篩選需手動列舉。
*   **enum 名稱叫 `DropType`**（不是 `EDropType`）。
*   無 unlock 等 runtime 篩選。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_StatusDropPool.cs`
*   **繼承自**：`RCG_Asset<RCG_StatusDropPool>`
*   **AssetGroup**：`EditDropSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_DropPool` | 掉落池 | `RCG_CommonDropSetting<RCG_CustomStatusGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_DropType` | DropType | `DropType` enum | 只有 `DropPool` / `MixPool` |

### A.3 與其他系統的互動

*   **`RCG_CustomStatusData`** — 掉落目標。
*   **`RCG_CustomStatusGenData`** / **`RCG_StatusDropPoolGenData`** — Asset Entry 包裝。
