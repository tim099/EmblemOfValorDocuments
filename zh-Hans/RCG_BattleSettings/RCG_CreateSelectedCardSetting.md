---
title: 生成卡牌(多选) 说明
description: 从候选清单中让玩家选择卡牌生成（或从牌库乱数选），可指定加入位置与张数
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 生成卡牌(多选)

> 程式类别名称：`RCG_CreateSelectedCardSetting`

## 用途
从**候选清单**或**牌库**让玩家挑选卡牌生成 — 比「**生成卡牌**」多了「**从多选一**」的环节。常见用途：
*   「从 3 张随机奖励中选一张加入手牌」
*   「从整个牌库挑一张加入手牌」

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **CreateType** | 是 | 候选来源：<br>• **Default** — 从 `CardGenDatas` 设定的清单选<br>• **FromDeck** — 从指定牌库选 |
| **CreateCardType** | 是 | 加入位置（同「生成卡牌」）：手牌 / 牌堆 / 弃牌堆 / 牌堆顶。 |
| **CreateCount** | 是 | 玩家最多可选张数。 |
| **IsRandomSelect** | — | 打勾 = **随机**选择（不出 UI 直接抽）；不打勾 = 玩家手动选。 |
| **CardGenDatas** | CreateType=Default | 候选卡片清单（多张卡片模板）。 |
| **Deck** | CreateType=FromDeck | 候选牌库资料（`RCG_DeckGenData`）。 |

## 行为说明
*   `Default` + 玩家选择 → 从 `CardGenDatas` 开选择 UI 让玩家选 N 张。
*   `FromDeck` + 玩家选择 → 从指定牌库的所有卡开选择 UI。
*   `IsRandomSelect = true` → 不开 UI，直接随机抽 N 张。
*   选定后依 `CreateCardType` 加入指定位置。

## 注意事项
*   **与「生成卡牌」的差别**：「生成卡牌」直接生成写死的卡；本设定**有让玩家选**的环节（除非 `IsRandomSelect`）。
*   **CardGenDatas 重复的处理**：候选清单中重复出现的卡会在 `Infos` 自动去重；但选择 UI 仍可能展示多次。
*   **AI 触发此设定**：玩家选择 UI 对 AI 无效；建议 AI 用 `IsRandomSelect = true` 避免卡住。
*   **FromDeck 拉取整个牌库**：可能造成大量候选，UI 可能拥挤 — 请考虑是否要先过滤。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CreateSelectedCardSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_CreateSelectedCardSetting` → 「生成卡牌(多选)」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_CreateType` | `CreateType` | enum (档内) | — | `Default` / `FromDeck` |
| `m_CreateCardType` | `CreateCardType` | `CreateCardType` (共用) | — | 4 种位置 |
| `m_CreateCount` | `CreateCount` | `IntVariable` | — | 预设 1 |
| `m_IsRandomSelect` | `IsRandomSelect` | `bool` | — | |
| `m_CardGenDatas` | `CardGenDatas` | `List<RCG_CardGenData>` | — | `[Conditional("m_CreateType", false, Default)]` |
| `m_Deck` | `Deck` | `RCG_DeckGenData` | — | `[Conditional("m_CreateType", false, FromDeck)]` |

### A.3 重要 Method 摘要
*   **`Infos`**：`Default` 模式 hash 去重后加每张卡 info；`FromDeck` 模式加 deck info。
*   **`CardDatas` (private)**：依 CreateType 取候选 `RCG_CardData` 清单。
*   **`AddAction`**（档案 90 行外）：选牌 + 加入位置（依 IsRandomSelect 分流）。

### A.4 与其他系统的互动
*   **`RCG_DeckGenData`**：牌库资料来源。
*   **`CreateAction.AddSelectCardAction / CreateCard`**：UI 选牌与卡片插入入口。
