---
title: 词条 说明
description: 触发指定词条（Term）所定义的效果；词条由独立的 RCG_Term 系统管理
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 词条

> 程式类别名称：`RCG_TermSetting`

## 用途
**触发词条（Term）效果**。「词条」是游戏中可重用的命名效果，例如「**消耗**」「**咏唱**」「**保留**」「**即死**」「**急速咏唱**」等 — 每个词条都有独立的触发逻辑与卡片资讯面板。

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Term** | 是 | 要触发的词条 enum 值（预设 `HighSpeedChant` 急速咏唱）。 |

## 行为说明
*   触发时呼叫 `RCG_Term.GetTerm(m_Term).TriggerEffect(iData, endAction)`。
*   描述显示词条的本地化名称（绿色标题色）。
*   **`HasTerm(iTerm)` 回传 true** 当 `iTerm == m_Term` — 上层辞条反查能找到此设定。
*   `Info` 显示对应词条的完整 Tooltip（除非 `Term.Empty`）。

## 注意事项
*   **词条本身决定行为**：本设定只是「触发**已定义的词条**」 — 词条逻辑在 `RCG_Term.GetTerm(...)` 那边。要新增 Term 请查 Term 系统。
*   **Term.Empty**：合法但无效；常用于占位或未来扩充。
*   **词条触发点**：「保留」「消耗」这类词条本身有触发时机（例如「打出时」「丢弃时」）— 本设定主动触发 `TriggerEffect` 是另一回事，请确认词条本身支援主动呼叫。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_TermSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_TermSetting` → 「词条」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_Term` | `Term` | `Term` (enum) | `Term` (=「词条」) | 预设 `HighSpeedChant` |

### A.3 重要 Method 摘要
*   **`AddAction`**：`RCG_Term.GetTerm(m_Term).TriggerEffect(iData, iEndAction)`。
*   **`HasTerm(Term iTerm)` (override)** → `iTerm == m_Term`；上层 `GetBattleSettings<RCG_TermSetting>` 后可进一步查辞条归属。
*   **`Info`** → `m_Term == Empty` 回 null；否则 `new CardInfoData(RCG_Term.GetTerm(m_Term))`。
*   **`TermDes` (private property)** → `LocalizeName.GetTagColor(TagColors.Term)`。

### A.4 与其他系统的互动
*   **`RCG_Term.GetTerm(Term)`**：词条工厂；回传对应的 Term 逻辑实例。
*   **`Term` enum**：词条 ID 集合（`HighSpeedChant`、`InstantDeath`、`Retain`、`Empty`、...）。
*   **`RCG_Extensions.TagColors.Term`**：词条色票。

### A.5 已知议题
*   旧版 `m_Term == Term.Retain` 的反序列化兼容已被注解（强制覆写为 `Term.Empty` 的旧逻辑）。
