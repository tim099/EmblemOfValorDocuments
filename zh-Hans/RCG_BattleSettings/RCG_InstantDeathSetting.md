---
title: 即死 说明
description: 直接击杀目标；可选无条件即死或低于指定 HP 即死两种模式
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 即死

> 程式类别名称：`RCG_InstantDeathSetting`

## 用途
**直接击杀目标**（无视 HP / 护甲）。常见用途：
*   「斩首」「处决」类超强卡：对血量低于 X 的目标即死
*   特殊事件「无条件击杀」（剧情演出 / 诅咒）

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Type** | 是 | 即死模式：<br>• **Default** — 无条件即死全部目标<br>• **HP** — 目标 HP ≤ 指定值才即死 |
| **目标** (`Target`) | 是 | 即死目标选择器。 |
| **HP** | Type=HP 必填 | 即死的血量门槛；目标血量 ≤ 此值才即死。`Conditional` 控制显示。 |

## 行为说明
*   **Default**：对所有目标呼叫 `RCG_TermInstantDeath.TriggerInstantDeath(iData, targets)`（内含词条触发）。
*   **HP**：对每个目标独立判定 `target.HP <= m_HP`，符合者呼叫 `target.InstantDeath()`。
*   描述格式：
    *   `Default` → 「**对 {目标} 即死**」
    *   `HP` → 「**血量低于 {HP} 时对 {目标} 即死**」

### 卡片资讯
显示 `RCG_Term.GetTerm(Term.InstantDeath)` 词条框（解释即死机制）。

## 注意事项
*   **Default 模式很强**：对 Boss 也直接击杀。除非 Boss 有「无视即死」状态，否则会被秒。设计时请慎用。
*   **HP 模式对 AOE 公平**：每个目标独立判定，不会出现「整批 AOE 但只击杀第一个」的问题（与其他即死实作的差异）。
*   **负值 HP**：`m_HP` 设负值会永远不触发（因为目标血量不会低于 0），等于「永远不即死」。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_InstantDeathSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_InstantDeathSetting` → 「即死」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_Type` | `Type` | `InstantDeathType` (档内) | — | `Default` / `HP` |
| `m_Target` | 目标 | `RCG_SelectTargetData` | `Target` | |
| `m_HP` | `HP` | `IntVariable` | `HP` (=「生命」) | `[Conditional(nameof(m_Type), false, HP)]`，预设 1 |

### A.3 重要 Method 摘要
*   **`AddAction`**：依 `m_Type` 分流：
    *   `Default` → `RCG_TermInstantDeath.TriggerInstantDeath(iData, targets)`。
    *   `HP` → 对每个 target 检查 `target.HP <= m_HP.GetValue(iData)` 即呼叫 `target.InstantDeath()`。
*   **`Info`** → `new CardInfoData(RCG_Term.GetTerm(Term.InstantDeath))`。
*   **`GetDescriptionFormat`**：`InstantDeath_Des` (Default) / `InstantDeath_HP_Des` (HP)。

### A.4 与其他系统的互动
*   **`RCG_TermInstantDeath.TriggerInstantDeath`**：词条触发入口，含特效演出。
*   **`RCG_BattleUnit.InstantDeath`**：直接设定 HP=0 并走死亡流程的入口。
*   **`Term.InstantDeath`**：对应的词条 enum。

### A.5 已知议题
*   `Condition` 模式（`m_Conditions`）已被注解掉 — 想做「条件式即死」目前要用「条件判断」+「即死(Default)」组合。
