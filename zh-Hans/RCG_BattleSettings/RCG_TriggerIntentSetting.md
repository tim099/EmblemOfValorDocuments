---
title: 触发意图 说明
description: 强制触发目标的下回合行动（攻击提前 / 模仿敌人技能等）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 触发意图

> 程式类别名称：`RCG_TriggerIntentSetting`

## 用途
**强制触发目标单位的下回合意图**（即头顶显示的攻击计划）。常见用途：
*   「强制敌人提前出招」（让玩家可以预判反击）
*   「模仿敌方技能」（复制目标的行动）
*   「触发后刷新意图」（触发完换新的攻击计划）

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **TriggerIntentType** | 是 | 触发类型：<br>• **Default** — 触发意图<br>• **TriggerAndRefresh** — 触发后刷新（换新意图）<br>• **ImitateIntention** — 模仿目标意图（让使用者执行目标的行动） |
| **目标** (`Target`) | 是 | 目标单位选择器；无 AI 的目标会被过滤。 |

## 行为说明
*   解析 `Target` 并过滤出有 AI 的目标，依 `BattleID` 排序确保稳定。
*   **Default / TriggerAndRefresh**：对每个目标插入 `RCG_IntentAction(target, type)` Action，**插入位置为 Last**（堆到伫列末端，等其他动作完成后执行）。
*   **ImitateIntention**：取**第一个目标**的 AI 当前 `Action.m_UnitAction`，让使用者（`iData.User`）作为发动者执行 — 模仿其攻击。

## 注意事项
*   **无 AI 目标自动跳过**：友军角色通常无 AI，本设定对友军无效。
*   **ImitateIntention 只取一个目标**：多目标选择器会取第一个，其他被忽略。
*   **与「清除意图」的对比**：清除是让敌人**重决策**；本设定是**强制执行当前意图**。
*   **Last 插入位置**：意图触发会**等其他动作完成后**才执行 — 与 `InsertInOrder` 不同，注意串接顺序。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_TriggerIntentSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_TriggerIntentSetting` → 「触发意图」
*   **同档顶层 enum**：`ETriggerIntentType` (`Default`, `TriggerAndRefresh`, `ImitateIntention`)

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_TriggerIntentType` | `TriggerIntentType` | `ETriggerIntentType` | — | 3 种类型 |
| `m_Target` | 目标 | `RCG_SelectTargetData` | `Target` | |

### A.3 重要 Method 摘要
*   **`AddAction`**：`m_Target.GetTargets(iData).Where(HasAI).OrderBy(BattleID)`，依 `m_TriggerIntentType` 分流：
    *   `ImitateIntention` → 取 `aTargets.FirstOrDefault().UnitAI.CurrentAction.m_UnitAction.AddAction(iData, InsertInOrder)`。
    *   其他 → 对每个 target `iData.AddAction(new RCG_IntentAction(target, m_TriggerIntentType), AddActionMode.Last)`。
*   **`GetDescriptionFormat`** → `m_TriggerIntentType.GetLocalizeDes(target)`。

### A.4 与其他系统的互动
*   **`RCG_BattleUnit.HasAI / UnitAI.CurrentAction`**：AI 系统的当前行动查询。
*   **`RCG_IntentAction`**：实际触发意图的 Action class。
*   **`AddActionMode.Last`**：伫列尾端插入，与 `InsertInOrder` 行为不同。
