---
title: 故事掉落池 (RCG_StoryDropPool) 说明
description: 定义「在某情境下会抽到哪些故事 (RCG_StoryData)」的资料；剧情段落、随机故事插入点来源
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 故事掉落池

> 程式类别名称：`RCG_StoryDropPool`

## 用途

定义「**这个情境下可能抽到哪些故事段落**」。与 `RCG_QuestDropPool` 不同，这里掉的是 `RCG_StoryData`（纯剧情、对话、过场），不含战斗或选择。例如「进入新章节时播放的开场白」「夜晚休息时的旅途见闻」。

继承自 `RCG_Asset<RCG_StoryDropPool>`，实作介面：`UCL.Core.UCLI_ShortName`。

## 编辑器中的样貌

```
RCG_StoryDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ DropPool / MixDropPools / FilterDropData
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **DropType** | 是 | `DropPool` / `MixPool` / `FilterDrop` |
| **DropPool** | DropType=DropPool | 故事清单 + 权重 |
| **MixDropPools** | DropType=MixPool | 引用其他池子并指定权重 |
| **FilterDropData** | DropType=FilterDrop | 用内部 `DropFilter` 筛 |

## 行为说明

与其他 Drop Pool 同骨架；无 unlock / skill 等 runtime 筛选。

## 注意事项

*   结构与 `RCG_QuestDropPool` 几乎相同，差异只在掉落目标型别与筛选 tag 种类。
*   **DropPool 模式下**反序列化会自动清理失效 ID（依 `iData.Exist()`）。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_StoryDropPool.cs`
*   **继承自**：`RCG_Asset<RCG_StoryDropPool>`
*   **AssetGroup**：`EditDropSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_DropPool` | 掉落池 | `RCG_CommonDropSetting<RCG_StoryGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_FilterDropData` | 条件式 | `FilterDropData` | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `EDropType` enum | 预设 `DropPool` |

### A.3 重要 Method

*   **`GetDrops(int)`** / **`GetDropsWithFilterFunc(int, Func)`** — 抽 N 个故事。
*   **`GetDropRate(...)`** — 标准化权重表。

### A.4 与其他系统的互动

*   **`RCG_StoryData`** — 掉落目标。
*   **`RCG_StoryGenData`** / **`RCG_StoryDropPoolGenData`** — Asset Entry 包装。
