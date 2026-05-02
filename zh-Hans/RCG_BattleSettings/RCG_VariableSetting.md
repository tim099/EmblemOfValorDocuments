---
title: 宣告变数 说明
description: 从多种战斗条件取出数值并存入命名变数，后续设定可引用
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 宣告变数

> 程式类别名称：`RCG_VariableSetting`

## 用途
**从各种战斗条件取出一个数值，存入指定变数名称**，让后续设定可引用此变数。是「**动态效果**」的核心 — 实现如「**对每张使用过的攻击卡造成 1 点伤害**」这类依战斗状态缩放的卡牌。

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **VariableName** | 是 | 变数名称（预设 `X`），后续设定以此名称引用。 |
| **VariableCondition** | 是 | 取值条件（**18 种**，见下表）。 |
| **Range** | TargetStatusCount/TargetAttackPower/TargetArmorCount/TargetDebuffLayersSum 模式 | 目标范围（attack range 风格）。 |
| **UsedCardType** | CardUsedCount/TurnCardUsedCount 模式 | 计算的卡牌类型（空清单 = 不限类型）。 |
| **Status** | TargetStatusCount 模式 | 要查询层数的状态类型。 |
| **MonsterTags** | MonsterCount 模式 | 限制怪物类型；空清单 = 全部。 |
| **Value** | IntVariable 模式 | 直接给定的固定数值（变数可变）。 |

### VariableCondition 18 种模式
| 值 | 取出什么 |
|---|---|
| **RemainCost** | 玩家当前能量（预设） |
| **HandCardCount** | 手牌张数 |
| **DiscardedCardCount** | 弃牌堆张数 |
| **DeckCardCount** | 抽牌堆张数 |
| **BanishedCardCount** | 消灭牌堆张数 |
| **CardUsedCount** | 本场战斗使用过的卡（可限类型） |
| **TurnCardUsedCount** | 本回合使用过的卡（可限类型） |
| **ThisCardUseCount** | 这张卡的累计使用次数 |
| **ThisCardCost** | 这张卡的当前费用 |
| **CostOfSelectedCards** | 选取卡的费用总和 |
| **TargetStatusCount** | 目标的指定状态层数 |
| **TargetDebuffLayersSum** | 目标所有 Debuff 层数总和 |
| **TargetArmorCount** | 目标的护甲层数 |
| **TargetAttackPower** | 目标当前攻击力（怪物 AI 行动中最大值） |
| **MonsterCount** | 场上活着的敌人数量（可限类型） |
| **PlayerUnitCount** | 场上活着的我方单位数量 |
| **DarkMistLevel** | 黯雾等级（资源 `Supply`） |
| **IntVariable** | 直接给定的固定数值或变数计算结果 |

## 行为说明
*   执行时：依 VariableCondition 计算数值 → 存入 `iData.VariableDic[VariableName]`。
*   描述格式：「**{变数条件描述} {变数名称}({实际值}) = {取出的值}**」 — 在战斗中会即时显示当前值。
*   Editor 模式（非战斗）下，变数值显示为占位（不真的执行）。

## 注意事项
*   **变数名称冲突**：多个 `VariableSetting` 用同名变数，后者会覆盖前者。请语意化命名（如 `HandCount` / `EnemyAtk`）。
*   **变数作用域**：存于 `iData.VariableDic` — 同一个 `TriggerEffectData` 树内共享。**子 ChildData 也能看到**（继承）。
*   **VariableName 为空**：`AddAction` 直接 return — 等于这个设定**完全没效果**。
*   **计算时机**：在 `AddAction` 触发点计算 — 后续设定取值时拿的是该瞬间的快照。
*   **预览伤害**：`GetPreviewDamage` 会把变数值预先写入 `VariableDic` 让下游预览计算使用。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_VariableSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_VariableSetting` → 「宣告变数」
*   **同档顶层 enum**：`VariableCondition` (18 个值)

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_VariableName` | `VariableName` | `string` | `VariableName` (=「变数名称」) | 预设 `"X"` |
| `m_VariableCondition` | `VariableCondition` | `VariableCondition` (档内 enum) | — | 预设 `RemainCost` |
| `m_Range` | `Range` | `AttackRange` | — | `[Conditional(... 4 种需要 Range 的条件)]` |
| `m_UsedCardType` | `UsedCardType` | `List<RCG_CardTagGenData>` | — | `[Conditional(... CardUsedCount, TurnCardUsedCount)]` |
| `m_Status` | `Status` | `RCG_CustomStatusGenData` | `Status` | `[Conditional(... TargetStatusCount)]` |
| `m_MonsterTags` | `MonsterTags` | `List<RCG_MonsterTagGenData>` | — | `[Conditional(... MonsterCount)]` |
| `m_Value` | `Value` | `IntVariable` | — | `[Conditional(... IntVariable)]` |

### A.3 重要 Method 摘要
*   **`GetVariableValue(iData)` (protected)**：18 种 case 分流计算实际数值；非战斗模式或 `RCG_Player.Ins == null` 直接回 0。
*   **`AddAction`**：`m_VariableName` 为空直接 return；否则 `iData.VariableDic.Set(m_VariableName, GetVariableValue(iData))`。
*   **`GetPreviewDamage`** → 把当前值写入 VariableDic 并回传 `-1`（**非攻击牌但仍要写入变数让下游预览**）。
*   **`VarName(iData)` (public)** → 战斗中显示 `{name}({value})`，否则只显示 `{name}`。
*   **`GetDescriptionParams / GetDescriptionFormat`**：依 18 种条件生成不同句型，套对应 i18n key（`VariableCondition_*Des` / `VariableEffectDes` / `VariableEffectDes_IntVariable`）。
*   **`Info`**：`TargetStatusCount` 模式回传 `m_Status` 资讯；其他回 null。

### A.4 与其他系统的互动
*   **`iData.VariableDic`**：变数写入目标；后续 `IntVariable` 解析时引用。
*   **`RCG_Player.Ins.Cost / GetHandCardCount / DiscardedCardCount / BanishedCardCount / Deck.Deck.Count`**：抽出资料来源。
*   **`RCG_BattleAnalytics.Ins.GetCardUsedCount / GetTurnCardUsedCount`**：卡片使用统计。
*   **`RCG_BattleScene.Ins.GetUnits(UnitPos.All, faction)`**：场上单位查询。
*   **`RCG_DataService.Ins.GetResource(Supply)`**：黯雾等级资料来源。

### A.5 已知议题
*   旧版 `DeserializeFromJson` 处理（如 `IntVariable → RemainCost` 替换）已注解。
*   `// QWQ2` / `// TODO` 注解标记了部分待修区块（`HandCardCount` 描述参数）。
