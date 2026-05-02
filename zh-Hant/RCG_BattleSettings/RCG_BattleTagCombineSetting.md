---
title: BattleTagCombine（標籤組合）說明
description: 「組合效果」的特化版；以前綴戰鬥標籤包覆主效果，並可指定觸發時機
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# BattleTagCombine（標籤組合）

> 程式類別名稱：`RCG_BattleTagCombineSetting`

## 用途
「**組合效果**」的特化版。差別在於：**前面多綁了一組「戰鬥標籤」作為前綴條件**，搭配指定的觸發時機（`OnPlay` / `OnDraw` 等），讓主效果只在「**特定觸發時機 + 帶有指定標籤**」下生效。

實務上的用法：「若這張卡帶有【消耗】標籤，打出時觸發 X 效果」「若帶有【敏捷】標籤，抽牌時觸發 Y 效果」。

繼承自：`RCG_CombineSetting`（所以也有「描述」「OverrideDescription」「組合效果」清單；見「組合效果」說明）

## 主要欄位（在「組合效果」之外多兩個）

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **TriggerOn** | 是 | 觸發時機：`OnPlay`（打出時，預設）/ `OnDraw`（抽牌時）/ ... 詳見 `TriggerOn` enum。 |
| **PrefixBattleTagSetting** | 是 | 前綴的「戰鬥標籤」設定 — 主效果只在這些標籤被觸發時跑。 |

加上「組合效果」基底的：**OverrideDescription** / **描述** / **組合效果（子設定清單）**。

## 行為說明
*   `CombineSettings` 會把 `PrefixBattleTagSetting` 放在子設定清單**最前面**作為「條件」，後面接原本的 `組合效果` 清單。
*   描述顯示為「**若 {前綴標籤}，則 {主效果}**」（i18n key `IfCardConditions2` + `ConditionFitThenDes`）。
*   觸發時：先對 `PrefixBattleTagSetting` 的每個標籤呼叫 `OnTriggerEffect(TriggerOn)`，**再執行**剩下的子設定。
*   `GetBattleTags()` **忽略** prefix 標籤 — 避免標籤被當作主效果的一部分聚合。

## 注意事項
*   **TriggerOn 必須與卡片實際觸發點對齊**：若卡片本身在 `OnPlay` 觸發，但這裡 TriggerOn 設成 `OnDraw`，主效果永遠不會跑。
*   **PrefixBattleTagSetting 為空**：等於「無前置條件」，描述會變成「若 ，則 …」 — 屬於資料錯誤。
*   **與一般「條件判斷」的差別**：「條件判斷」是依 `RCG_Condition` 判斷是否符合；本設定則是**讓特定觸發時機 + 標籤**作為前置條件，語意較精準但用途較窄。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_BattleTagCombineSetting.cs`
*   **繼承自**：`RCG_CombineSetting`
*   **無 i18n 類別名 key**：編輯器顯示 stripped name `BattleTagCombine`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_TriggerOn` | `TriggerOn` | `TriggerOn` (enum) | — | 預設 `OnPlay` |
| `m_PrefixBattleTagSetting` | `PrefixBattleTagSetting` | `RCG_BattleTagSetting` | — | 嵌套的標籤設定 |
| (繼承) `m_OverrideDescription` / `m_Description` / `m_CombineSettings` | 同「組合效果」 | — | — | 見父類 |

### A.3 重要 Method 摘要
*   **`CombineSettings` (override)**：把 `m_PrefixBattleTagSetting` 插在最前 + base.CombineSettings。
*   **`GetBattleTags`** (override)：**跳過** prefix，只聚合 `m_CombineSettings.GetEnableBattleSettings()` 的標籤。
*   **`AddAction`**：對 `i==0` 的元素（prefix）改呼叫各 BattleTag 的 `OnTriggerEffect(m_TriggerOn, iData)`；其餘正常 `AddAction(InsertInOrder)`。
*   **`GetDescriptionFormat / GetDescription / GetDescriptionShort`**：以 `IfCardConditions2` + `ConditionFitThenDes` 包裝 prefix 描述與主描述。

### A.4 與其他系統的互動
*   **`RCG_BattleTagSetting`**：作為 prefix 容器。
*   **`RCG_EffectTriggerOn.GetEffectTriggerOn(TriggerOn)`**：把 `TriggerOn` enum 轉成可觸發的物件。
*   **i18n keys**：`IfCardConditions2` / `ConditionFitThenDes`。

### A.5 已知議題
*   未本地化於 `AllTypes` 之外，子類選單顯示為 stripped name。
