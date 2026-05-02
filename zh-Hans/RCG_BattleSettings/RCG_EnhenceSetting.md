---
title: 强化 说明
description: 对选取的手牌附加强化效果（OnPlay 时触发），永久改造卡片行为
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 强化

> 程式类别名称：`RCG_EnhenceSetting`

## 用途
**对选取的手牌附加额外效果**：被强化的卡打出时会在原本的效果之外，**多触发一次** `EnhenceSetting`。常见用途：
*   「强化下一张攻击卡：附加吸血效果」
*   「强化所有手牌：每张多抽 1 张卡」

被强化的卡会永久带有这个效果，直到被「**无效卡牌效果**」削弱或卡片消失。

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **SelectHandCardSetting** | 是 | 选取要强化的手牌。 |
| **EnhenceSetting** | 是 | 附加上去的「**组合效果**」 — 触发时机固定为 `OnPlay`。 |

## 行为说明
*   选牌后，对每张卡呼叫 `aCard.Enhence(commonEffect)`，把 `EnhenceSetting` 包进 `RCG_CommonEffect`（`OnPlay` 触发点）并附加。
*   描述格式：「**{选取} : {强化标题} {强化内容描述}**」（i18n key `EnhenceSettingDes`，含 `EnhenceTitle` 颜色）。
*   附带资讯框（`Infos`）：第一条为「强化说明」总览，后接强化效果本身的 Infos。

### 标签
强化设定不会把自己内含的标签（`m_EnhenceSetting.GetBattleTags`）暴露在外层 — 是设计选择，避免标签被误计入原卡。

## 注意事项
*   **强化是永久的**：一旦强化效果附加上去，会跟着卡到其消失或被削弱。
*   **触发时机固定 `OnPlay`**：目前**不能改变**强化效果的触发点。要其他时机请考虑用「条件判断」+「状态」实作。
*   **与「无效卡牌效果」的对应**：被削弱时优先消耗强化叶效果，本设定的内容首当其冲。
*   **不暴露标签**：刻意设计 — 不要期待强化内含的「易碎」「敏捷」等标签会出现在卡片资讯上。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_EnhenceSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_EnhenceSetting` → 「强化」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_SelectHandCardSetting` | `SelectHandCardSetting` | `RCG_SelectHandCardSetting` | — | `[SerializeField] protected` |
| `m_EnhenceSetting` | `EnhenceSetting` | `RCG_CombineSetting` | — | `[SerializeField] protected` |

### A.3 重要 Method 摘要
*   **`AddAction`**：选牌 → 对每张卡建 `RCG_CommonEffect { m_CombineSetting = m_EnhenceSetting, m_EffectTriggerOn.m_TriggerOn = OnPlay }`，呼叫 `aCard.Enhence(aEffect)`。
*   **`GetBattleTags`** → 永远回传空清单（**刻意覆写**，不暴露 `m_EnhenceSetting` 的标签）。
*   **`GetDescriptionFormat`**：「{选取 desc}\n{Title} : {Enhence desc + 内含 BattleTags}」。
*   **`GetEnhenceSettingDescription` (private)**：取 `m_EnhenceSetting.GetDescription` + 其 BattleTags 描述。
*   **`Infos`**：插入 `"RCG_EnhenceSetting" + "\n" + EnhenceSettingInfo(...)` 为第 0 项。

### A.4 与其他系统的互动
*   **`RCG_CommonEffect`**：强化效果包覆容器（含 `TriggerOn`）。
*   **`RCG_CardBattleData.Enhence(commonEffect)`**：实际的强化附加入口。
*   **`RCG_Extensions.TagColors.EnhenceTitle`**：标题颜色。
*   **i18n keys**：`RCG_EnhenceSetting` / `EnhenceSettingDes` / `EnhenceSettingInfo`。

### A.5 已知议题
*   旧版 `m_CostAlter` / `m_EffectTriggerTiming` 栏位已注解（强化系统重构中），目前无法调整费用变化或自订触发点。
