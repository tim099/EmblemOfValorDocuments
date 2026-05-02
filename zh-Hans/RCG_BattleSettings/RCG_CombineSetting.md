---
title: RCG_CombineSetting 说明
description: 组合战斗设定 (CombineSetting) 的职责、字段语义与行为合并规则
last_updated: 2026-05-02
target_audience: [AI_Agent, Gameplay_Programmer, Designer]
---

# RCG_CombineSetting

## 1. 职责 (Responsibility)
将多个 `RCG_BattleSetting` 合并为**单一**设定来依序或同时执行，是组合复合行为（例如「造成伤害 + 抽牌 + 加状态」）的最常用容器。

继承自：`RCG_BattleSetting`

## 2. 主要字段

| 字段 | 类型 | 物理意义 |
|---|---|---|
| `m_OverrideDescription` | `bool` | 是否覆写子设定串接出来的描述。`true` 时改用 `m_Description`，避免子设定描述叠加过长。 |
| `m_Description` | `RCG_LocalizeData` | 覆写描述本体；仅在 `m_OverrideDescription = true` 时生效（由 `Conditional` 属性管控可见性）。 |
| `m_CombineSettings` | `List<RCG_BattleSetting>` | 要组合执行的子战斗设定清单。`AlwaysExpendOnGUI` 让它在 Inspector 中永远展开。 |

## 3. 行为合并规则

### 3.1 可玩判定 `CheckPlayable`
**全部** 子设定皆 `CheckPlayable == true` 才视为可玩；任一失败即整体不可玩（AND 逻辑）。

### 3.2 预览伤害 `GetPreviewDamage`
取所有子设定 `GetPreviewDamage` 的**最大值**作为代表（**MAX 逻辑**）。即：UI 显示的攻击力 = 子设定中最强那一刀。

### 3.3 战斗标签 / 卡片资讯
`Infos` / `GetBattleTags()` 采**去重串接**，避免同类型 Buff/Tag 在多个子设定间重复堆叠显示。

### 3.4 描述生成 `GetDescription`
*   **未覆写**：将每个子设定的 `GetDescription` 换行串接（跳过空字串）。
*   **覆写**：直接返回 `m_Description.Name`，子设定不再参与描述合成。

## 4. 设计建议
*   **拆分粒度**：保持每个子设定只负责单一行为（damage、draw、buff 等），由 CombineSetting 负责组装，方便重用与单元测试。
*   **避免嵌套过深**：CombineSetting 内再放 CombineSetting 虽可行，但会让 `GetPreviewDamage` 的 MAX 逻辑变得反直觉，建议最多两层。
