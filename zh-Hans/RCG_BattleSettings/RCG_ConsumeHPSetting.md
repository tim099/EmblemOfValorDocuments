---
title: 消耗生命值 说明
description: 对目标扣血（固定值或当前 / 最大 HP 百分比）；用于自损型效果
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 消耗生命值

> 程式类别名称：`RCG_ConsumeHPSetting`

## 用途
**对目标扣血**（不视为攻击，因此**不受护甲影响**）。常见用途：
*   「自损 5 点 HP 换取大量资源」
*   「消耗 30% 当前 HP 触发强力效果」

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **ConsumeType** | 是 | 消耗模式：<br>• **CurHPPercentage** — 当前 HP 的百分比（预设）<br>• **MaxHPPercentage** — 最大 HP 的百分比<br>• **Value** — 固定数值 |
| **值** (`Amount`) | 是 | 消耗量（变数可变）；百分比模式为 0~100。 |
| **目标** (`Target`) | 是 | 目标选择器（通常选自身）。 |

## 行为说明
*   依 `ConsumeType` 计算实际扣血量：
    *   `CurHPPercentage` → `0.01 * Amount * 当前 HP` 取整
    *   `MaxHPPercentage` → `0.01 * Amount * 最大 HP` 取整
    *   `Value` → 直接 `Amount`
*   呼叫 `target.UnitHeal(-aConsumeAmount)` 扣血。
*   数值会记入 `RCG_BattleManager.Ins.m_BattleStats.LogStatsCostHealth(...)`。
*   描述格式：`ConsumeHP_{ConsumeType}_Des`（会出现心型图示）。

## 注意事项
*   **百分比模式的取整**：使用 `RoundToInt`，边界值（如 1% × 50 HP = 0.5）会四舍五入到 0 或 1。设计时请以**整数玩家可预测**为原则。
*   **不会直接击杀**：扣血至 0 会触发死亡判定，但若想「强制击杀」请改用「**即死**」设定。
*   **不视为攻击**：护甲、反击、伤害修饰都不适用 — 纯粹扣血。
*   **目标选敌方**：技术上合法（给敌方扣血），但这通常该用「攻击」设定 — 两者语意不同。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ConsumeHPSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_ConsumeHPSetting` → 「消耗生命值」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_ConsumeType` | `ConsumeType` | enum | — | 3 种模式 |
| `m_Amount` | 值 | `IntVariable` | `Amount` | `[UCL_FieldOnGUI]`，预设 50 |
| `m_Target` | 目标 | `RCG_SelectTargetData` | `Target` | |

### A.3 重要 Method 摘要
*   **`AddAction`**：依 ConsumeType 计算 `aConsumeAmount`，呼叫 `aTarget.UnitHeal(-aConsumeAmount)`，最后 `LogStatsCostHealth`。
*   **`GetDescriptionFormat`**：i18n key `ConsumeHP_{ConsumeType}_Des`，内含心型 sprite。
*   **`GetDescriptionShort`** → 直接 `m_Amount.GetDes(true)`。

### A.4 与其他系统的互动
*   **`RCG_BattleUnit.UnitHeal(int)`**：负值代表扣血（共用治疗入口）。
*   **`RCG_BattleManager.m_BattleStats.LogStatsCostHealth`**：战斗统计记录。
