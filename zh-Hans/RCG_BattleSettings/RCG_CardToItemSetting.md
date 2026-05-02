---
title: 卡牌转为物品 说明
description: 把选取的手牌道具化（产生「打出此卡」效果的物品），可选临时或永久
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 卡牌转为物品

> 程式类别名称：`RCG_CardToItemSetting`

## 用途
**把卡牌转化为背包物品**。物品的使用效果就是「**生成这张卡到手牌并打出**」。常见用途：
*   「将攻击卡储存为药水」
*   「卡片化道具」风格的特殊机制

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **SelectHandCardSetting** | 是 | 选取要道具化的手牌；展开设定条件、张数、过滤。 |
| **IsTmpItem** | — | 预设打勾。打勾 = 战斗结束自动移除；不打勾 = 永久保留。 |

## 行为说明
*   先让玩家透过 `SelectHandCardSetting` 选牌。
*   对每张选中的卡：
    1. clone 出新 `RCG_ItemData`（名称、图示沿用卡片）。
    2. 物品的「使用效果」自动写入 `RCG_CardCreateSetting`（生成此卡并加入手牌）。
    3. 咏唱卡会把 `IsChanted` 带入避免重复咏唱。
    4. 注册为 `BattleRuntime` 资料并加入背包。
*   原牌会被消灭。
*   描述格式：`IsTmpItem=true` → `CardToTmpItem_Des`；否则 → `CardToItem_Des`。

## 注意事项
*   **临时物品 vs 永久物品的存档**：临时物品在战斗结束时清除；永久物品则保存到玩家背包，跨战斗保留 — 大量永久道具化可能让背包爆炸，请设计上限。
*   **道具化后不能再融合**：原卡已被消灭；想做「卡牌融合 + 道具化」请先融合再道具化。
*   **与「卡牌融合」的差别**：融合是合多张变新卡；本设定是「**卡 → 物品**」单向转换。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardToItemSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_CardToItemSetting` → 「卡牌转为物品」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_SelectHandCardSetting` | `SelectHandCardSetting` | `RCG_SelectHandCardSetting` | — | |
| `m_IsTmpItem` | `IsTmpItem` | `bool` | `IsTmpItem` (=「是否为临时物品?」) | 预设 `true` |

### A.3 重要 Method 摘要
*   **`AddAction`**：选牌后对每张卡：
    1. 建 `RCG_ItemData`（`Data.m_Name = m_LocalizeName`, `Data.m_Icon = m_CardIcon`, `ID = aCard.ID`）。
    2. 嵌套 `RCG_CommonEffect` (`OnPlay`) → `RCG_CardCreateSetting` (`AddToHandCard`, `m_RuntimeCardData = new RCG_CardBattleDataPointer(aCard)`)。
    3. 咏唱卡 → `aSetting.m_IsChanted = true`。
    4. `RCG_DataService.Ins.AddRuntimeData(aItemData, DataType.BattleRuntime)`。
    5. `new RCG_Item(aItemData, RCG_Item.ItemType.Runtime)` + `m_TmpItem = m_IsTmpItem` + `AddItem()`。
*   **`Info`**：`m_IsTmpItem ? CardInfoData.TmpItemInfo : null`。

### A.4 与其他系统的互动
*   **`RCG_CardCreateSetting` + `RCG_CardBattleDataPointer`**：物品的使用效果以这个组合表达「打出记下的卡」。
*   **`DataType.BattleRuntime`**：战斗结束自动清除的资料容器。
*   **`RCG_AquireItemPanel`**：UI 反馈（档案 80 行外的后续逻辑）。
