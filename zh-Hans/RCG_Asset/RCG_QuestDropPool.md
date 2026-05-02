---
title: 任务/事件掉落池 (RCG_QuestDropPool) 说明
description: 定义「在某情境下会触发哪些事件 (RCG_QuestData)」的资料；地图节点、随机事件来源
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 任务/事件掉落池

> 程式类别名称：`RCG_QuestDropPool`

## 用途

定义「**这个情境下可能触发哪些事件 (Quest)**」。例如「中型城镇」「森林神秘节点」「夜晚遭遇」分别是不同池，池内列出可能的 `RCG_QuestData`（每个 Quest 是一段事件脚本：对话、选择、结果）+ 权重。

继承自 `RCG_Asset<RCG_QuestDropPool>`，实作介面：`UCL.Core.UCLI_ShortName`。

## 编辑器中的样貌

```
RCG_QuestDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ DropPool / MixDropPools / FilterDropData
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **DropType** | 是 | `DropPool` / `MixPool` / `FilterDrop` |
| **DropPool** | DropType=DropPool | 事件清单 + 权重 |
| **MixDropPools** | DropType=MixPool | 引用其他池子并指定权重 |
| **FilterDropData** | DropType=FilterDrop | 用内部 `DropFilter`（FilterType: Tag / Operator）筛 |

## 行为说明

### 三种模式
与其他 Drop Pool 同。FilterDrop 只支援「事件标签」单一维度。

### 隐性筛选
本类别**没有 unlock 筛选**——预设 filter 永远 return true（程式内有注解掉的「不连续触发相同事件」逻辑，目前未启用）。

### 预览
按 `ShowDetail` 即时看当前掉落率。

## 注意事项

*   **`enum DropType` 不是 `EDropType`**：与 BattleSetDropPool / ItemDropPool 等命名不一致；这是历史遗留差异。
*   **无 Name 栏位**：`GetShortName()` 直接取第一个 drop 的名称。
*   **没有自动清理失效 ID 的 `DeserializeFromJson` override**——坏 ID 不会自动消失，需要手动修。

---

## 附录：程式人员参考 (Programmer Reference)

> 此段以下使用程式内部术语，受众转为程式人员与 AI agent。前半段内容请优先采信。

### A.1 类别资讯

*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_QuestDropPool.cs`
*   **继承自**：`RCG_Asset<RCG_QuestDropPool>`
*   **实作介面**：`UCL.Core.UCLI_ShortName`
*   **AssetGroup**：`EditDropSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_DropPool` | 掉落池 | `RCG_CommonDropSetting<RCG_QuestGenData>` | — | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | — | `Conditional(MixPool)` |
| `m_FilterDropData` | 条件式 | `FilterDropData` = `FilterDropDataBase<RCG_QuestGenData, RCG_QuestData, DropFilter>` | — | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `DropType` enum | `DropType` | 预设 `DropPool` |

### A.3 重要 Method 摘要

*   **`GetDrops(int, CheckDropConditionData)`** → 主要入口。
*   **`GetDropsWithFilterFunc(int, Func)`** → 自订筛选版。
*   **`GetDropRate(CheckDropConditionData, int)`** → 预设 filter 永远 true（程式注解掉「不连续触发相同事件」逻辑）。

### A.4 与其他系统的互动

*   **`RCG_QuestData`** — 池子掉的目标型别。
*   **`RCG_QuestGenData`** / **`RCG_QuestDropPoolGenData`** — Asset Entry 包装；后者预设 ID = `"NormalDrop"`。
*   **`RCG_EventTagGenData`** — FilterDrop 条件用的事件标签型别。

### A.5 已知议题

*   程式内有注解掉的「不连续触发相同事件」(`triggeredStory.LastElement()`) 逻辑——若需此行为要解注解。
*   无 `DeserializeFromJson` override，坏 ID 不会自动清理。
