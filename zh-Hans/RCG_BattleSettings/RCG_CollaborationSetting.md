---
title: 协力 说明
description: 队伍中符合专精需求的角色达到指定数量时触发效果；可作为条件触发或数量变数两种模式
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 协力

> 程式类别名称：`RCG_CollaborationSetting`

## 用途
**团队合击机制**：依「**我方有几个角色符合特定专精**」决定是否触发效果，或把符合数量作为变数使用。常见用途：
*   「若队伍中有 2 名魔法师，造成额外伤害」
*   「依符合人数决定治疗量」

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **CollaborationType** | 是 | 模式：<br>• **Default** — 达到 `CollaborationCount` 才触发<br>• **Variable** — 自动触发；符合人数作为变数 X |
| **CollaborationCount** | Default 模式 | 至少要有几个角色符合条件（预设 2）。 |
| **RequireSkills** | 否 | 专精需求清单；空清单代表不检查（任何队员都算）。 |
| **CollaborationVariable** | Variable 模式 | 变数名称（预设 `X`）；存入符合人数，后续设定可引用。 |
| **CollaborationSetting** | 是 | 触发后执行的「**组合效果**」。 |

## 行为说明
*   **Default 模式**：
    *   检查我方角色中符合 `RequireSkills`（任一即可）的数量是否 ≥ `CollaborationCount`。
    *   达标 → 执行 `CollaborationSetting`；未达标 → 不触发。
*   **Variable 模式**：
    *   不检查门槛，**直接触发**。
    *   把符合人数写入 `CollaborationVariable`，`CollaborationSetting` 内部可引用该变数做「依人数缩放」的效果。
*   描述包含：条件描述（i18n key `CollaborationTypeInfo_*`）+ 子设定描述。

## 注意事项
*   **RequireSkills 为空 + Default 模式**：等于「全员都算符合」，协力人数总是队伍人数 — 确认设计意图。
*   **变数命名冲突**：多个设定都用 `X`，后者会覆盖前者；用语意化的变数名（如 `CollabCount`）。
*   **触发子效果中可叠加新动作**：`CollaborationSetting` 是个 `RCG_CombineSetting`，可组合多种效果，但**注意巢状深度**对描述可读性的影响。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CollaborationSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_CollaborationSetting` → 「协力」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_CollaborationType` | `CollaborationType` | enum | — | `Default` / `Variable` |
| `m_CollaborationCount` | `CollaborationCount` | `int` | — | 预设 2 |
| `m_RequireSkills` | `RequireSkills` | `List<RCG_SkillTagGenData>` | — | |
| `m_CollaborationVariable` | `CollaborationVariable` | `string` | — | `[Conditional("m_CollaborationType", false, Variable)]`；预设 `"X"` |
| `m_CollaborationSetting` | `CollaborationSetting` | `RCG_CombineSetting` | — | `[AlwaysExpendOnGUI]` |

### A.3 重要 Method 摘要
*   **`Infos`**：自身 description（含 `GetCollaborationDes()` + `CollaborationTypeInfo_*` i18n）+ `m_CollaborationSetting.Infos`。
*   **`GetCollaborationRequireSkillDes()` (private)**：空清单 → `CollaborationRequireSkills_Any`；否则 `CollaborationRequireSkills_Specify`。
*   **`AddAction`**（档案 100 行外）：解析符合人数，依 CollaborationType 决定是否触发 `m_CollaborationSetting`。

### A.4 与其他系统的互动
*   **`RCG_SkillTagGenData`**：专精标签模板。
*   **`RCG_CombineSetting`**：子效果容器。
