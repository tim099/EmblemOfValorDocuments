---
title: 改变队伍顺序 说明
description: 切换我方队伍的角色出战顺序（前排轮替或将指定角色拉前）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 改变队伍顺序

> 程式类别名称：`RCG_ChangePartyOrderSetting`

## 用途
**改变我方角色的排序**，让不同角色登场战斗。常见用途：
*   「换手卡」— 把第一名轮到最后
*   「拉前」— 把指定角色（通常是受到威胁的）拉到第一位

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **ChangePartyOrderType** | 是 | 排序变更方式：<br>• **FirstToLast** — 第一个角色排到最后（后轮轮替）<br>• **TargetToFirst** — 把选中角色拉到第一个 |
| **目标** (`Target`) | 是 | 目标选择器（`TargetToFirst` 模式必须是单体）。 |

## 行为说明
*   `FirstToLast` 直接呼叫 `RCG_BattleManager.Ins.SwitchCharacterUI.FirstToLast()`。
*   `TargetToFirst` 对单体目标：播放「光点」VFX 后切换 UI。
*   描述会根据 ChangePartyOrderType 套不同 i18n（`ChangePartyOrderDes_FirstToLast` / `ChangePartyOrderDes_TargetToFirst`）。

## 注意事项
*   **TargetToFirst 多目标**：选择器若回传 0 或多个目标，行为**不一定如预期**（程式只在 `aTargets.Count == 1` 时播 VFX，其他情况可能跳过）。请选择器设成「单体」。
*   **战斗节奏**：切换顺序通常代表「下一个行动者改变」，请确认后续触发点是否依赖正确的当前角色。
*   **永远展开**：此设定有 `[AlwaysExpendOnGUI]`，Inspector 中**永远不会折叠**。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ChangePartyOrderSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **`[System.Serializable] + [AlwaysExpendOnGUI]`** 标记
*   **i18n 类别名 key**：`RCG_ChangePartyOrderSetting` → 「改变队伍顺序」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_ChangePartyOrderType` | `ChangePartyOrderType` | `ChangePartyOrderType` (档内 enum) | — | `FirstToLast` / `TargetToFirst` |
| `m_Target` | 目标 | `RCG_SelectTargetData` | `Target` | 取代旧的 `m_Range`（已淘汰） |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：依 ChangePartyOrderType 分流。`TargetToFirst` 在 `aTargets.Count == 1` 时 `RCG_VFXManager.CreateVFX(VFX_LightTarget)` 并切换。
*   **`GetDescriptionShort`** → 直接回传 i18n key `RCG_ChangePartyOrderSetting`（=「改变队伍顺序」）作为精简标签。

### A.4 与其他系统的互动
*   **`RCG_BattleManager.Ins.SwitchCharacterUI`**：实际的队伍切换 UI / 逻辑。
*   **`CommonVFX.VFX_LightTarget`**：拉前的视觉效果。
