---
title: Diminished（已被削弱占位）說明
description: 「無效卡牌效果」削弱後的占位類別；顯示紅字提示且不可再被削弱
last_updated: 2026-05-05
target_audience: [Designer, Modder, AI_Agent]
---

# Diminished（已被削弱占位）

> 程式類別名稱：`RCG_DiminishedPlaceholder`

## 用途
**「無效卡牌效果」（`RCG_DiminishSetting`）削弱後的佔位類別**。原本的葉效果（例如「攻擊」）會被替換為這個 placeholder，在描述中顯示**紅字「已被削弱」**。

> [!IMPORTANT]
> 這個設定**通常不會由你手動建立** — 它由削弱系統在執行時自動替換進來。如果你在資料中看到它，代表這張卡曾經被削弱過。

## 編輯器中的樣貌
通常不會出現在新建設定的下拉中（雖然 `AllTypes` 有列），出現時欄位空無一物 — 它就是個視覺 marker。

## 主要欄位
（無）

## 行為說明
*   觸發時什麼也不做（純 placeholder）。
*   描述永遠顯示為**已被削弱**（紅字，`RCG_Extensions.TagColors.Diminished`）。
*   作為「葉節點」存在，**不可再被削弱**（融合候選為空）。

## 注意事項
*   **不要在資料中手動建立**：除非你想故意預先標記某個位置為「已削弱」。
*   **與「Placeholder」的差別**：`RCG_PlaceholderSetting` 是**融合系統**用的中性占位（藍色框）；`RCG_DiminishedPlaceholder` 是**削弱系統**用的（紅字）。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：[`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_DiminishedPlaceholder.cs`](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_DiminishedPlaceholder.cs)
*   **繼承自**：`RCG_PlaceholderSetting`
*   **無 i18n 類別名 key**：編輯器顯示 stripped name `Diminished`

### A.2 欄位對照
（無自有欄位）

### A.3 重要 Method 摘要
*   **`GetDescription / GetDescriptionShort / GetShortName`**：均回傳 `UCL_LocalizeManager.Get("DiminishedPlaceholder").GetTagColor(TagColors.Diminished)`。
*   **`GetFusionCandidateSettings`** → 空清單（不能作為融合候選）。
*   **`GetFusionBaseSetting`** → `this`（保留自身為 base，**不會被進一步替換**）。

### A.4 與其他系統的互動
*   **`RCG_DiminishSetting.AddAction`**：觸發此 placeholder 的替換流程。
*   **`RCG_CardBattleData.Diminish`**：執行替換的入口。
*   **`RCG_Extensions.TagColors.Diminished`**：紅字色票來源。
*   **i18n key `DiminishedPlaceholder`**：顯示文字（「已被削弱」之類）。
