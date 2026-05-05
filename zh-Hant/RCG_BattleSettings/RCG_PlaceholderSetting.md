---
title: Placeholder（融合占位）說明
description: 卡牌融合系統使用的占位類別；保留巢狀結構，融合確認後會被替換為實際效果
last_updated: 2026-05-05
target_audience: [Designer, Modder, AI_Agent]
---

# Placeholder（融合占位）

> 程式類別名稱：`RCG_PlaceholderSetting`

## 用途
**卡牌融合系統內部使用的占位類別**。在融合預覽階段保留巢狀結構（例如「條件判斷裡有 placeholder」），讓玩家看出**哪裡會被填入新效果**。融合確認後會被替換為真正的子設定。

> [!IMPORTANT]
> 這個設定**通常不會由你手動建立** — 它由融合系統在執行時自動產生。如果你在資料中看到它，代表這張卡是融合過程中暫存的中介狀態。

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **PlaceholderKey** | 是 | i18n key，用來顯示提示文字（預設 `CardFusionPlaceholder`，會帶上 PlaceholderIndex 編號）。 |
| **PlaceholderIndex** | 是 | 占位編號（多個 placeholder 用編號區分）。 |
| **PreviewDescription** | 否 | 自訂的預覽描述；非空則覆寫 i18n 結果。 |

## 行為說明
*   觸發時什麼也不做（純占位）。
*   描述優先使用 `PreviewDescription`，否則顯示 i18n key `PlaceholderKey` 的結果。
*   作為**葉節點**存在 — `GetFusionCandidateSettings()` 為空（不可作為融合候選），`GetFusionBaseSetting()` 回傳自身（不會被進一步替換）。

## 注意事項
*   **不要在資料中手動建立**：除非你想**故意**標記某個位置為「待融合的洞」。
*   **與「DiminishedPlaceholder」的差別**：本類是**融合系統**的中性占位（藍框）；`RCG_DiminishedPlaceholder` 是**削弱系統**的（紅字「已被削弱」）。
*   **PreviewDescription 優先**：UI 顯示測試時常用此欄位手動 override 提示文字。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：[`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_PlaceholderSetting.cs`](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_PlaceholderSetting.cs)
*   **繼承自**：`RCG_BattleSetting`
*   **無 i18n 類別名 key**：編輯器顯示 stripped name `Placeholder`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_PlaceholderKey` | `PlaceholderKey` | `string` | — | 預設 `"CardFusionPlaceholder"` |
| `m_PlaceholderIndex` | `PlaceholderIndex` | `int` | — | i18n 模板用 `{0}` 替換 |
| `m_PreviewDescription` | `PreviewDescription` | `string` | — | 預設空字串；非空則覆寫描述 |

### A.3 重要 Method 摘要
*   **`GetDescription`**：`m_PreviewDescription` 非空 → 直接返回；否則 `UCL_LocalizeManager.Get(m_PlaceholderKey, m_PlaceholderIndex)`。
*   **`GetDescriptionShort / GetShortName / GetDescriptionFormat`**：均代理到 `GetDescription`。
*   **`GetFusionCandidateSettings`** → 空清單。
*   **`GetFusionBaseSetting`** → `this`。
*   **`AddAction`** → 空實作。

### A.4 與其他系統的互動
*   **融合系統的 `GetFusionBaseSetting`**：父類 `RCG_BattleSetting.GetFusionBaseSetting` 預設回傳 `new RCG_PlaceholderSetting()`，所有未覆寫的子類在融合預覽時都會生成 placeholder。
*   **`RCG_DiminishedPlaceholder`**：繼承自此類；削弱系統的特化。
