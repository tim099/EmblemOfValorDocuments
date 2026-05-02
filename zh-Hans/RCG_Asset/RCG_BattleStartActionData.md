---
title: 战斗开场动作 (RCG_BattleStartActionData) 说明
description: 战斗一开始就触发的特殊行为：偷袭伤害、晕眩、初始 buff 等
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 战斗开场动作

> 程式类别名称：`RCG_BattleStartActionData`

## 用途

**战斗刚开始时要触发的特殊行为**。例如「偷袭」可以在敌人行动前先扣血、「准备充足」可以给己方初始 buff、「魔法阵」可以 turn 0 就释放 AOE。每场战斗可以引用一个 `RCG_BattleStartActionData`，内含 `User`（谁来执行）+ 一连串 `RCG_BattleSetting`（要做什么）。

继承自 `RCG_Asset<RCG_BattleStartActionData>`。

## 编辑器中的样貌

```
RCG_BattleStartActionData: <ID>
    User                  ← 执行者选择规则
    BattleStartActions    ← 要执行的 BattleSetting 序列
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **User** | 是 | 执行者选择规则（`RCG_SelectTargetData`）。预设选 Ally / All 范围；空 → fallback 到玩家第一位 |
| **BattleStartActions** | 是 | 战斗开始时要按顺序执行的 `RCG_BattleSetting` 序列（伤害、buff、抽牌⋯） |

## 行为说明

### `AddAction(TriggerEffectData)`
1. 用 `m_User` 取得目标清单。
2. 取第一个目标当作 `iData.User`（找不到 → 玩家第一位）。
3. 把 `m_BattleStartActions` 全部依序加入动作伫列（`PushBack` 模式）。

### 预设 ID
`RCG_BattleStartActionGenData.DefaultID = "None"`：表示「无开场动作」。

## 注意事项

*   **`User` 是「执行者」不是「目标」**：每个 BattleSetting 内部会有自己的目标选择；`m_User` 只是决定**动作从谁的视角发动**。
*   **目标清单空时 fallback**：到 `RCG_BattleManager.PlayerBattleUnits[0]`；确保开场动作不会因 User 规则没命中而失效。
*   **`OnGUI` 已被整段注解**：目前只用 `Preview` 显示资料，编辑时用预设 `OnGUI` 流程绘制。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_BattleStartActionData.cs`
*   **继承自**：`RCG_Asset<RCG_BattleStartActionData>`
*   **AssetGroup**：`EditBattleSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_User` | User | `RCG_SelectTargetData` | 预设 `SelectTargetType.Range`，范围 `Ally + All` |
| `m_BattleStartActions` | BattleStartActions | `List<RCG_BattleSetting>` | 开场动作序列 |

### A.3 重要 Method

*   **`AddAction(TriggerEffectData)`** — 主入口；计算 User → 设定 iData.User → 把所有 actions 加入伫列。
*   **`DefaultAction`** (static) — `Util.GetData(RCG_BattleStartActionGenData.DefaultID)`，预设「None」。

### A.4 与其他系统的互动

*   **`RCG_SelectTargetData`** — 执行者选择规则。
*   **`RCG_BattleSetting`** — 各动作节点。
*   **`RCG_BattleManager.PlayerBattleUnits`** — 目标 fallback 来源。
*   **`AddActionMode.PushBack`** — 动作伫列加入方式。

### A.5 已知议题

*   `OnGUI` 完整实作已注解；改用基底类预设绘制。
