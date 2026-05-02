---
title: 戰鬥標籤 說明
description: 為效果附加戰鬥標籤（如「易碎」「敏捷」），不直接產生動作而是改變上層設定的描述與 Tag 聚合
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 戰鬥標籤

> 程式類別名稱：`RCG_BattleTagSetting`

## 用途
**附加戰鬥標籤**到效果上 — 自身**不產生任何戰鬥動作**，純粹做為「**標籤載體**」。常用於：
*   讓組合效果獲得「**易碎**」「**敏捷**」「**消耗**」等屬性標籤
*   作為「卡片資訊」面板上的詞條來源

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **BattleTags** | 是 | 戰鬥標籤清單，每項是 `RCG_BattleTagGenData`（標籤模板）。可含多個。 |

## 行為說明
*   **不產生 Action**：這個設定的 `AddAction` 是空實作 — 它只用來宣告標籤。
*   **卡片資訊**：每個標籤會在卡片懸停 Tooltip 上顯示自己的名稱與描述；可堆疊的標籤（如 `m_IsStackable = true`）描述中會帶層數。
*   **短描述**：直接串接所有標籤的縮寫名（沒有額外修飾）。
*   **`GetBattleTags()`**：上層的「組合效果」「迴圈」等容器會透過此方法**聚合所有子設定的標籤**統一顯示。

## 注意事項
*   **要產生實際效果請放在「組合效果」或「狀態」**：戰鬥標籤本身只是 metadata；想要「易碎」帶來的減傷效果，得另外設定觸發邏輯。
*   **空清單**：合法但無意義；會佔資料空間又不影響任何行為。
*   **永遠展開**：此設定有 `[AlwaysExpendOnGUI]` 屬性，Inspector 中**永遠不會折疊** — 直接展示標籤清單。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_BattleTagSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **`[System.Serializable]` + `[UCL.Core.ATTR.AlwaysExpendOnGUI]`** 標記
*   **i18n 類別名 key**：`RCG_BattleTagSetting` → 「戰鬥標籤」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_BattleTags` | `BattleTags` | `List<RCG_BattleTagGenData>` | — | |

### A.3 重要 Method 摘要
*   **`AddAction`** → 空實作（僅 `// base.AddAction(...)` 被註解掉）。
*   **`GetBattleTags()`** → `m_BattleTags.Clone()`；上層容器透過此方法聚合。
*   **`Infos`**：每個 tag 組出 `{LocalizedName}\n{Description}`，可堆疊類型加上 `StackedBattleTag` 模板（含 `eBattleVariable_BattleTagStackCount` 變數）。
*   **`GetShortName`** → 串接 `m_BattleTags[i].GetShortName()`；空清單回退到 base。
*   **`GetDescription`** → 永遠回傳 `string.Empty`（被刻意覆寫）。

### A.4 與其他系統的互動
*   **`RCG_BattleTagGenData / RCG_BattleTag`**：標籤模板與實例。
*   **i18n keys**：`StackedBattleTag` / `eBattleVariable_BattleTagStackCount` / tag 顏色 `RCG_BattleTag`。
*   **被誰呼叫 `GetBattleTags`**：所有容器類設定（CombineSetting / LoopSetting / ConditionalSetting / ForeachTargetSetting 等）。
