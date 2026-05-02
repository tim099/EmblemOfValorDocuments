---
title: 场地效果 (RCG_FieldEffectData) 说明
description: 作用于整个战场的环境效果模板（火焰场地、黑暗垄罩、各回合 -HP 等）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 场地效果

> 程式类别名称：`RCG_FieldEffectData`

## 用途

**作用于整个战场的环境效果模板**。不像状态 (CustomStatus) 是贴在单位身上，场地效果**全场通用**——例如「火焰场地：所有单位每回合 -3 HP」「黑暗：所有攻击命中率 -20%」「神圣：所有治疗 +50%」。

继承自 `RCG_Asset<RCG_FieldEffectData>`，实作介面：`RCGI_Infos` / `UI.RCGI_StatusInfo`。

## 编辑器中的样貌

```
RCG_FieldEffectData: <ID>
    Name(多国语言)
    Icon                 ← 场地效果图示
    Effects              ← 触发效果（依 trigger 套用到全场）
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Name** | 是 | 场地效果名（多语系） |
| **Icon** | 是 | 图示，预设 `FieldEffectIcons_volcano.png` |
| **Effects** | 是 | 各 trigger 上要触发的效果（`RCG_CommonEffect` list） |

## 行为说明

### 触发
`OnUnitState(triggerOn, data)` → 从 `m_Effects` 取对应 trigger 的 effects 并依序触发。**`OnUnitState` 命名是历史遗留**——实际上场地效果不一定绑定 unit state，可在任何 trigger 触发（OnBattleStart / OnTurnStart / OnTurnEnd 等）。

### 描述
自动由 `Effects` 串接（含 BattleTags 解说）。

## 注意事项

*   **没有层数机制**：场地效果是「有 / 没有」的二元状态，不像 CustomStatus 有层数。
*   **多个场地效果**可叠加（透过 BattleManager 管理当前生效列表）；UI 上会列出所有 active 场地。
*   **抽取来源**：`RCG_BattleSceneData.m_FieldEffectDrops` 引用 `RCG_FieldEffectDropPool`，战斗开始时抽出本场场地效果。
*   **未实作 RCGI_Status 介面**：类别宣告有 `//, RCGI_Status` 注解标示曾规划要实作但目前未做。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_FieldEffectData.cs`
*   **继承自**：`RCG_Asset<RCG_FieldEffectData>`
*   **实作介面**：`RCGI_Infos` / `UI.RCGI_StatusInfo`
*   **AssetGroup**：`EditBattleSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Icon` | Icon | `RCG_SpriteData` | 预设 `FieldEffectIcons_volcano.png` |
| `m_Effects` | Effects | `List<RCG_CommonEffect>` | |

### A.3 重要 Method 摘要

*   **`OnUnitState(triggerOn, data)`** — 触发效果（命名历史遗留）。
*   **`TriggerOnUnitState(triggerOn)`** — quick check。
*   **`Description` (property)** — `m_Effects.GetDescription(showBattleTags = true)`。
*   **`Infos` (property)** — `m_Effects.GetInfos()`。

### A.4 与其他系统的互动

*   **`RCG_BattleSceneData.m_FieldEffectDrops`** — 战斗场景引用的场地效果池。
*   **`RCG_FieldEffectDropPool`** — 场地效果随机池。
*   **`RCG_BattleManager.TriggerFieldEffect`** — runtime 触发场地效果的入口。
*   **`RCG_FieldEffectGenData`** / **`RCG_FieldEffectDropPoolGenData`** — Asset Entry 包装。

### A.5 已知议题

*   类别宣告 `, RCGI_Status` 是注解掉的——曾规划但未实作此介面。
*   `m_AcquireSkillEvents` 等程式内被注解，疑似从 UnitSkill 复制过来时残留的 dead code。
