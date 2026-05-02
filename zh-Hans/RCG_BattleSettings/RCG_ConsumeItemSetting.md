---
title: 消耗道具 说明
description: 从玩家背包消耗指定物品；可作为「消耗道具才能打出」的卡片条件
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 消耗道具

> 程式类别名称：`RCG_ConsumeItemSetting`

## 用途
**消耗玩家背包中的物品**。可作为「**打出条件**」— 没有对应物品时这张卡无法打出。常见用途：
*   「消耗 1 个火药包，造成 AOE 火焰伤害」
*   「消耗 2 个药草，全队回血」

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **道具** (`Items`) | 是 | 要消耗的物品清单；每项是 `RCG_ItemGenData`（物品模板）。 |

## 行为说明
*   **可玩判定**：背包必须含有清单上的全部物品才能打出（按 ID 配对，**多个同 ID 也要对应消耗**）。
*   **触发**：依清单逐一消耗对应背包物品。
*   描述：列出所有要消耗的物品名称。

## 注意事项
*   **TODO 提醒**：目前**资源变化时不会即时刷新卡牌可玩判定** — 玩家可能看到「不可打出」状态延迟更新。详情查 source 中 `Todo:资源变化时 要刷新卡牌才能正确判断目前是否能打出`。
*   **空清单**：合法，但等于无消耗 — 这张卡会永远可打出，且不会消耗任何东西。
*   **重复消耗同 ID 物品**：清单中同 ID 出现两次代表要消耗 2 个；背包不足 2 个则不可打出。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ConsumeItemSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_ConsumeItemSetting` → 「消耗道具」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_Items` | 道具 | `List<RCG_ItemGenData>` | `Items` (=「道具」) | 注解写成「获得的资源(金币等..)」是旧注解，现为消耗 |

### A.3 重要 Method 摘要
*   **`CheckPlayable`**：clone `RCG_DataService.Ins.m_ItemsData.GetItems()`，逐一以 `aItem.ID` 配对找到并 RemoveAt；找不到立即 false。
*   **`Infos`**：base.Infos + 每个 `m_Items[i].GetData()` 的资讯。
*   **`AddAction`**（档案 80 行外）：实际从背包扣除物品。

### A.4 与其他系统的互动
*   **`RCG_DataService.Ins.m_ItemsData`**：玩家背包资料来源。
*   **`RCG_ItemGenData / RCG_Item`**：物品模板与实例。

### A.5 已知议题
*   程式中明文 `Todo:资源变化时 要刷新卡牌才能正确判断目前是否能打出` — 卡片可玩状态未即时刷新。
*   栏位注解写「获得的资源」可能是 copy-paste 残留，实际为「消耗的物品」。
