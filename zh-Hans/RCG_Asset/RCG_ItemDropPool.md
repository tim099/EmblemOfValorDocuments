---
title: 道具掉落池 (RCG_ItemDropPool) 说明
description: 定义一组「会掉哪些道具、各自权重」的资料；事件奖励、宝箱、商店补货用它抽道具
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 道具掉落池

> 程式类别名称：`RCG_ItemDropPool`

## 用途

定义「**这个池子会掉哪些消耗道具、各自的权重**」。宝箱、事件奖励、商人补货从这里抽道具（药水、卷轴、食材⋯）。与 `RCG_CardDropPool` 相同骨架，只是抽的对象换成 `RCG_ItemData`。

继承自 `RCG_Asset<RCG_ItemDropPool>`，实作介面：`UCL.Core.UCLI_ShortName`。

## 编辑器中的样貌

```
RCG_ItemDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    Name(多国语言)
    ▼ DropPool / MixDropPools / FilterDropData  ← 视 DropType 显示
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **DropType** | 是 | `DropPool`（直接列）/ `MixPool`（混合别的池）/ `FilterDrop`（标签条件筛） |
| **Name** | 否 | 显示名称（多语系）。空白时 fallback 到第一个 drop 名称再 fallback 到 ID |
| **DropPool** | DropType=DropPool | 道具清单 + 权重（`RCG_CommonDropSetting<RCG_ItemGenData>`） |
| **MixDropPools** | DropType=MixPool | 引用其他池子并指定权重 |
| **FilterDropData** | DropType=FilterDrop | 用内部 `DropFilter`（FilterType: Tag / RarityTag / Operator）动态筛 |

## 行为说明

### 三种模式
*   **DropPool**：手动列出每个道具 + 权重。
*   **MixPool**：把多个既有池子按权重合并。最多递回 10 层。
*   **FilterDrop**：条件式（道具标签、稀有度），支援 AND / OR / NOT。没条件 = 全道具库均等。

### 隐性筛选（runtime）
*   **未解锁**（`UnlockData.m_LockedItems.CheckLocked`）：跳过。
被剔除后剩余道具**重新标准化权重**。

### 预览
编辑器内按 `ShowDetail` 可即时看当前掉落率列表。

## 注意事项

*   **DropPool 模式下** 反序列化时会自动移除不存在的道具 ID（检查 `iData.Exist()`）。
*   **FilterDrop 内部 `DropFilter`** 与 `RCG_CardDropPool` 使用的 `CardDropFilter` 不是同一个型别 — 道具不需要 SkillTag、EquipmentType 等卡牌专属筛选。
*   **MixPool 循环**会被 `iLayer > 10` 截断。
*   **道具未解锁**会直接从 runtime 池中剔除，编辑器预览不一定会反映此 runtime 逻辑。

---

## 附录：程式人员参考 (Programmer Reference)

> 此段以下使用程式内部术语，受众转为程式人员与 AI agent。前半段内容请优先采信。

### A.1 类别资讯

*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_ItemDropPool.cs`
*   **继承自**：`RCG_Asset<RCG_ItemDropPool>`
*   **实作介面**：`UCL.Core.UCLI_ShortName`
*   **AssetGroup**：`EditDropSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_Name` | 显示名称 | `RCG_LocalizeData` | `Name` | |
| `m_DropPool` | 掉落池 | `RCG_CommonDropSetting<RCG_ItemGenData>` | — | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | — | `Conditional(MixPool)` |
| `m_FilterDropData` | 条件式 | `FilterDropData` = `FilterDropDataBase<RCG_ItemGenData, RCG_ItemData, DropFilter>` | — | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `EDropType` enum | `DropType` | |

### A.3 重要 Method 摘要

*   **`GetDropItems(int)`** → 主要入口；自动套用 unlock 筛选后抽 N 个。
*   **`GetDropItemsWithFilterFunc(int, Func)`** → 自订筛选版。
*   **`GetDropRate(CheckDropConditionData, int)`** → 标准化后权重表。
*   **`DeserializeFromJson`** → 清理失效 ID。

### A.4 与其他系统的互动

*   **`RCG_ItemData`** — 池子掉的目标型别。
*   **`RCG_ItemGenData`** — Asset Entry 包装。
*   **`RCG_ItemDropPoolGenData`** — 其他 Asset 引用此池子时的型别。
*   **`RCG_DataService.Ins.m_UnlockData.m_LockedItems`** — unlock 筛选来源。
*   **`FilterDropDataBase<TGenData, TData, TFilter>`** — 通用条件筛 base class。

### A.5 已知议题

*   程式注解 `// 清理不存在的道具 QWQ` — 标示 deserialize 阶段的清理流程。
*   递回上限 10 层（`iLayer > 10`）。
