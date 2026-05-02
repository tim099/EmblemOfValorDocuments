---
title: 无效卡牌效果 说明
description: 削弱选取的手牌：移除指定数量的叶效果，从强化效果优先消耗
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 无效卡牌效果

> 程式类别名称：`RCG_DiminishSetting`

## 用途
**削弱选取的手牌** — 移除一定数量的「叶效果」（最末端的单一动作，例如「攻击」「治疗」「抽牌」）。优先从强化效果开始消耗，耗尽后才动到基础效果。常见用途：
*   敌人技能：「使你下一张牌效果减半」
*   诅咒卡：「目标卡的某项效果被消除」

被削弱的叶效果会被 `RCG_DiminishedPlaceholder` 取代（显示「已被削弱」红字）。

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **SelectHandCardSetting** | 是 | 选取要削弱的手牌。 |
| **DiminishEffectCount** | 是 | 要移除的叶效果数量（预设 1）。 |

## 行为说明
*   选牌后，对每张卡呼叫 `aCard.Diminish(token, DiminishEffectCount)`。
*   消耗顺序：**强化效果**先消耗，耗尽再动到**基础效果**。
*   每被消耗的叶效果会替换为 `RCG_DiminishedPlaceholder`，描述显示为「已被削弱」（红字）。

## 注意事项
*   **不可被进一步削弱**：被替换为 `RCG_DiminishedPlaceholder` 的叶效果**不能再次被削弱**（避免无限叠加）。
*   **叶效果不一定明显**：「组合效果」「条件判断」内部含多个叶效果；`DiminishEffectCount = 1` 不一定能砍到「最关键的那一个」。
*   **DiminishEffectCount 太大**：超过卡片叶效果总数时，整张卡几乎全变 placeholder — 设计时请拿捏。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_DiminishSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_DiminishSetting` → 「无效卡牌效果」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_SelectHandCardSetting` | `SelectHandCardSetting` | `RCG_SelectHandCardSetting` | — | `[SerializeField] protected` |
| `m_DiminishEffectCount` | `DiminishEffectCount` | `int` | — | `[SerializeField] protected`，预设 1 |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：选牌 → 对每张 `aCard.Diminish(iToken, m_DiminishEffectCount)`。
*   **`GetBattleTags`** → 永远回传空（被削弱的卡不继承标签）。
*   **`GetDescription`**：「{选取描述}\n{被削弱说明} ({DiminishEffectCount}个)」（i18n key `DiminishSettingDes`）。
*   **`GetDescriptionParams`**：包含 `Title`、`DC`（DiminishEffectCount 字串）、选取设定的子 params。

### A.4 与其他系统的互动
*   **`RCG_CardBattleData.Diminish(token, count)`**：实际削弱的入口；负责从强化端开始消耗叶效果并替换为 placeholder。
*   **`RCG_DiminishedPlaceholder`**：占位类别。
*   **`RCG_Extensions.TagColors.EnhenceTitle`**：标题颜色（borrow 自强化系统）。
