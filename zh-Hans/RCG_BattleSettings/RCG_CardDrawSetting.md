---
title: 抽牌 说明
description: 从牌堆 / 弃牌堆抽取指定张数，可限定卡牌标签
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 抽牌

> 程式类别名称：`RCG_CardDrawSetting`

## 用途
**抽牌**到手牌。支援从**牌堆顶**（一般抽牌）、**牌堆内选取**（玩家挑）、**弃牌堆内选取**（捡回来）三种来源。

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **DrawCardNum** | 是 | 抽取张数（支援变数）。实际抽取会夹到 `[0, 玩家手牌空位]`。 |
| **DrawType** | 是 | 抽取方式：<br>• **FromDeckTop** — 从牌堆顶抽（一般抽牌，预设）<br>• **PickFromDeck** — 从牌堆中**选择**抽<br>• **PickFromDiscardPile** — 从弃牌堆**选择**抽（捡牌） |
| **CardTags** | 否 | 限定抽取有此标签的卡（如「治疗卡」）。 |
| **DrawCardNotIncludedTags** | 否 | 反向：抽**不含**这些标签的卡。 |

## 行为说明
*   `FromDeckTop` 直接抽 N 张到手牌，**手牌已满则不抽多余的**。
*   `Pick*` 模式会跳出选牌 UI 让玩家选择 N 张。
*   描述格式为「**抽 {张数} 张 {标签} 卡**」（i18n key `DrawCardIcon_{DrawType}`）。

### 卡片融合
两张「抽牌」融合会把张数加总（**不检查 DrawType / CardTags 是否相同**，可能踩坑）。

## 注意事项
*   **手牌空位限制**：`DrawCardNum` 会被夹到 `RCG_Player.Ins.CardSpace`，要抽的张数超过空位的部分会被吃掉而非触发弃牌。
*   **标签过滤与牌堆耗尽**：限定标签抽牌时，若牌堆内无符合条件的卡，**不会自动洗弃牌堆**，可能抽到比 N 少。
*   **PickFrom* 模式对 AI 无效**：选牌 UI 是给玩家用的，敌人若触发此设定会无动作（建议 AI 一律用 `FromDeckTop`）。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardDrawSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **`[System.Serializable]`** 标记
*   **i18n 类别名 key**：`RCG_CardDrawSetting` → 「抽牌」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_DrawCardNum` | `DrawCardNum` | `IntVariable` | — | `[UCL_FieldOnGUI]` |
| `m_DrawType` | `DrawType` | `DrawType` (档内 enum) | — | 3 种来源 |
| `m_CardTags` | `CardTags` | `List<RCG_CardTagGenData>` | — | |
| `m_DrawCardNotIncludedTags` | `DrawCardNotIncludedTags` | `bool` | — | |

### A.3 重要 Method 摘要
*   **`AddAction`**：`Mathf.Clamp(m_DrawCardNum, 0, RCG_Player.Ins.CardSpace)` → 依 `m_DrawType` 分流：
    *   `FromDeckTop` → `CreateAction.AddDrawCardAction(iData, ...)`。
    *   `PickFromDeck` → `CreateAction.AddSelectCardAction(CardPos.Deck, ...)` + `AddPickFromDeckAction`。
    *   `PickFromDiscardPile` → 同上但 `CardPos.DiscardPile`。
*   **`Fusion(other)`**：clone + `IntVariable.FuseAdd(m_DrawCardNum)`，**不检查其他栏位**。
*   **`LocalizeKey`** → `"DrawCardIcon_" + m_DrawType.ToString()`。

### A.4 与其他系统的互动
*   **`SelectCardSetting`**：`RCG_CardDiscardSetting.cs` 中定义的工具类，用于标签过滤。
*   **`CreateAction.AddSelectCardAction / AddPickFromDeckAction / AddDrawCardAction`**：抽牌动作建构工具。
*   **`RCG_Player.Ins.CardSpace`**：手牌空位上限。
