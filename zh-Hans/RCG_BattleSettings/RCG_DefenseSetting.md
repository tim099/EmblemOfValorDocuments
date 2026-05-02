---
title: 护甲 说明
description: 对目标增加 / 减少 / 清除 / 倍增 / 衰减 / 设定护甲层数
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 护甲

> 程式类别名称：`RCG_DefenseSetting`

## 用途
**修改护甲层数**。最常见的「防御卡」「Buff 卡」核心，支援多种变化模式。

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **DefenseAlterType** | 是 | 变化模式：<br>• **Add** — 增加护甲<br>• **Sub** — 减少护甲<br>• **Clear** — 清空护甲<br>• **Double** — 倍增护甲<br>• **Decay** — 自然衰减（**可被某些状态阻止**，与 Clear 不同）<br>• **Set** — 设为指定值 |
| **护甲** (`Defense`) | Add/Sub/Set 必填 | 护甲量（变数可变）。`Conditional` 控制显示 — `Clear/Double/Decay` 模式会自动隐藏。 |
| **DefenseTarget** | 是 | 护甲套用目标选择器（取代旧的 `m_Range`）。 |

## 行为说明
*   依 DefenseAlterType 套用：
    *   `Add` / `Sub` / `Set` — 直接加减 / 设定数值
    *   `Clear` — 强制清零（**不受任何状态保护**）
    *   `Decay` — 衰减（**可被某些状态（如「持盾」）阻止**）
    *   `Double` — 倍增当前护甲
*   描述格式依模式套不同 i18n key（`DefIcon_DesAdd` / `DefIcon_DesSub` / `DefIcon_DesClear` / `DecayDes` / ...）+ 护甲图示。

## 注意事项
*   **Decay vs Clear 的差别**：`Clear` 强制归零，`Decay` 可被防衰减状态阻止 — 设计回合结束的自然衰减一律用 `Decay`。
*   **Sub 不会变负值**：护甲下限为 0；扣超过会卡在 0 不会变负。
*   **永远展开**：`[AlwaysExpendOnGUI]`。
*   **与「状态」的差别**：护甲是**独立的数值**，不算状态层数；查询层数类条件不会看到护甲。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_DefenseSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **`[System.Serializable] + [AlwaysExpendOnGUI]`** 标记
*   **i18n 类别名 key**：`RCG_DefenseSetting` → 「护甲」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_DefenseAlterType` | `DefenseAlterType` | enum (档内) | — | 6 种模式 |
| `m_Defense` | 护甲 | `IntVariable` | `Defense` | `[UCL_FieldOnGUI]` + `[Conditional(m_DefenseAlterType, false, Add, Sub, Set)]` |
| `m_DefenseTarget` | `DefenseTarget` | `RCG_SelectTargetData` | — | 取代旧的 `m_Range` |

### A.3 重要 Method 摘要
*   **`AddAction`**（档案 90 行外）：依 DefenseAlterType 对 `m_DefenseTarget.GetTargets(iData)` 套用变化。
*   **`GetDescriptionFormat`**：依 DefenseAlterType 套不同 i18n key（`DefIcon_DesXxx` / `DecayDes` 等）。
*   **`Infos`**：`IsShowOnUI` 启用时加入护甲图示资讯。

### A.4 与其他系统的互动
*   **`EffectIcon.Armor.GetTMPSpriteName()`**：描述中的护甲图示。
*   **`RCG_BattleUnit` 护甲层**：实际的护甲容器；衰减判定可能由 `m_UnitStatus` 中的 buff 阻止。
