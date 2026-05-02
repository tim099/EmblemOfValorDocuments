---
title: 弃牌 说明
description: 将手牌移出（弃牌、消灭、回牌堆顶、保留等）；支援多种选取方式与标签筛选
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 弃牌

> 程式类别名称：`RCG_CardDiscardSetting`

## 用途
**将手牌移出当前位置**到指定处（弃牌堆 / 消灭 / 回牌堆顶等），是「弃牌卡」「消灭卡」的核心设定。

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **DiscardType** | 是 | 弃牌方式：<br>• **DiscardByCount** — 玩家自行选择 N 张<br>• **DiscardAll** — 全部手牌<br>• **DiscardAnyCount** — 玩家任意数量<br>• **RandomDiscardByCount** — 随机 N 张<br>• **SelectedCards** — 已被前置设定选好的卡<br>• **SelectHandCard** — 透过内嵌的选取设定 |
| **DiscardCardNum** | DiscardByCount/RandomDiscardByCount 时必填 | 弃牌张数（支援变数）。`Conditional` 控制显示。 |
| **RemoveType** | 是 | 移除方式：`Discard` 弃牌 / `Banish` 消灭 / `ToDeckTop` 回牌堆顶 / `TurnEndDiscard` 回合结束时弃牌（不触发弃牌效果）/ `Retain` 留在手牌。 |
| **SelectHandCardSetting** | DiscardType=SelectHandCard 时必填 | 内嵌的选取设定。 |
| **DiscardCardTags** | 否 | 限定要弃的卡的标签（如「攻击牌」）。 |
| **DiscardCardNotIncludedTags** | 否 | 反向选取（不含这些标签的卡）。 |
| **DiscardVariable** | 否 | 把实际弃掉的张数**写入变数**，后续设定可引用（例如「弃牌后抽 X 张」中的 X）。 |

## 行为说明
*   依 DiscardType 取得待弃卡清单，呼叫 `CreateAction.AddDiscardCardActions(...)` 并指定 `RemoveType`。
*   有 `DiscardVariable` 时，把张数存进 `iData.VariableDic[变数名]`。
*   `DiscardAnyCount` 模式下若有 `DiscardVariable`，描述会显示 `X` 而非 `N`。
*   描述会根据 `RemoveType` 套不同 i18n key（例如 `BanishCardIcon_Des` / `DiscardSelectedCardsIcon_Des`）。

### 卡片融合
仅 `DiscardByCount` 与 `RandomDiscardByCount` 之间可融合（张数加总）；其他模式不可融合。

## 注意事项
*   **TurnEndDiscard 不触发弃牌效果**：故意设计用于「锁定卡片在手中，回合结束强制清掉」的情境，不要拿来当一般弃牌。
*   **DiscardAnyCount 没有 DiscardCardNum**：填上限请改用 `DiscardByCount`；要任意张数请保持空白并善用 `DiscardVariable`。
*   **变数命名冲突**：若多个设定都用 `DiscardVariable = "X"`，后者会覆盖前者 — 请命名清楚。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardDiscardSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_CardDiscardSetting` → 「弃牌」
*   **同档案巢状类**：`SelectCardSetting`（支援以 `RCG_CardTagGenData` 过滤，含 `m_NotIncludedTags` 反选与 `m_WithoutEnhancement`）

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_DiscardType` | `DiscardType` | enum | — | 6 种模式 |
| `m_DiscardCardNum` | `DiscardCardNum` | `IntVariable` | — | `[Conditional(m_DiscardType, false, DiscardByCount, RandomDiscardByCount)]` |
| `m_RemoveType` | `RemoveType` | `RemoveType` (档内 enum) | — | `Discard` / `Banish` / `ToDeckTop` / `TurnEndDiscard` / `Retain` |
| `m_SelectHandCardSetting` | `SelectHandCardSetting` | `RCG_SelectHandCardSetting` | — | `[Conditional(m_DiscardType, false, SelectHandCard)]` |
| `m_DiscardCardTags` | `DiscardCardTags` | `List<RCG_CardTagGenData>` | — | |
| `m_DiscardCardNotIncludedTags` | `DiscardCardNotIncludedTags` | `bool` | — | |
| `m_DiscardVariable` | `DiscardVariable` | `string` | — | |

### A.3 重要 Method 摘要
*   **`AddAction`**：依 `m_DiscardType` 走 6 种分支；共通走 `CreateAction.AddDiscardCardActions(iData, list, mode, RemoveType)`。
*   **`Fusion`**：仅 `DiscardByCount` / `RandomDiscardByCount` 模式可融合；clone + `IntVariable.FuseAdd` 合并张数。
*   **`GetDescriptionFormat`**：依 RemoveType / DiscardType 双重分流套 i18n key（例如 `Banish{Type}Icon_Des`）。

### A.4 与其他系统的互动
*   **`CreateAction.AddSelectCardAction / AddDiscardCardActions`**：UI 选牌与弃牌动作建构工具。
*   **`RCG_GameManager.Random.RandomPick`**：RandomDiscard 的随机来源。
*   **`SelectCardSetting`**：标签过滤的工具类（同档案）。
