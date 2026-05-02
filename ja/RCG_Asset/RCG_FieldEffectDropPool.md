---
title: 場地效果掉落池 (RCG_FieldEffectDropPool) 說明
description: 定義「會抽到哪些場地效果」的資料；地圖節點 / 戰鬥開始時的隨機場地修正來源
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-ja
---

> [!WARNING]
> 翻訳待機中 — このファイルは日本語翻訳が必要です。
参考用に zh-Hant 原文を以下に掲載しています。


# 場地效果掉落池

> 程式類別名稱：`RCG_FieldEffectDropPool`

## 用途

定義「**這個情境下可能抽到哪些場地效果**」。場地效果是**作用於整個戰場**的修正（例：本場戰鬥所有單位每回合 -1 HP / 火焰場地 / 黑暗壟罩）；本池決定隨機場地時的候選與權重。

繼承自 `RCG_Asset<RCG_FieldEffectDropPool>`，實作介面：`UCL.Core.UCLI_ShortName`。

## 編輯器中的樣貌

```
RCG_FieldEffectDropPool: <ID>
    DropType  ▾ DropPool / MixPool      ← 沒有 FilterDrop
    ▼ DropPool / MixDropPools
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **DropType** | 是 | `DropPool` / `MixPool`（**只有兩種模式**） |
| **DropPool** | DropType=DropPool | 場地效果清單 + 權重 |
| **MixDropPools** | DropType=MixPool | 混合其他池子 |

## 行為說明

與 `RCG_StatusDropPool` 結構相同——只有 DropPool / MixPool 兩種模式。

## 注意事項

*   **沒有 FilterDrop 模式**。
*   **enum 名稱叫 `DropType`**（不是 `EDropType`）。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_FieldEffectDropPool.cs`
*   **繼承自**：`RCG_Asset<RCG_FieldEffectDropPool>`
*   **AssetGroup**：`EditDropSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_DropPool` | 掉落池 | `RCG_CommonDropSetting<RCG_FieldEffectGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_DropType` | DropType | `DropType` enum | 只有 `DropPool` / `MixPool` |

### A.3 與其他系統的互動

*   **`RCG_FieldEffectData`** — 掉落目標。
*   **`RCG_FieldEffectGenData`** / **`RCG_FieldEffectDropPoolGenData`** — Asset Entry 包裝。