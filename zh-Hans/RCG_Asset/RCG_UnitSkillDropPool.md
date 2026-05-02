---
title: 单位技能掉落池 (RCG_UnitSkillDropPool) 说明
description: 定义「升级时可抽到哪些单位技能」的资料；角色升级、技能解锁选单背后的池子
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 单位技能掉落池

> 程式类别名称：`RCG_UnitSkillDropPool`

## 用途

定义「**升级或选择技能时，会抽到哪些 `RCG_UnitSkillData`**」。例如角色升级时会给玩家三选一的技能候选，这些候选就从某个 `RCG_UnitSkillDropPool` 抽。

继承自 `RCG_Asset<RCG_UnitSkillDropPool>`，实作介面：`UCL.Core.UCLI_ShortName`。

## 编辑器中的样貌

```
RCG_UnitSkillDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ DropPool / MixDropPools / FilterDropData
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **DropType** | 是 | `DropPool` / `MixPool` / `FilterDrop` |
| **DropPool** | DropType=DropPool | 技能清单 + 权重 |
| **MixDropPools** | DropType=MixPool | 混合其他池子 |
| **FilterDropData** | DropType=FilterDrop | 用内部 `DropFilter` 筛 |

## 行为说明

与其他 Drop Pool 同骨架。

## 注意事项

*   enum 名称叫 `UnitSkillDropType`（没有 `E` 前缀也没叫 `DropType`），与其他 Drop Pool 命名不一致；历史遗留差异。
*   结构与 `RCG_CardDropPool` 类似，差异只在掉落目标换成 `RCG_UnitSkillData`。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_UnitSkillDropPool.cs`
*   **继承自**：`RCG_Asset<RCG_UnitSkillDropPool>`
*   **AssetGroup**：`EditDropSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_DropPool` | 掉落池 | `RCG_CommonDropSetting<RCG_UnitSkillGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_FilterDropData` | 条件式 | `FilterDropData` | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `UnitSkillDropType` enum | 预设 `DropPool` |

### A.3 重要 Method

*   **`GetDrops(int)`** / **`GetDropsWithFilterFunc(int, Func)`** — 抽 N 个技能。
*   **`GetDropRate(...)`** — 标准化权重表。

### A.4 与其他系统的互动

*   **`RCG_UnitSkillData`** — 掉落目标。
*   **`RCG_UnitSkillGenData`** / **`RCG_UnitSkillDropPoolGenData`** — Asset Entry 包装。
