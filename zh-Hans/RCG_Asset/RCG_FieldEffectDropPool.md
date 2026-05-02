---
title: 场地效果掉落池 (RCG_FieldEffectDropPool) 说明
description: 定义「会抽到哪些场地效果」的资料；地图节点 / 战斗开始时的随机场地修正来源
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 场地效果掉落池

> 程式类别名称：`RCG_FieldEffectDropPool`

## 用途

定义「**这个情境下可能抽到哪些场地效果**」。场地效果是**作用于整个战场**的修正（例：本场战斗所有单位每回合 -1 HP / 火焰场地 / 黑暗垄罩）；本池决定随机场地时的候选与权重。

继承自 `RCG_Asset<RCG_FieldEffectDropPool>`，实作介面：`UCL.Core.UCLI_ShortName`。

## 编辑器中的样貌

```
RCG_FieldEffectDropPool: <ID>
    DropType  ▾ DropPool / MixPool      ← 没有 FilterDrop
    ▼ DropPool / MixDropPools
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **DropType** | 是 | `DropPool` / `MixPool`（**只有两种模式**） |
| **DropPool** | DropType=DropPool | 场地效果清单 + 权重 |
| **MixDropPools** | DropType=MixPool | 混合其他池子 |

## 行为说明

与 `RCG_StatusDropPool` 结构相同——只有 DropPool / MixPool 两种模式。

## 注意事项

*   **没有 FilterDrop 模式**。
*   **enum 名称叫 `DropType`**（不是 `EDropType`）。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_FieldEffectDropPool.cs`
*   **继承自**：`RCG_Asset<RCG_FieldEffectDropPool>`
*   **AssetGroup**：`EditDropSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_DropPool` | 掉落池 | `RCG_CommonDropSetting<RCG_FieldEffectGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_DropType` | DropType | `DropType` enum | 只有 `DropPool` / `MixPool` |

### A.3 与其他系统的互动

*   **`RCG_FieldEffectData`** — 掉落目标。
*   **`RCG_FieldEffectGenData`** / **`RCG_FieldEffectDropPoolGenData`** — Asset Entry 包装。
