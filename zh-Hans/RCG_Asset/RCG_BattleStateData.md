---
title: 战斗状态资料 (RCG_BattleStateData) 说明
description: 战斗的「阶段」(state) 定义：进入此状态时触发哪些行为、下一个状态是什么
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 战斗状态资料

> 程式类别名称：`RCG_BattleStateData`

## 用途

**战斗的「阶段 (state)」定义**——例如「玩家回合」「敌方回合」「伤害结算」「回合结束」。每个 state 进入时会跑一段固定的行为（抽牌、处理 buff 结算、触发回合事件等）。整个战斗就是在这些 state 之间流转。

继承自 `RCG_Asset<RCG_BattleStateData>`。

## 编辑器中的样貌

```
RCG_BattleStateData: <ID>
    Name                ← 显示名（多语系）
    NextState           ← 下一个 state 的 ID
    EnterStateActions   ← 进入此 state 时要跑的 BattleSetting 序列
    StateData           ← state 自己的 runtime 资料
    BattleState         ← 对应的 BattleManager.BattleState 列举值
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Name** | 否 | 显示名（多语系）；空白时 fallback 到 ID |
| **NextState** | 是 | 此 state 结束后切到哪个 state；构成状态机骨架 |
| **EnterStateActions** | 否 | 进入此 state 时依序触发的 `RCG_BattleSetting` 序列 |
| **StateData** | — | state 内部的 `RCG_RuntimeData`（变数储存） |
| **BattleState** | — | 对应的 `RCG_BattleManager.BattleState` enum 值（None / PlayerTurn / EnemyTurn 等） |

## 行为说明

### 状态机流程
1. 进入此 state → 呼叫 `OnEnterState(data)`，依序套用 `EnterStateActions`（过滤掉 `Enable=false` 的）。
2. State 持续期间 → 由外部驱动 `StateUpdate()`（目前为空）。
3. 结束时 → `ExitState()`（目前为空）。
4. 切到 `m_NextState`。

### State 与 BattleState 的对应
`State` (property) 从 `m_BattleState.DefaultValue` 字串转成 `RCG_BattleManager.BattleState` enum；缺漏时回 `None`。**这个映射让资料驱动的 state 能对应到程式内部的 enum 逻辑**。

## 注意事项

*   **`StateUpdate` / `ExitState` 是空实作**：目前状态机由外部驱动，这两个 hook 留作未来扩充。
*   **`m_StateActions` 已弃用**：被 `m_EnterStateActions` 取代；旧资料反序列化时会走 base 流程，**不会自动迁移**（程式内 `m_EnterStateActions = m_StateActions.Clone();` 已注解）。
*   **`m_State` enum 栏位已弃用**：改用 `m_BattleState`（`RCG_RuntimeDataConst`）；旧栏位仍存在于 base class 内，但不再写入。
*   **`EnterStateActions` 过滤**：会过滤掉 `Enable=false` 的 setting，动态启停可在编辑时调整。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_BattleStateData.cs`
*   **继承自**：`RCG_Asset<RCG_BattleStateData>`
*   **AssetGroup**：`EditBattleSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_NextState` | NextState | `RCG_BattleStateGenData` | |
| `m_EnterStateActions` | EnterStateActions | `List<RCG_BattleSetting>` | |
| `m_StateData` | StateData | `RCG_RuntimeData` | |
| `m_BattleState` | BattleState | `RCG_RuntimeDataConst` | 预设 `BattleState` 类型 |

### A.3 重要 Method 摘要

*   **`OnEnterState(TriggerEffectData, AddActionMode)`** — 进入 state；套用所有 `EnterStateActions`。
*   **`StateUpdate()`** — 空壳，外部驱动。
*   **`ExitState()`** — 空壳。
*   **`State` (property)** — 从 `m_BattleState.DefaultValue` 取 enum 值。
*   **`LocalizeName`** — `m_Name.HasLocalize ? m_Name.Name : ID`。
*   **`EnterStateActions` (property)** — 过滤 `Enable=true` 的 action 清单。

### A.4 与其他系统的互动

*   **`RCG_BattleManager.BattleState` (enum)** — 对应的程式内部状态。
*   **`RCG_BattleStateGenData`** — Asset Entry 包装。
*   **`RCG_BattleSetting`** — `EnterStateActions` 的元素。
*   **`RCG_RuntimeData / RCG_RuntimeDataConst`** — 自带资料容器。

### A.5 已知议题

*   `m_StateActions` 与 `m_State` enum 已弃用，反序列化没有迁移路径。
