---
title: 清除意图 说明
description: 清除目标单位的下回合意图（敌人 AI 的攻击计划）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 清除意图

> 程式类别名称：`RCG_ClearIntentSetting`

## 用途
**清除敌人的下回合意图**（Intent）— 也就是头顶显示的「下回合要做的动作」。常见用途：
*   「使敌人的下次攻击失效」风格的卡
*   「打断」「干扰」类的设定

清除后敌人会在下次决策时重新计算意图。

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **目标** (`Target`) | 是 | 要清除意图的目标选择器；可选敌方单体或全体。 |

## 行为说明
*   对每个目标（**有 AI 才有效**）呼叫 `target.UnitAI.ClearIntent()`。
*   描述格式为「**清除 {目标} 的意图**」（i18n key `ClearIntentDes`）。

## 注意事项
*   **无 AI 的目标跳过**：友军角色通常无 AI，使用此设定对友军无效。
*   **不会立即重决策**：清除后**等下次 AI 决策时机才会生成新意图**；当回合内敌人不会立刻重新攻击。
*   **意图中已执行的部分不会回退**：如果敌人已经开始打出某段攻击，清除意图只会影响尚未发生的部分。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ClearIntentSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_ClearIntentSetting` → 「清除意图」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_Target` | 目标 | `RCG_SelectTargetData` | `Target` | |

### A.3 重要 Method 摘要
*   **`AddAction`**：对每个 target 检查 `target.HasAI`，是的话呼叫 `target.UnitAI.ClearIntent()`。
*   **`GetDescriptionFormat`** → i18n key `ClearIntentDes`。

### A.4 与其他系统的互动
*   **`RCG_BattleUnit.HasAI / UnitAI.ClearIntent`**：AI 系统的意图清除入口。
