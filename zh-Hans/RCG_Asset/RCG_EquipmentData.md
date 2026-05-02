---
title: 装备资料 (RCG_EquipmentData) 说明
description: 角色穿戴的装备模板（武器、防具、饰品、遗物）：类型、稀有度、效果、强化分支、不重复掉落
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 装备资料

> 程式类别名称：`RCG_EquipmentData`

## 用途

**角色身上穿戴的装备模板**。武器、防具、饰品、遗物（特殊加成物品）皆是。每个装备有自己的稀有度、类型、效果（战斗中各 trigger 触发）、强化分支、是否能重复掉落。

继承自 `RCG_Asset<RCG_EquipmentData>`，实作介面：`RCGI_Item` / `RCGI_Unloackable`。

## 编辑器中的样貌

```
RCG_EquipmentData: <ID>
    Name / Description / Rarity / Tags / EquipmentType / Icon / Price
    Effects                    ← 战斗中触发的效果
    InitCounters               ← 起始计数器
    SkillTags                  ← 职业专属限制
    Unlock                     ← 解锁条件
    UpgradeBranch              ← 可强化成哪些装备
    EnhenceLevel / HideInCodex / CanDropRepeatedly
    Preview
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Name** | 是 | 装备名（多语系） |
| **Description** | 否 | 风味描述 |
| **Rarity** | 是 | 稀有度 |
| **Tags** | 否 | 一般标签（DropPool 用） |
| **EquipmentType** | 是 | `Weapon` / `Armor` / `Accessory` / `Relic`（**遗物：不占装备格、强制装在主角身上**） |
| **Icon** | 是 | 装备图 |
| **Price** | 是 | 商店售价；预设 500，可用 `Auto Price` 按钮重算（基础 10 × 稀有度价值） |
| **Effects** | 否 | 战斗中各 trigger 触发的效果 |
| **InitCounters** | 否 | 给 effect 中计数器类效果用的初值 |
| **SkillTags** | 否 | 职业专属限制（队伍中要有对应职业才掉落；OR 关系：任一符合即可） |
| **Unlock** | 否 | 解锁条件 |
| **UpgradeBranch** | 否 | 可强化成哪些装备（透过游戏内强化机制）；空 = 不能强化 |
| **EnhenceLevel** | — | 强化等级 > 0 表示「强化版」，**不显示在图鉴** |
| **HideInCodex** | — | 图鉴隐藏（测试 / 强化版会自动隐藏） |
| **CanDropRepeatedly** | — | 是否能重复掉落；遗物 (`Relic`) 通常设 false |

## 行为说明

### 触发
`OnTriggerEffect(data, triggerOn)` → 从 `m_Effects` 取对应 trigger 的 effects 并依序触发。`TriggerOnUnitState(triggerOn)` 是 quick check。

### 图鉴隐藏判断
`HideInCodex` (property) = `m_HideInCodex || m_EnhenceLevel > 0`：强化版自动隐藏，避免图鉴出现「+1 / +2」等同名重复。

### 描述
`GetFullDescription` = `Effects 描述 + m_Description（风味文字）`。

### Auto Price
编辑器页面有 `Auto Price` 按钮：对所有装备套用 `Price = 10 × Rarity.m_Value`。

### 排序
`RCG_EquimpmentComparer` 按「装备类型（Weapon=10, Armor=20, Accessory=30）」→「价格降序」排序。

## 注意事项

*   **遗物 (`Relic`) 规则**：不占装备格、强制在主角身上、`CanDropRepeatedly` 通常设 false（DropPool 会检查身上是否已有）。
*   **`SkillTags` 与卡牌专精的差异**：装备是 OR 关系（任一队员有就能掉），卡牌的 `RequireSkills` 才是 AND。
*   **强化版 (`EnhenceLevel > 0`) 自动隐藏图鉴**：图鉴只会看到原版。
*   **`NullEquipmentID = "NullEquipment"`** 是系统保留 ID，代表「无装备」；不要拿来命名一般装备。
*   **Auto Price 会洗手动价**。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_EquipmentData.cs`
*   **继承自**：`RCG_Asset<RCG_EquipmentData>`
*   **实作介面**：`RCGI_Item` / `RCGI_Unloackable`
*   **AssetGroup**：`EditItems`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Description` | Description | `RCG_LocalizeData` | |
| `m_Rarity` | Rarity | `RCG_RarityTagGenData` | |
| `m_Tags` | Tags | `List<RCG_ItemTagGenData>` | |
| `m_EquipmentType` | EquipmentType | `EquipmentType` enum | `Weapon` 预设 |
| `m_Icon` | Icon | `RCG_SpriteData` | |
| `m_Price` | Price | `int` | 预设 500 |
| `m_Effects` | Effects | `List<RCG_CommonEffect>` | |
| `m_InitCounters` | InitCounters | `List<int>` | |
| `m_SkillTags` | SkillTags | `List<RCG_SkillTagGenData>` | |
| `m_Unlock` | Unlock | `RCG_UnlockEntry` | |
| `m_UpgradeBranch` | UpgradeBranch | `List<RCG_EquipmentGenData>` | |
| `m_EnhenceLevel` | EnhenceLevel | `int` | 预设 0 |
| `m_HideInCodex` | HideInCodex | `bool` | |
| `m_CanDropRepeatedly` | CanDropRepeatedly | `bool` | 预设 `true` |

### A.3 重要 Method 摘要

*   **`AddItem()`** — 玩家获得：`m_EquipmentsData.AddEquipment(new RCG_Equipment(ID))`。
*   **`OnTriggerEffect(data, triggerOn)`** — 战斗触发。
*   **`HideInCodex` (property)** — `m_HideInCodex || m_EnhenceLevel > 0`。
*   **`GetDescription` / `GetFullDescription`** — 描述生成。
*   **`CreateSelectAssetPage`** — `RCG_EquipmentDataEditorPage.Create()`。

### A.4 与其他系统的互动

*   **`RCG_Equipment`** — runtime 装备实例。
*   **`RCG_DataService.Ins.m_EquipmentsData`** — 玩家装备储存。
*   **`RCG_EquipmentDropPool`** — 掉落池（含 `m_CanDropRepeatedly` 检查）。
*   **`RCG_EquipmentInfoPanel`** — 详细资讯 UI。
*   **`RCG_EquimpmentComparer`** — 排序 helper。

### A.5 已知议题

*   `m_CanDropRepeatedly` 的 TODO 注解（"需要记录掉落 & 排除"）暗示「不重复掉落」的纪录机制有历史改动；目前由 DropPool runtime 比对玩家装备清单实现。
*   `DeserializeFromJson` 的自动价格 `m_Price = m_Rarity.GetData().m_Value.GetValue(null) * 15` 已注解，被 Auto Price 工具取代。
