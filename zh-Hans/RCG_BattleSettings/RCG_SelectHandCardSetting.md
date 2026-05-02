---
title: 选取手牌 说明
description: 开启选牌 UI（或自动选），把选中的卡写入后续设定可引用的 SelectedHandCards
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 选取手牌

> 程式类别名称：`RCG_SelectHandCardSetting`

## 用途
**让玩家（或系统）选取手牌** — 将结果存入 `SelectedHandCards` 给后续设定使用（如「弃牌」「强化」「复制」「融合」「卡牌转物品」等）。是「**前置设定**」性质。

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **SelectType** | 是 | 选取方式（**13 种**，最常用见下方）。 |
| **SelectRange** | SelectFromRange* 必填 | 选取范围（手牌 / 牌堆 / 弃牌堆，可多选）。 |
| **SelectNum** | SelectByCount/RandomSelect/RandomOneCardWithCost/LeftSide(Of)/RightSide 必填 | 选取张数（或费用值）。 |
| **CardTags** | 否 | 限定有此标签的卡。 |
| **SelectCardNotIncludedTags** | — | 反向选取（不含这些标签）。 |
| **SelectCardWithoutEnhancement** | — | 限定未强化的卡。 |
| **SelectCountVariable** | 否 | 把选取张数写入变数（后续设定可引用）。 |
| **DetailSetting** | 否 | 描述格式微调（`Default` / `Then` / `SelectPrefix`）。 |

### 主要选取方式（SelectType）
| 值 | 行为 |
|---|---|
| **SelectByCount** | 玩家选 N 张（最常用，预设） |
| **SelectAnyCount** | 玩家任意数量 |
| **SelectAll** | 自动全选（无 UI） |
| **SkipSelect** | 跳过选取（前面已选好） |
| **CreatedCards** | 自动选取本次效果生成的卡 |
| **RandomSelect** | 随机选 N 张 |
| **SelectFromRange** | 从手牌 / 牌堆 / 弃牌堆混合选 |
| **SelectFromRangeAll** | 从多个范围**全选** |
| **ThisCard** / **TriggeredCard** | 这张触发中的卡 |
| **LeftSide** / **RightSide** | 该卡左 / 右侧 N 张 |
| **LeftSideOfThis** | 此卡左侧的卡 |
| **RandomOneCardWithCost** | 随机选一张费用为 X 的卡（X 由 `SelectNum` 指定） |

## 行为说明
*   **执行流程**：开启选择 UI（或依模式自动选），将结果填入 `iData.SelectedHandCards`。
*   **后续引用**：下游设定（如弃牌、强化）读取 `iData.SelectedHandCards`。
*   **变数写入**：`SelectCountVariable` 非空时，把实际选取张数写入该变数。
*   **描述**：根据 `DetailSetting.DescriptionType` 决定句型。

## 注意事项
*   **本设定通常不单独使用**：它只负责「选取」并填值；要对选中的卡做事，**后面要接** 弃牌 / 强化 / 复制 / 融合 / ... 等设定。
*   **AI 触发**：玩家选择 UI 对 AI 无效；AI 用此设定请选 `RandomSelect` / `SelectAll` / `SkipSelect` 等不需玩家输入的模式。
*   **SelectFromRange 范围空清单**：等于「不进行选取」。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_SelectHandCardSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_SelectHandCardSetting` → 「选取手牌」
*   **同档内巢状类**：`SelectType` (13 个值) / `DescriptionType` (3 个值) / `DetailSetting` (含 `m_DescriptionType`)

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_SelectType` | `SelectType` | enum | — | 13 种选取方式 |
| `m_SelectRange` | `SelectRange` | `List<CardPos>` | — | `[Conditional(nameof(m_SelectType), false, SelectFromRange, SelectFromRangeAll)]` |
| `m_SelectNum` | `SelectNum` | `IntVariable` | — | `[Conditional(... 6 种需要数量的 SelectType)]` |
| `m_CardTags` | `CardTags` | `List<RCG_CardTagGenData>` | — | |
| `m_SelectCardNotIncludedTags` | `SelectCardNotIncludedTags` | `bool` | — | |
| `m_SelectCardWithoutEnhancement` | `SelectCardWithoutEnhancement` | `bool` | — | |
| `m_SelectCountVariable` | `SelectCountVariable` | `string` | — | |
| `m_DetailSetting` | `DetailSetting` | `DetailSetting` (档内) | `DetailSetting` | |

### A.3 重要 Method 摘要
*   **`AddAction`**（档案 100+ 行外）：依 `m_SelectType` 14 种分支 — 开 UI / 自动选 / 范围选 / 随机选等，最终填入 `iData.SelectedHandCards`。
*   **`TagDes` (private)**：标签描述生成；含 `NotIncludedDes` 与 `SelectCardWithoutEnhancementDes` 两种额外修饰。
*   **`GetDescriptionFormat`** / **`GetDescriptionParams`**：依 `m_DetailSetting.m_DescriptionType` 变化句型。

### A.4 与其他系统的互动
*   **`iData.SelectedHandCards`**：填值目标；下游设定读取此清单。
*   **`CardPos` enum**：选取范围类型（Hand / Deck / DiscardPile）。
*   **`CreateAction.AddSelectCardAction`**：开 UI 选牌的入口。
