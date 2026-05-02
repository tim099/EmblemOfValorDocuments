---
title: 戰鬥狀態資料 (RCG_BattleStateData) 說明
description: 戰鬥的「階段」(state) 定義：進入此狀態時觸發哪些行為、下一個狀態是什麼
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-en
---

> [!WARNING]
> Translation pending — this file needs an English translation.
The original zh-Hant content is included below for reference.


# 戰鬥狀態資料

> 程式類別名稱：`RCG_BattleStateData`

## 用途

**戰鬥的「階段 (state)」定義**——例如「玩家回合」「敵方回合」「傷害結算」「回合結束」。每個 state 進入時會跑一段固定的行為（抽牌、處理 buff 結算、觸發回合事件等）。整個戰鬥就是在這些 state 之間流轉。

繼承自 `RCG_Asset<RCG_BattleStateData>`。

## 編輯器中的樣貌

```
RCG_BattleStateData: <ID>
    Name                ← 顯示名（多語系）
    NextState           ← 下一個 state 的 ID
    EnterStateActions   ← 進入此 state 時要跑的 BattleSetting 序列
    StateData           ← state 自己的 runtime 資料
    BattleState         ← 對應的 BattleManager.BattleState 列舉值
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Name** | 否 | 顯示名（多語系）；空白時 fallback 到 ID |
| **NextState** | 是 | 此 state 結束後切到哪個 state；構成狀態機骨架 |
| **EnterStateActions** | 否 | 進入此 state 時依序觸發的 `RCG_BattleSetting` 序列 |
| **StateData** | — | state 內部的 `RCG_RuntimeData`（變數儲存） |
| **BattleState** | — | 對應的 `RCG_BattleManager.BattleState` enum 值（None / PlayerTurn / EnemyTurn 等） |

## 行為說明

### 狀態機流程
1. 進入此 state → 呼叫 `OnEnterState(data)`，依序套用 `EnterStateActions`（過濾掉 `Enable=false` 的）。
2. State 持續期間 → 由外部驅動 `StateUpdate()`（目前為空）。
3. 結束時 → `ExitState()`（目前為空）。
4. 切到 `m_NextState`。

### State 與 BattleState 的對應
`State` (property) 從 `m_BattleState.DefaultValue` 字串轉成 `RCG_BattleManager.BattleState` enum；缺漏時回 `None`。**這個映射讓資料驅動的 state 能對應到程式內部的 enum 邏輯**。

## 注意事項

*   **`StateUpdate` / `ExitState` 是空實作**：目前狀態機由外部驅動，這兩個 hook 留作未來擴充。
*   **`m_StateActions` 已棄用**：被 `m_EnterStateActions` 取代；舊資料反序列化時會走 base 流程，**不會自動遷移**（程式內 `m_EnterStateActions = m_StateActions.Clone();` 已註解）。
*   **`m_State` enum 欄位已棄用**：改用 `m_BattleState`（`RCG_RuntimeDataConst`）；舊欄位仍存在於 base class 內，但不再寫入。
*   **`EnterStateActions` 過濾**：會過濾掉 `Enable=false` 的 setting，動態啟停可在編輯時調整。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_BattleStateData.cs`
*   **繼承自**：`RCG_Asset<RCG_BattleStateData>`
*   **AssetGroup**：`EditBattleSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_NextState` | NextState | `RCG_BattleStateGenData` | |
| `m_EnterStateActions` | EnterStateActions | `List<RCG_BattleSetting>` | |
| `m_StateData` | StateData | `RCG_RuntimeData` | |
| `m_BattleState` | BattleState | `RCG_RuntimeDataConst` | 預設 `BattleState` 類型 |

### A.3 重要 Method 摘要

*   **`OnEnterState(TriggerEffectData, AddActionMode)`** — 進入 state；套用所有 `EnterStateActions`。
*   **`StateUpdate()`** — 空殼，外部驅動。
*   **`ExitState()`** — 空殼。
*   **`State` (property)** — 從 `m_BattleState.DefaultValue` 取 enum 值。
*   **`LocalizeName`** — `m_Name.HasLocalize ? m_Name.Name : ID`。
*   **`EnterStateActions` (property)** — 過濾 `Enable=true` 的 action 清單。

### A.4 與其他系統的互動

*   **`RCG_BattleManager.BattleState` (enum)** — 對應的程式內部狀態。
*   **`RCG_BattleStateGenData`** — Asset Entry 包裝。
*   **`RCG_BattleSetting`** — `EnterStateActions` 的元素。
*   **`RCG_RuntimeData / RCG_RuntimeDataConst`** — 自帶資料容器。

### A.5 已知議題

*   `m_StateActions` 與 `m_State` enum 已棄用，反序列化沒有遷移路徑。