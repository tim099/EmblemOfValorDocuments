---
title: 战斗组合掉落池 (RCG_BattleSetDropPool) 说明
description: 定义「这个地图节点会出现哪些战斗组合」的资料；地图事件、章节遭遇底层的随机池
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 战斗组合掉落池

> 程式类别名称：`RCG_BattleSetDropPool`

## 用途

定义「**踩到地图上某种战斗节点时，会抽到哪几种战斗组合**」。例如「普通遭遇」「精英遭遇」「Boss 战」分别是不同池子；池内列出可能的 `RCG_BattleSet`（每个 BattleSet 是一场具体的怪物配置）+ 权重。

继承自 `RCG_Asset<RCG_BattleSetDropPool>`，实作介面：`UCL.Core.UCLI_ShortName`。

## 编辑器中的样貌

```
RCG_BattleSetDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ DropPool / MixDropPools / FilterDropData
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **DropType** | 是 | `DropPool` / `MixPool` / `FilterDrop` |
| **DropPool** | DropType=DropPool | BattleSet 清单 + 权重 |
| **MixDropPools** | DropType=MixPool | 引用其他池子并指定权重 |
| **FilterDropData** | DropType=FilterDrop | 用内部 `DropFilter`（Tag / EnemyType / Operator）筛 |

## 行为说明

### 三种模式
与其他 Drop Pool 同。FilterDrop 条件支援「BattleSet 标签」与「敌人类型 (EnemyType)」两种维度。

### 隐性筛选
本类别**没有 unlock / skill 等 runtime 筛选**——直接以权重抽。

### 预览
按 `ShowDetail` 看最终掉落率。

## 注意事项

*   **enum 名称叫 `EDropType` 不是 `DropType`**：类别内注解明示「取名 DropType 会导致 GoogleSheet 同步语言档失败，先改名」。设计时不要试图改回。
*   **DropPool 模式下** 反序列化时会自动移除不存在的 BattleSet ID。
*   **MixPool 循环**会被截断成空（`iLayer > 10`）；池子掉空先检查混合链。
*   **无 Name 栏位**：本类别没有额外显示名称，`GetShortName()` 直接取第一个 drop 的名称。

---

## 附录：程式人员参考 (Programmer Reference)

> 此段以下使用程式内部术语，受众转为程式人员与 AI agent。前半段内容请优先采信。

### A.1 类别资讯

*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_BattleSetDropPool.cs`
*   **继承自**：`RCG_Asset<RCG_BattleSetDropPool>`
*   **实作介面**：`UCL.Core.UCLI_ShortName`
*   **AssetGroup**：`EditBattleSetting`（注意：与其他 DropPool 不同，这个归在 BattleSetting）

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_DropPool` | 掉落池 | `RCG_CommonDropSetting<RCG_BattleSetGenData>` | — | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | — | `Conditional(MixPool)` |
| `m_FilterDropData` | 条件式 | `FilterDropData` = `FilterDropDataBase<RCG_BattleSetGenData, RCG_BattleSet, DropFilter>` | — | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `EDropType` enum | `DropType` | 预设 `DropPool` |

### A.3 重要 Method 摘要

*   **`GetBattleSets(int)`** → 主要入口；无内建筛选，直接抽 N 个。
*   **`GetBattleSetsWithFilterFunc(int, Func)`** → 外部自订筛选版。
*   **`GetDropRate(CheckDropConditionData, int)`** → 预设 filter 是 `_ => true`。

### A.4 与其他系统的互动

*   **`RCG_BattleSet`** — 池子掉的目标型别。
*   **`RCG_BattleSetGenData`** / **`RCG_BattleSetDropPoolGenData`** — Asset Entry 包装；后者预设 ID = `"NormalBattle"`。
*   **`RCG_BattleSetTagGenData`** / **`RCG_EnemyTypeTagGenData`** — FilterDrop 用的标签型别。

### A.5 已知议题

*   类别名 `enum EDropType`（前缀 E）是为了避开 GoogleSheet 同步冲突的历史包袱。
*   递回上限 10 层。
