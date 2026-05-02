---
title: 卡牌强化掉落池 (RCG_CardEnhenceDropPool) 说明
description: 定义「强化卡牌时可抽到哪些强化分支」的资料；强化选单背后的随机池
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 卡牌强化掉落池

> 程式类别名称：`RCG_CardEnhenceDropPool`

## 用途

定义「**强化某张卡时，会抽到哪些强化分支 (`RCG_CardEnhenceData`) 作为候选**」。每张卡的 `m_EnhencePool` 栏位会引用一个 `RCG_CardEnhenceDropPool`；强化时系统从此池抽 N 个分支让玩家挑。

继承自 `RCG_Asset<RCG_CardEnhenceDropPool>`，实作介面：`UCL.Core.UCLI_ShortName`。

## 编辑器中的样貌

```
RCG_CardEnhenceDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ DropPool / MixDropPools / FilterDropData
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **DropType** | 是 | `DropPool` / `MixPool` / `FilterDrop` |
| **DropPool** | DropType=DropPool | 强化分支清单 + 权重 |
| **MixDropPools** | DropType=MixPool | 混合其他池子 |
| **FilterDropData** | DropType=FilterDrop | 用内部 `DropFilter`（Operator）筛 |

## 行为说明

与其他 Drop Pool 同骨架。内部 `DropFilter` 目前**只支援 Operator 类型**（Tag 已被注解掉），实务上 FilterDrop 模式较少使用，多数采 DropPool 直接列。

抽取时还会被卡牌端 `BannedEnhence` 列表 + `RCG_CardEnhenceCondition` 二次过滤（见 `RCG_CardData.GetEnhenceBranchs`）。

## 注意事项

*   **enum 名称叫 `DropType`**（不是 `EDropType`）。
*   **DropFilter Tag 已被注解掉**：FilterDrop 模式现阶段功能受限；建议用 DropPool 模式直接列。
*   **强化条件与 BannedEnhence** 的二次筛选在卡牌端进行，本池只负责「初次抽出候选」。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_CardEnhenceDropPool.cs`
*   **继承自**：`RCG_Asset<RCG_CardEnhenceDropPool>`
*   **AssetGroup**：`EditDropSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_DropPool` | 掉落池 | `RCG_CommonDropSetting<RCG_CardEnhenceGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_FilterDropData` | 条件式 | `FilterDropData` | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `DropType` enum | 预设 `DropPool` |

### A.3 与其他系统的互动

*   **`RCG_CardEnhenceData`** — 掉落目标（强化分支定义）。
*   **`RCG_CardEnhenceGenData`** / **`RCG_CardEnhenceDropPoolGenData`** — Asset Entry 包装。
*   **`RCG_CardData.m_EnhencePool` / `RCG_CardData.GetEnhenceBranchs`** — 强化流程的呼叫端，会于此池抽出候选后再做二次过滤。
