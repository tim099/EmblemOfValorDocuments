---
title: 最大生命值变动 说明
description: 修改目标的最大 HP；可选永久或本场战斗两种范围
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 最大生命值变动

> 程式类别名称：`RCG_MaxHPAlterSetting`

## 用途
**修改目标的最大 HP**。常见用途：
*   永久增加角色血量（战斗奖励 / 等级提升）
*   本场战斗内提升血量（暂时 Buff，战后恢复）

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **AlterType** | 是 | 变化范围：<br>• **Permanently** — 永久增加最大 HP（战后保留）<br>• **ThisBattle** — 只在本场战斗（战后恢复） |
| **值** (`Amount`) | 是 | 变化量（变数可变，可正可负）。 |
| **目标** (`Target`) | 是 | 目标单位选择器。 |

## 行为说明
*   对未死亡的目标呼叫 `target.AlterMaxHP(AlterType, Amount)`。
*   描述格式：「**{Permanently/ThisBattle} 增加 {Amount} 最大 HP 给 {Target}**」（i18n key `MaxHPIcon_Des`，含心型图示）。
*   执行后等待 0.6 秒让 UI 动画播完。

### 卡片融合
两张本设定融合会把 Amount 加总（**不检查 AlterType 是否相同**，可能会踩坑）。

## 注意事项
*   **AlterType 不对齐的融合**：永久 + 本场融合后 AlterType 取**前者**，可能造成「永久升级被误标为暂时」。
*   **负值 Amount = 削减 HP 上限**：合法，但同时会把目标当前 HP 上限拉低（**若当前 HP 超过新上限会被截断？** — 详情查 `AlterMaxHP` 实作）。
*   **死亡目标跳过**：已死亡的目标不会被修改，避免死人加血。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_MaxHPAlterSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_MaxHPAlterSetting` → 「最大生命值变动」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_AlterType` | `AlterType` | `AlterType` (档内 enum) | — | `Permanently` / `ThisBattle` |
| `m_Amount` | 值 | `IntVariable` | `MaxHP` (=「生命值上限」) | `[UCL_FieldOnGUI]` 已被注解 |
| `m_Target` | 目标 | `RCG_SelectTargetData` | `Target` | |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：对 `aTargets` 中未死亡的呼叫 `aTarget.AlterMaxHP(m_AlterType, aAmount)`，后 `await Delay(0.6f)`。
*   **`Fusion`**：clone + `IntVariable.FuseAdd(m_Amount)`，**不检查 `m_AlterType` 一致性**。
*   **`GetDescriptionFormat`** → i18n key `MaxHPIcon_Des`，4 个参数（Amount / Target / 心型 sprite / AlterType localize name）。

### A.4 与其他系统的互动
*   **`RCG_BattleUnit.AlterMaxHP(AlterType, int)`**：实际的修改入口；负责处理当前 HP / 上限的同步。
*   **`EffectIcon.Health`**：心型图示。

### A.5 已知议题
*   旧版 `CanEnhence / Enhence` 已注解；强化系统重构中。
