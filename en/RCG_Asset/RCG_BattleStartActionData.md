---
title: 戰鬥開場動作 (RCG_BattleStartActionData) 說明
description: 戰鬥一開始就觸發的特殊行為：偷襲傷害、暈眩、初始 buff 等
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-en
---

> [!WARNING]
> Translation pending — this file needs an English translation.
The original zh-Hant content is included below for reference.


# 戰鬥開場動作

> 程式類別名稱：`RCG_BattleStartActionData`

## 用途

**戰鬥剛開始時要觸發的特殊行為**。例如「偷襲」可以在敵人行動前先扣血、「準備充足」可以給己方初始 buff、「魔法陣」可以 turn 0 就釋放 AOE。每場戰鬥可以引用一個 `RCG_BattleStartActionData`，內含 `User`（誰來執行）+ 一連串 `RCG_BattleSetting`（要做什麼）。

繼承自 `RCG_Asset<RCG_BattleStartActionData>`。

## 編輯器中的樣貌

```
RCG_BattleStartActionData: <ID>
    User                  ← 執行者選擇規則
    BattleStartActions    ← 要執行的 BattleSetting 序列
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **User** | 是 | 執行者選擇規則（`RCG_SelectTargetData`）。預設選 Ally / All 範圍；空 → fallback 到玩家第一位 |
| **BattleStartActions** | 是 | 戰鬥開始時要按順序執行的 `RCG_BattleSetting` 序列（傷害、buff、抽牌⋯） |

## 行為說明

### `AddAction(TriggerEffectData)`
1. 用 `m_User` 取得目標清單。
2. 取第一個目標當作 `iData.User`（找不到 → 玩家第一位）。
3. 把 `m_BattleStartActions` 全部依序加入動作佇列（`PushBack` 模式）。

### 預設 ID
`RCG_BattleStartActionGenData.DefaultID = "None"`：表示「無開場動作」。

## 注意事項

*   **`User` 是「執行者」不是「目標」**：每個 BattleSetting 內部會有自己的目標選擇；`m_User` 只是決定**動作從誰的視角發動**。
*   **目標清單空時 fallback**：到 `RCG_BattleManager.PlayerBattleUnits[0]`；確保開場動作不會因 User 規則沒命中而失效。
*   **`OnGUI` 已被整段註解**：目前只用 `Preview` 顯示資料，編輯時用預設 `OnGUI` 流程繪製。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_BattleStartActionData.cs`
*   **繼承自**：`RCG_Asset<RCG_BattleStartActionData>`
*   **AssetGroup**：`EditBattleSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_User` | User | `RCG_SelectTargetData` | 預設 `SelectTargetType.Range`，範圍 `Ally + All` |
| `m_BattleStartActions` | BattleStartActions | `List<RCG_BattleSetting>` | 開場動作序列 |

### A.3 重要 Method

*   **`AddAction(TriggerEffectData)`** — 主入口；計算 User → 設定 iData.User → 把所有 actions 加入佇列。
*   **`DefaultAction`** (static) — `Util.GetData(RCG_BattleStartActionGenData.DefaultID)`，預設「None」。

### A.4 與其他系統的互動

*   **`RCG_SelectTargetData`** — 執行者選擇規則。
*   **`RCG_BattleSetting`** — 各動作節點。
*   **`RCG_BattleManager.PlayerBattleUnits`** — 目標 fallback 來源。
*   **`AddActionMode.PushBack`** — 動作佇列加入方式。

### A.5 已知議題

*   `OnGUI` 完整實作已註解；改用基底類預設繪製。