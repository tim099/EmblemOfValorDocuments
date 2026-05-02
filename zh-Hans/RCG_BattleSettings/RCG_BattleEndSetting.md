---
title: 结束战斗 说明
description: 立即结束当前战斗；可指定结算类型（逃跑、胜利、失败等）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 结束战斗

> 程式类别名称：`RCG_BattleEndSetting`

## 用途
**直接触发结束战斗**。最常见的用途是「**逃跑**」卡 — 玩家手中的逃跑卡使用此设定即可离开战斗。

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **WinState** | 是 | 结算类型：`Flee`（逃跑）/ `Win`（胜利）/ `Lose`（失败）等。详见 `WinState` enum。 |

## 行为说明
*   **可玩判定**：唯有「**敌人类型允许逃跑**」（`m_EnemyType.GetData().m_CanFlee`）时这张卡才可打出。例如 Boss 战可能设为不可逃。
*   **触发**：产生 `RCG_BattleEndAction(WinState)` 并插入伫列；战斗流程会依 WinState 走对应的结算脚本。
*   描述会直接显示 `WinState` 对应的 i18n 描述（例如「逃跑」「胜利」）。

## 注意事项
*   **Boss / 强制战斗场合**：别意外让玩家用此设定逃出来；Boss 战请设定 `EnemyType.m_CanFlee = false`。
*   **Win / Lose 慎用**：通常战斗胜负由 HP 结算自动判定；只有特殊剧情（自动触发失败、强制胜利）才该手动设定。
*   **卡片描述显示「不可逃」状态**：可玩判定为 false 时卡片会灰掉，但描述仍会显示「逃跑」 — 玩家会疑惑为何不能用，请在卡片附加注记。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_BattleEndSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **`[System.Serializable]`** 标记
*   **i18n 类别名 key**：`RCG_BattleEndSetting` → 「结束战斗」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_WinState` | `WinState` | `WinState` (enum) | — | 预设 `Flee` |

### A.3 重要 Method 摘要
*   **`CheckPlayable`** → `CanFlee`，依 `RCG_BattleManager.Ins.m_BattleData.m_EnemyType.GetData().m_CanFlee`；无 BattleManager 时预设 `true`。
*   **`GetDescriptionFormat / GetDescriptionShort`** 都返回 `m_WinState.GetLocalizeDes()`。
*   **`AddAction`** → `iData.AddAction(new RCG_BattleEndAction(m_WinState), iAddActionMode)`，但仅 `CanFlee` 时才执行（**已在可玩判定挡过一次，这里是双重保险**）。

### A.4 与其他系统的互动
*   **`RCG_BattleEndAction`**：实际结算 Action。
*   **`RCG_EnemyType.m_CanFlee`**：Boss / 特殊敌类禁止逃跑的开关。
