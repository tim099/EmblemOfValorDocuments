---
title: 消耗狀態 說明
description: 消耗自身狀態層數觸發效果；層數不足時可選擇阻擋使用或不觸發
last_updated: 2026-05-05
target_audience: [Designer, Modder, AI_Agent]
---

# 消耗狀態

> 程式類別名稱：`RCG_ConsumeStatusSetting`

## 用途
**消耗自身狀態層數來觸發效果**。例如「消耗 1 層劍氣護體 → 造成 20 點物傷」。常見用途：
*   建立「累積 + 爆發」風格的卡牌組合
*   讓某些強力效果需要事前蓄能

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **狀態** (`Status`) | 是 | 要消耗的狀態類型（`RCG_CustomStatusGenData`）。 |
| **值** (`Amount`) | 是 | 消耗多少層；支援變數。 |
| **Setting** | 是 | 消耗成功後執行的「**組合效果**」。 |
| **詳細設定** (`DetailSetting`) | 否 | 與「狀態」設定的詳細設定共用（控制動畫、Tag 等）。 |
| **CheckPlayable** | — | 預設不打勾。打勾 = 層數不足時**不可打出**；不打勾 = 仍可打出但不觸發效果。 |

## 行為說明
*   **層數判定**：以 `iData.User`（自身）的當前狀態層數比對 `Amount`。
*   **CheckPlayable 模式**：
    *   打勾：層數不足時這張卡灰掉不可用。
    *   不打勾：層數不足時卡片仍可打出，但 `m_Setting` 不會觸發。
*   **觸發**：扣除指定層數（`-amount`）→ 執行 `m_Setting`。
*   **預覽傷害**：條件成立時取 `m_Setting` 的預覽；不成立時回傳 0（**非 -1**，差別在於不被視為「非攻擊牌」）。
*   描述格式：「**消耗 {Amount} 層 {Status} 圖示**」+ 子設定描述（i18n key `ConsumeStatusDes`）。

## 注意事項
*   **預覽傷害不準的情況**：若狀態層數在卡片計算時是動態的（例如「先抽牌再消耗」），預覽可能與實際不符。
*   **Status 必須是可堆疊的狀態**：對於不支援層數的狀態，行為未定義。
*   **與「狀態」設定的差別**：「狀態」是給予層數；本設定是**消耗自身**層數。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：[`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ConsumeStatusSetting.cs`](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ConsumeStatusSetting.cs)
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_ConsumeStatusSetting` → 「消耗狀態」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_Status` | 狀態 | `RCG_CustomStatusGenData` | `Status` | |
| `m_Amount` | 值 | `IntVariable` | `Amount` | 預設 1 |
| `m_Setting` | `Setting` | `RCG_CombineSetting` | — | 子效果容器 |
| `m_DetailSetting` | 詳細設定 | `RCG_StatusSetting.DetailSetting` | `DetailSetting` | 共用「狀態」設定的 DetailSetting |
| `m_CheckPlayable` | `CheckPlayable` | `bool` | — | 預設 `false` |

### A.3 重要 Method 摘要
*   **`CheckCondition(iData, out amount)` (public)**：對 `iData.User.m_UnitStatus.GetStatusLayer(m_Status)` 比對 `m_Amount.GetValue(iData)`。
*   **`CheckPlayable`**：`m_CheckPlayable` 為 false 時恆為 true；否則跑 `CheckCondition`。
*   **`AddAction`**：條件成立 → `CreateAction.StatusAction(User, Status, -amount, ..., DetailSetting)` + `m_Setting.AddAction(InsertInOrder)`。
*   **`GetBattleSettings<T> / (Type)`**：自身 + 遞迴 `m_Setting`。
*   **`GetDescriptionFormat`**：暫存 `m_FullSentence = false` → i18n key `ConsumeStatusDes` + 子設定 format → 復原並套句尾。
*   **`GetPreviewDamage`**：條件不成立回 0（**非 -1**）；成立則代理到 `m_Setting`。
*   **`PreloadData`**：await `m_Setting.PreloadData`。

### A.4 與其他系統的互動
*   **`RCG_UnitStatus.GetStatusLayer(status)`**：當前層數查詢入口。
*   **`CreateAction.StatusAction`**：實際扣層數的 Action 工具。
*   **`RCG_CustomStatusGenData.IconTMPKey`**：描述中內嵌的圖示。
