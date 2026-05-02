---
title: 牌组资料 (RCG_DeckData) 说明
description: 一份完整牌组的定义（卡牌清单与配置）；角色起始牌组、加入牌组、玩家牌组底层
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 牌组资料

> 程式类别名称：`RCG_DeckData`

## 用途

**一份完整牌组的定义**。每张卡牌 + 数量 = 牌组。应用情境：
*   角色 `RCG_CharacterData.m_Deck`（起始牌组）
*   角色 `m_JoinDeck`（中途加入时带来的牌组）
*   `RCG_BattlePresetData.m_Deck`（测试战斗用牌组）
*   特殊事件给予的整套牌

继承自 `RCG_Asset<RCG_DeckData>`，实作介面：`RCGI_Unloackable`（可解锁）。

## 编辑器中的样貌

```
RCG_DeckData: <ID>
    Name           ← 牌组显示名（多语系）
    Deck           ← 卡牌清单（SpawnDeckData，含每张卡 + 数量）
    Unlock         ← 解锁条件
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Name** | 否 | 显示名（多语系）；空白时 fallback 到 ID |
| **Deck** | 是 | 卡牌清单与数量（`SpawnDeckData`） |
| **Unlock** | 否 | 解锁条件 |

## 行为说明

### `SelectCard(setting)`
按 `SelectCardSetting` 从牌组里**顺序**筛出第一张符合的卡（**非随机**）；命中则 clone 成 `RCG_CardBattleData` 回传。

### `SelectCards(count, setting)`
TODO：尚未实作（程式内 `// ToDo` 直接 return null）。

### `GetAllCards()`
回传整副牌组的 `RCG_CardGenData` 清单。

### Tooltip Infos
`Infos = m_Deck.Infos`：聚合所有卡上的状态效果说明（例：「出血」、「燃烧」状态解说）。

## 注意事项

*   **`SelectCards` 还没实作**：要批量随机抽卡，目前不能用此入口；需自己实作或走别的 utility。
*   **预设 ID `Default` / `BackUp`**：`RCG_DeckGenData.DefaultID = "Default"`、`BackUpID = "BackUp"`——常作为「初始牌组」与「备用牌组」的命名约定。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_DeckData.cs`
*   **继承自**：`RCG_Asset<RCG_DeckData>`
*   **实作介面**：`RCGI_Unloackable`
*   **AssetGroup**：`EditItems`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Deck` | Deck | `SpawnDeckData` | 卡牌清单 + 配置 |
| `m_Unlock` | Unlock | `RCG_UnlockEntry` | |

### A.3 重要 Method

*   **`SelectCard(setting)`** — 按条件顺序选一张，clone 成 `RCG_CardBattleData`。
*   **`SelectCards(count, setting)`** — **未实作**（return null + TODO）。
*   **`GetAllCards()`** — 全副牌列表。
*   **`AllCardsName / Infos / LocalizedName`** — 显示用属性。

### A.4 与其他系统的互动

*   **`SpawnDeckData`** — 实际牌组容器。
*   **`RCG_CardBattleData`** — 战斗用卡片实例。
*   **`SelectCardSetting`** — 卡牌选择规则。
*   **`RCG_DeckGenData`** — Asset Entry；`Default` / `BackUp` 两个常数 ID。

### A.5 已知议题

*   `SelectCards` 尚未实作（`// ToDo`）。
