---
title: 卡牌融合 说明
description: 让玩家选取多张手牌进行融合，产生新卡牌；原牌会被消灭
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 卡牌融合

> 程式类别名称：`RCG_CardFusionSetting`

## 用途
**让玩家选取手牌进行融合**：选中的卡会被消灭，并产生一张新的融合卡加入手牌。常见用途：
*   「炼金术士」风格的合成卡
*   「将两张攻击卡合成为强化攻击」

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **SelectHandCardSetting** | 是 | 内嵌的选取设定，决定选择 UI 的条件、张数、限制等。 |

## 行为说明
*   先触发内嵌的 `SelectHandCardSetting` 让玩家选牌。
*   选定后：
    1. 对选中卡呼叫 `CardFusion()` 计算融合后的 `RCG_CardData`。
    2. 用融合结果建立新 `RCG_CardBattleData`。
    3. 把选中的卡全部 **`Banish` 消灭**（不是弃牌！）。
    4. 把新卡加入手牌。
*   描述格式为「**将选取的卡融合**」（i18n key `CardFusion_Des`）。

## 注意事项
*   **融合逻辑由 `CardFusion()` 决定**：实际融合规则是**卡牌系统层级**的（而非本设定）— 想知道两张卡融合会产生什么，请查看 `RCG_CardData` / `IList<RCG_CardData>.CardFusion()`。
*   **原牌一律消灭**：是 `RemoveType.Banish`，不会进弃牌堆，**不会触发弃牌相关效果**。
*   **没有融合候选时**：选择 UI 仍会跳出但无可选项；玩家只能取消。请在 `SelectHandCardSetting` 中设定合理的最低张数。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardFusionSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_CardFusionSetting` → 「卡牌融合」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_SelectHandCardSetting` | `SelectHandCardSetting` | `RCG_SelectHandCardSetting` | — | |

### A.3 重要 Method 摘要
*   **`AddAction`**：先呼叫 `m_SelectHandCardSetting.AddAction(InsertInOrder)`，再 ActionTrigger 内：
    1. `iData.SelectedHandCards` → 收集 `RCG_CardData` 并逐张加入 `aDiscardCardList`。
    2. `aSelectedCards.CardFusion()` → `RCG_CardData` 融合结果。
    3. `RCG_CardBattleData.CreateCard(aFusionCardData, false)` → 新 battle data。
    4. `CreateAction.AddDiscardCardActions(..., RemoveType.Banish)` 消灭原牌。
    5. `CreateAction.CreateCard(aNewCard, CreateCardType.AddToHandCard)` 加入手牌。
*   **`GetDescriptionFormat`**：i18n key `CardFusion_Des`，内含 `m_SelectHandCardSetting.GetDescriptionFormat`。

### A.4 与其他系统的互动
*   **`IList<RCG_CardData>.CardFusion()`**：实际融合逻辑（扩充方法）。
*   **`RCG_SelectHandCardSetting`**：选牌 UI 触发。
*   **`CreateAction.AddDiscardCardActions / CreateCard`**：消灭 + 生成的 Action 建构工具。
