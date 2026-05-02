---
title: 选取目标 说明
description: 重新选取目标（玩家点击式选择）；会覆盖原本的目标清单
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 选取目标

> 程式类别名称：`RCG_SelectTargetSetting`

## 用途
**让玩家重新选取攻击 / 效果目标**（弹出选择 UI）。后续设定可使用新的目标清单。常见用途：
*   「先选一个目标，再对其造成 5 倍伤害」
*   配合复杂效果让玩家手动指定对象

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **TargetType** | 是 | 可选择的目标类型（`TargetType` enum，例如 All / Enemy / Ally / SingleEnemy 等）。 |

## 行为说明
*   执行时：解锁战斗输入 → 开启目标选择 UI → 玩家点击后填入 `iData.Targets` → 重新锁回。
*   描述为 `TargetType` 的本地化字串。
*   选择期间战斗动作暂停，玩家必须点击才继续。

## 注意事项
*   **AI 触发无效**：此设定依赖玩家输入；AI 触发时会卡住战斗。**AI 改用「**设定目标**」**(`RCG_SetTargetSetting`)。
*   **覆盖原目标**：选完后 `iData.Targets` 被覆盖；之后子设定全部以新目标为准。
*   **与「设定目标」的差别**：本设定**有 UI 让玩家选**；「设定目标」是**自动套规则**。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_SelectTargetSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_SelectTargetSetting` → 「选取目标」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_TargetType` | `TargetType` | `TargetType` (enum) | — | 预设 `All` |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：`RCG_BattleManager.Ins.UnBlockAction(true)` → `await RCG_BattleField.Ins.SelectTargetsAsync(m_TargetType, false, iToken)` → `iData.SetTargets(targets)` → `BlockAction()`。
*   **`GetDescriptionFormat`**：直接显示 `m_TargetType.GetDescription()`。

### A.4 与其他系统的互动
*   **`RCG_BattleField.Ins.SelectTargetsAsync`**：UI 选目标入口。
*   **`RCG_BattleManager.UnBlockAction / BlockAction`**：战斗输入锁控制。
*   **`TargetType` enum**：可选择的目标类型集合。
