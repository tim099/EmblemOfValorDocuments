---
title: 結束戰鬥 說明
description: 立即結束當前戰鬥；可指定結算類型（逃跑、勝利、失敗等）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 結束戰鬥

> 程式類別名稱：`RCG_BattleEndSetting`

## 用途
**直接觸發結束戰鬥**。最常見的用途是「**逃跑**」卡 — 玩家手中的逃跑卡使用此設定即可離開戰鬥。

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **WinState** | 是 | 結算類型：`Flee`（逃跑）/ `Win`（勝利）/ `Lose`（失敗）等。詳見 `WinState` enum。 |

## 行為說明
*   **可玩判定**：唯有「**敵人類型允許逃跑**」（`m_EnemyType.GetData().m_CanFlee`）時這張卡才可打出。例如 Boss 戰可能設為不可逃。
*   **觸發**：產生 `RCG_BattleEndAction(WinState)` 並插入佇列；戰鬥流程會依 WinState 走對應的結算腳本。
*   描述會直接顯示 `WinState` 對應的 i18n 描述（例如「逃跑」「勝利」）。

## 注意事項
*   **Boss / 強制戰鬥場合**：別意外讓玩家用此設定逃出來；Boss 戰請設定 `EnemyType.m_CanFlee = false`。
*   **Win / Lose 慎用**：通常戰鬥勝負由 HP 結算自動判定；只有特殊劇情（自動觸發失敗、強制勝利）才該手動設定。
*   **卡片描述顯示「不可逃」狀態**：可玩判定為 false 時卡片會灰掉，但描述仍會顯示「逃跑」 — 玩家會疑惑為何不能用，請在卡片附加註記。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_BattleEndSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **`[System.Serializable]`** 標記
*   **i18n 類別名 key**：`RCG_BattleEndSetting` → 「結束戰鬥」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_WinState` | `WinState` | `WinState` (enum) | — | 預設 `Flee` |

### A.3 重要 Method 摘要
*   **`CheckPlayable`** → `CanFlee`，依 `RCG_BattleManager.Ins.m_BattleData.m_EnemyType.GetData().m_CanFlee`；無 BattleManager 時預設 `true`。
*   **`GetDescriptionFormat / GetDescriptionShort`** 都返回 `m_WinState.GetLocalizeDes()`。
*   **`AddAction`** → `iData.AddAction(new RCG_BattleEndAction(m_WinState), iAddActionMode)`，但僅 `CanFlee` 時才執行（**已在可玩判定擋過一次，這裡是雙重保險**）。

### A.4 與其他系統的互動
*   **`RCG_BattleEndAction`**：實際結算 Action。
*   **`RCG_EnemyType.m_CanFlee`**：Boss / 特殊敵類禁止逃跑的開關。
