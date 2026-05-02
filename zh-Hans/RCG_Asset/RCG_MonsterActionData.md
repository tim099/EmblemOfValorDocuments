---
title: 怪物动作资料 (RCG_MonsterActionData) 说明
description: 怪物可使用的单一招式 / 动作模板：选择规则、效果、图示、描述
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 怪物动作资料

> 程式类别名称：`RCG_MonsterActionData`

## 用途

**怪物可使用的单一招式 / 动作模板**。例如「挥砍」「召唤从者」「自爆」。每个 Action 内含：
*   选目标规则（哪些单位可被选为目标）
*   效果（伤害、buff、召唤⋯）
*   图示
*   描述（自动 / 手动）

`RCG_UnitData` 的 `MonsterStates` 引用一组 `RCG_MonsterActionData` 作为该状态下可用招式池。

继承自 `RCG_Asset<RCG_MonsterActionData>`。

## 编辑器中的样貌

```
RCG_MonsterActionData: <ID>
    Action (m_Action)
        SelectUnitRule         ← 选目标规则
        Effect 结构            ← 实际效果
        Icon                   ← 预览图示
        OverrideDescription    ← 是否手动覆写描述
        UnitAction             ← 真正执行的动作
        Infos                  ← Tooltip 列出的附加资讯
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Action** | 是 | 内部 `RCG_MonsterAction`，包含此招式所有资料 |

`RCG_MonsterAction` 内部：

| 子栏位 | 说明 |
|---|---|
| **SelectUnitRule** | 选目标规则（`RandomEnemy` / `Friend` / 自身 / 站位 N 之类） |
| **OverrideDescription** | 是否使用 `m_UnitAction` 的描述覆盖自动描述 |
| **UnitAction** | 招式真正执行的动作（伤害公式、效果序列） |
| **Icon** | 预览 / Tooltip 显示用的图示 |
| **SkillLevel** | 招式等级（runtime 由 `RCG_MonsterLevelActionData.GetAction` 动态填入） |

## 行为说明

### 选目标 → 触发 → 描述
1. 战斗中怪物选定要使用此 Action 后，依 `SelectUnitRule` 找出目标。
2. 套用 `m_UnitAction` 的效果到目标。
3. UI Tooltip 上显示 `Icon` + `GetDescription()` + `Infos` 补充标签资讯。

### 自动 vs 覆写描述
`OverrideDescription = true` 时，预览会在主描述下面**额外**附加 `m_UnitAction.GetDescription()`（两段都显示）。

### 预设 ID
透过 `RCG_MonsterActionGenData.IdleID` (`"Idle"`) 取得「无动作」的预设 Action；任何没设动作的单位回合会跑这个。

## 注意事项

*   **SkillLevel 由外部填入**：`RCG_MonsterLevelActionData.GetAction(level)` 会把等级写到 `m_SkillLevel`，本资料只是模板，runtime 取的是 clone 后的副本。
*   **OverrideDescription 双显示**：可能造成 Tooltip 过长，必要时关闭。
*   **Idle 动作不要删**：许多 fallback 路径会回传 `RCG_MonsterActionGenData.Idle.GetData()`，缺了这个 ID 会 NRE。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_MonsterActionData.cs`
*   **继承自**：`RCG_Asset<RCG_MonsterActionData>`
*   **AssetGroup**：`EditBattleSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_Action` | Action | `RCG_MonsterAction` | 主要资料容器 |

### A.3 重要 Method 摘要

*   **`Preview`** — 编辑器渲染：图示 + 选目标规则 + 描述 + Effects 描述（若 OverrideDescription）+ Infos。
*   建构式预设 `ID = "New Action"`。

### A.4 与其他系统的互动

*   **`RCG_MonsterAction`** — 主要资料模型；含 `m_SelectUnitRule` / `m_UnitAction` / `m_Icon` / `m_SkillLevel`。
*   **`RCG_MonsterActionGenData`**（档内）— Asset Entry 包装；`Idle` 为系统预设。
*   **`RCG_MonsterLevelActionData`** — 对 Action 包装以支援等级分级。
*   **`RCG_UnitData.m_MonsterStates`** — 引用 Actions 的容器。

### A.5 已知议题

*   有被注解掉的 `DeserializeFromJson` 容错逻辑（`m_ID == "AttackEffect"` → `DefaultID`），标示旧版 ID 迁移历史。
