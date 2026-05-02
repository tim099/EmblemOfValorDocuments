---
title: 選取目標 說明
description: 重新選取目標（玩家點擊式選擇）；會覆蓋原本的目標清單
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 選取目標

> 程式類別名稱：`RCG_SelectTargetSetting`

## 用途
**讓玩家重新選取攻擊 / 效果目標**（彈出選擇 UI）。後續設定可使用新的目標清單。常見用途：
*   「先選一個目標，再對其造成 5 倍傷害」
*   配合複雜效果讓玩家手動指定對象

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **TargetType** | 是 | 可選擇的目標類型（`TargetType` enum，例如 All / Enemy / Ally / SingleEnemy 等）。 |

## 行為說明
*   執行時：解鎖戰鬥輸入 → 開啟目標選擇 UI → 玩家點擊後填入 `iData.Targets` → 重新鎖回。
*   描述為 `TargetType` 的本地化字串。
*   選擇期間戰鬥動作暫停，玩家必須點擊才繼續。

## 注意事項
*   **AI 觸發無效**：此設定依賴玩家輸入；AI 觸發時會卡住戰鬥。**AI 改用「**設定目標**」**(`RCG_SetTargetSetting`)。
*   **覆蓋原目標**：選完後 `iData.Targets` 被覆蓋；之後子設定全部以新目標為準。
*   **與「設定目標」的差別**：本設定**有 UI 讓玩家選**；「設定目標」是**自動套規則**。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_SelectTargetSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_SelectTargetSetting` → 「選取目標」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_TargetType` | `TargetType` | `TargetType` (enum) | — | 預設 `All` |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：`RCG_BattleManager.Ins.UnBlockAction(true)` → `await RCG_BattleField.Ins.SelectTargetsAsync(m_TargetType, false, iToken)` → `iData.SetTargets(targets)` → `BlockAction()`。
*   **`GetDescriptionFormat`**：直接顯示 `m_TargetType.GetDescription()`。

### A.4 與其他系統的互動
*   **`RCG_BattleField.Ins.SelectTargetsAsync`**：UI 選目標入口。
*   **`RCG_BattleManager.UnBlockAction / BlockAction`**：戰鬥輸入鎖控制。
*   **`TargetType` enum**：可選擇的目標類型集合。
