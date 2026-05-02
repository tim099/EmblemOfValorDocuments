---
title: 移动 说明
description: 移动单位到指定位置（前排 / 后排 / 对面 / 指定位置）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 移动

> 程式类别名称：`RCG_MoveSetting`

## 用途
**移动单位**到指定位置。常见用途：
*   「换位卡」— 把后排角色拉到前排
*   「冲锋」— 角色移动到敌方位置
*   「撤退」— 移动到后排避战

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **MoveType** | 是 | 移动方式：<br>• **ToOther** — 移动到另一排（前↔后切换，预设）<br>• **ToFront** — 移动到前排<br>• **ToBack** — 移动到后排<br>• **ToTarget** — 移动到目标位置（与 `TargetPositions` 配合） |
| **MoveTarget** | 是 | 要移动的单位选择器。 |

## 行为说明
*   `ToFront` / `ToBack` / `ToOther` — 对每个目标各自移动到对应排：
    *   `ToOther` → 在前排则去后排，反之亦然
    *   `ToFront` → 一律到前排
    *   `ToBack` → 一律到后排
*   `ToTarget` — 配合外部设定的 `TargetPositions`，**最多移动 N 个单位到 N 个位置**（按索引配对）。
*   描述格式：「**{MoveTarget} 移动 {MoveType}**」（i18n key `Move_Des`）。

## 注意事项
*   **死亡目标跳过**：每次移动前会检查 `IsNullOrDead`，避免操作死人。
*   **ToTarget 模式需要 `TargetPositions`**：通常前面要接「**设定目标**」(`RCG_SetTargetSetting`) 或类似设定填入 `iData.m_TriggerData.m_TargetPositions`，否则没地方可去。
*   **目标位置已有单位**：技术上由 `RCG_BattleField.MoveUnit` 决定 — 多半会推挤或交换位置。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_MoveSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_MoveSetting` → 「移动」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_MoveType` | `MoveType` | `UnitMoveType` (档内 enum) | `MoveType` (=「移动行为」) | 4 种模式 |
| `m_MoveTarget` | `MoveTarget` | `RCG_SelectTargetData` | — | |

### A.3 重要 Method 摘要
*   **`AddAction`**：依 `m_MoveType` 分流：
    *   `ToTarget` → `iData.m_TriggerData.m_TargetPositions` 与 `aTargets` 按索引配对，呼叫 `RCG_BattleField.Ins.MoveUnit(target, position, token)`。
    *   其他 → 对每个 target 计算 `aTargetPosition`（前排 / 后排 / 对面），同样 `MoveUnit`。
*   **`GetDescriptionShort`** → 直接显示移动 sprite (`EffectIcon.Move`)。

### A.4 与其他系统的互动
*   **`RCG_BattleField.Ins.MoveUnit(unit, pos, token)`**：实际的移动入口（含动画）。
*   **`UnitPos`**：前 / 后 enum。
*   **`iData.m_TriggerData.m_TargetPositions`**：`ToTarget` 模式的目标位置来源。
