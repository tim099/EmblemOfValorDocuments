---
title: 消耗状态 说明
description: 消耗自身状态层数触发效果；层数不足时可选择阻挡使用或不触发
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 消耗状态

> 程式类别名称：`RCG_ConsumeStatusSetting`

## 用途
**消耗自身状态层数来触发效果**。例如「消耗 1 层剑气护体 → 造成 20 点物伤」。常见用途：
*   建立「累积 + 爆发」风格的卡牌组合
*   让某些强力效果需要事前蓄能

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **状态** (`Status`) | 是 | 要消耗的状态类型（`RCG_CustomStatusGenData`）。 |
| **值** (`Amount`) | 是 | 消耗多少层；支援变数。 |
| **Setting** | 是 | 消耗成功后执行的「**组合效果**」。 |
| **详细设定** (`DetailSetting`) | 否 | 与「状态」设定的详细设定共用（控制动画、Tag 等）。 |
| **CheckPlayable** | — | 预设不打勾。打勾 = 层数不足时**不可打出**；不打勾 = 仍可打出但不触发效果。 |

## 行为说明
*   **层数判定**：以 `iData.User`（自身）的当前状态层数比对 `Amount`。
*   **CheckPlayable 模式**：
    *   打勾：层数不足时这张卡灰掉不可用。
    *   不打勾：层数不足时卡片仍可打出，但 `m_Setting` 不会触发。
*   **触发**：扣除指定层数（`-amount`）→ 执行 `m_Setting`。
*   **预览伤害**：条件成立时取 `m_Setting` 的预览；不成立时回传 0（**非 -1**，差别在于不被视为「非攻击牌」）。
*   描述格式：「**消耗 {Amount} 层 {Status} 图示**」+ 子设定描述（i18n key `ConsumeStatusDes`）。

## 注意事项
*   **预览伤害不准的情况**：若状态层数在卡片计算时是动态的（例如「先抽牌再消耗」），预览可能与实际不符。
*   **Status 必须是可堆叠的状态**：对于不支援层数的状态，行为未定义。
*   **与「状态」设定的差别**：「状态」是给予层数；本设定是**消耗自身**层数。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ConsumeStatusSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_ConsumeStatusSetting` → 「消耗状态」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_Status` | 状态 | `RCG_CustomStatusGenData` | `Status` | |
| `m_Amount` | 值 | `IntVariable` | `Amount` | 预设 1 |
| `m_Setting` | `Setting` | `RCG_CombineSetting` | — | 子效果容器 |
| `m_DetailSetting` | 详细设定 | `RCG_StatusSetting.DetailSetting` | `DetailSetting` | 共用「状态」设定的 DetailSetting |
| `m_CheckPlayable` | `CheckPlayable` | `bool` | — | 预设 `false` |

### A.3 重要 Method 摘要
*   **`CheckCondition(iData, out amount)` (public)**：对 `iData.User.m_UnitStatus.GetStatusLayer(m_Status)` 比对 `m_Amount.GetValue(iData)`。
*   **`CheckPlayable`**：`m_CheckPlayable` 为 false 时恒为 true；否则跑 `CheckCondition`。
*   **`AddAction`**：条件成立 → `CreateAction.StatusAction(User, Status, -amount, ..., DetailSetting)` + `m_Setting.AddAction(InsertInOrder)`。
*   **`GetBattleSettings<T> / (Type)`**：自身 + 递回 `m_Setting`。
*   **`GetDescriptionFormat`**：暂存 `m_FullSentence = false` → i18n key `ConsumeStatusDes` + 子设定 format → 复原并套句尾。
*   **`GetPreviewDamage`**：条件不成立回 0（**非 -1**）；成立则代理到 `m_Setting`。
*   **`PreloadData`**：await `m_Setting.PreloadData`。

### A.4 与其他系统的互动
*   **`RCG_UnitStatus.GetStatusLayer(status)`**：当前层数查询入口。
*   **`CreateAction.StatusAction`**：实际扣层数的 Action 工具。
*   **`RCG_CustomStatusGenData.IconTMPKey`**：描述中内嵌的图示。
