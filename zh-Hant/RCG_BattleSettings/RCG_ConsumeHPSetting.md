---
title: 消耗生命值 說明
description: 對目標扣血（固定值或當前 / 最大 HP 百分比）；用於自損型效果
last_updated: 2026-05-05
target_audience: [Designer, Modder, AI_Agent]
---

# 消耗生命值

> 程式類別名稱：`RCG_ConsumeHPSetting`

## 用途
**對目標扣血**（不視為攻擊，因此**不受護甲影響**）。常見用途：
*   「自損 5 點 HP 換取大量資源」
*   「消耗 30% 當前 HP 觸發強力效果」

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **ConsumeType** | 是 | 消耗模式：<br>• **CurHPPercentage** — 當前 HP 的百分比（預設）<br>• **MaxHPPercentage** — 最大 HP 的百分比<br>• **Value** — 固定數值 |
| **值** (`Amount`) | 是 | 消耗量（變數可變）；百分比模式為 0~100。 |
| **目標** (`Target`) | 是 | 目標選擇器（通常選自身）。 |

## 行為說明
*   依 `ConsumeType` 計算實際扣血量：
    *   `CurHPPercentage` → `0.01 * Amount * 當前 HP` 取整
    *   `MaxHPPercentage` → `0.01 * Amount * 最大 HP` 取整
    *   `Value` → 直接 `Amount`
*   呼叫 `target.UnitHeal(-aConsumeAmount)` 扣血。
*   數值會記入 `RCG_BattleManager.Ins.m_BattleStats.LogStatsCostHealth(...)`。
*   描述格式：`ConsumeHP_{ConsumeType}_Des`（會出現心型圖示）。

## 注意事項
*   **百分比模式的取整**：使用 `RoundToInt`，邊界值（如 1% × 50 HP = 0.5）會四捨五入到 0 或 1。設計時請以**整數玩家可預測**為原則。
*   **不會直接擊殺**：扣血至 0 會觸發死亡判定，但若想「強制擊殺」請改用「**即死**」設定。
*   **不視為攻擊**：護甲、反擊、傷害修飾都不適用 — 純粹扣血。
*   **目標選敵方**：技術上合法（給敵方扣血），但這通常該用「攻擊」設定 — 兩者語意不同。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：[`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ConsumeHPSetting.cs`](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ConsumeHPSetting.cs)
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_ConsumeHPSetting` → 「消耗生命值」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_ConsumeType` | `ConsumeType` | enum | — | 3 種模式 |
| `m_Amount` | 值 | `IntVariable` | `Amount` | `[UCL_FieldOnGUI]`，預設 50 |
| `m_Target` | 目標 | `RCG_SelectTargetData` | `Target` | |

### A.3 重要 Method 摘要
*   **`AddAction`**：依 ConsumeType 計算 `aConsumeAmount`，呼叫 `aTarget.UnitHeal(-aConsumeAmount)`，最後 `LogStatsCostHealth`。
*   **`GetDescriptionFormat`**：i18n key `ConsumeHP_{ConsumeType}_Des`，內含心型 sprite。
*   **`GetDescriptionShort`** → 直接 `m_Amount.GetDes(true)`。

### A.4 與其他系統的互動
*   **`RCG_BattleUnit.UnitHeal(int)`**：負值代表扣血（共用治療入口）。
*   **`RCG_BattleManager.m_BattleStats.LogStatsCostHealth`**：戰鬥統計記錄。
