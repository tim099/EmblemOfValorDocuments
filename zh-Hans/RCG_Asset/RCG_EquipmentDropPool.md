---
title: 装备掉落池 (RCG_EquipmentDropPool) 说明
description: 定义一组「会掉哪些装备、权重」的资料；商店、战斗奖励、宝箱靠它抽装备
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 装备掉落池

> 程式类别名称：`RCG_EquipmentDropPool`

## 用途

定义「**这个池子会掉哪些装备、各自权重**」。战斗胜利后、商店、宝箱、Boss 战利品从这里抽装备（武器、护甲、饰品、遗物⋯）。与卡牌/道具池骨架相同，差异是「**遗物类装备不能重复掉落**」由此处 runtime 筛除。

继承自 `RCG_Asset<RCG_EquipmentDropPool>`，实作介面：`UCL.Core.UCLI_ShortName`。

## 编辑器中的样貌

```
RCG_EquipmentDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    Name(多国语言)
    ▼ DropPool / MixDropPools / FilterDropData  ← 视 DropType 显示
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **DropType** | 是 | `DropPool` / `MixPool` / `FilterDrop` |
| **Name** | 否 | 显示名称（多语系） |
| **DropPool** | DropType=DropPool | 装备清单 + 权重 |
| **MixDropPools** | DropType=MixPool | 引用其他池子并指定权重 |
| **FilterDropData** | DropType=FilterDrop | 用内部 `DropFilter`（SkillTag / RarityTag / Tag / EquipmentType / Operator）筛 |

## 行为说明

### 三种模式
与其他 Drop Pool 同：DropPool（直接列）/ MixPool（合并）/ FilterDrop（条件筛）。FilterDrop 比卡牌池多一层「**装备类型 (EquipmentType)**」可筛武器/防具/饰品。

### 隐性筛选（runtime）
*   **未解锁** (`UnlockData.m_LockedEquipments.CheckLocked`)：跳过。
*   **遗物已拥有** (`m_CanDropRepeatedly = false` 且玩家身上已有同 ID 装备)：跳过。**这就是遗物只会出现一次的机制**。
*   **队伍无人符合专精**（装备有指定 `m_SkillTags` 但队伍无人持有）：跳过。
被剔除后剩余装备**重新标准化权重**。

### 预览
编辑器按 `ShowDetail` 即时看最终掉落率。

## 注意事项

*   **遗物的不重复掉落**完全靠 runtime 比对 `RCG_DataService.Ins.m_EquipmentsData.m_Equipments`；单机预览看不到此逻辑，要实机测试才会生效。
*   **装备有专精限制**时池子可能在某些队伍组合下大幅缩水甚至空池。设计时要考虑所有可能队伍。
*   **DropPool 模式下** 反序列化时会自动移除不存在的装备 ID。
*   **`CheckRequireSkill` 在这个类别永远回 true**，过滤逻辑都集中在 `GetDropRate` 内的 closure 里。

---

## 附录：程式人员参考 (Programmer Reference)

> 此段以下使用程式内部术语，受众转为程式人员与 AI agent。前半段内容请优先采信。

### A.1 类别资讯

*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_EquipmentDropPool.cs`
*   **继承自**：`RCG_Asset<RCG_EquipmentDropPool>`
*   **实作介面**：`UCL.Core.UCLI_ShortName`
*   **AssetGroup**：`EditDropSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_Name` | 显示名称 | `RCG_LocalizeData` | `Name` | `[SerializeField] protected` |
| `m_DropPool` | 掉落池 | `RCG_CommonDropSetting<RCG_EquipmentGenData>` | — | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | — | `Conditional(MixPool)` |
| `m_FilterDropData` | 条件式 | `FilterDropData` = `FilterDropDataBase<RCG_EquipmentGenData, RCG_EquipmentData, DropFilter>` | — | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `EDropType` enum | `DropType` | 预设 `DropPool` |

### A.3 重要 Method 摘要

*   **`GetDropEquipments(int, List<RCG_SkillTagGenData>)`** → 主要入口；不指定 skills 时自动取队伍当前 skills。
*   **`GetDropRate(List<RCG_SkillTagGenData>, ...)`** → 套用 unlock + 遗物 + 专精检查。
*   **`CheckRequireSkill`** (static) → 永远回 `true`（保留签名一致性，实际过滤在 `GetDropRate` 的 closure）。
*   **`DeserializeFromJson`** → 清理失效 ID。

### A.4 与其他系统的互动

*   **`RCG_EquipmentData`** — 池子掉的目标型别。
*   **`RCG_EquipmentGenData`** / **`RCG_EquipmentDropPoolGenData`** — Asset Entry 包装。
*   **`RCG_DataService.Ins.m_EquipmentsData.m_Equipments`** — 玩家身上装备清单，用于遗物去重。
*   **`RCG_DataService.Ins.m_UnlockData.m_LockedEquipments`** — unlock 筛选。
*   **`RCG_CharacterDataService.Ins.GetAllSkillTags()`** — 队伍专精查询。

### A.5 已知议题

*   `CheckRequireSkill` 注解 `(应该不需要)` — 逻辑已搬到 `GetDropRate` 内 closure。
*   递回上限 10 层。
