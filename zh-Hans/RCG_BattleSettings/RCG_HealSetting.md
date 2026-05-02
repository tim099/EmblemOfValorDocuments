---
title: 治疗设定说明
description: 对选定目标恢复生命值的战斗设定，支援固定值与百分比两种治疗方式
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 治疗

> 程式类别名称：`RCG_HealSetting`

## 用途
对**指定目标**恢复生命值。最简单的「回血卡」「治疗技」「自我修复状态」都用这个设定。

## 编辑器中的样貌
```
▼ ✓ [治疗(Heal)] 回复 1 点 HP 给 (目标)
    HealType    [数值 / 百分比]
    治疗量      [数值] 1
    治疗目标    ▶ (展开后设定目标范围)
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **HealType** | 是 | 治疗方式：<br>• **数值 (Amount)** — 直接回复指定点数，例：「回 5 点 HP」。<br>• **百分比 (Percentage)** — 依目标最大 HP 的百分比回复，例：「回 30% HP」。 |
| **治疗量** | 是 | 数字（支援变数绑定）。HealType 为「数值」时是点数，为「百分比」时是 0~100 的百分比值。 |
| **治疗目标** | 是 | 治疗对象选择器。可以是「自身」「我方全体」「选中目标」等多种型态，由「**目标选择器**」决定。 |

> [!NOTE]
> **治疗量** 支援「变数」（例如「等于剩余卡片数」「等于某状态层数」），不限定常数。下拉「数值/变数」即可切换。

## 行为说明

### 战斗中发生什么
1. 系统解析「治疗目标」拿到实际目标清单。
2. 若清单非空，依 **HealType** 对每个目标执行回血。
3. UI 上会跳出绿色数字（生命值上升）。

### 描述会怎么显示
*   **HealType = 数值**：「回复 {治疗量} 点 HP 给 {治疗目标}」（搭配红心图示）
*   **HealType = 百分比**：「回复 {治疗量}% HP 给 {治疗目标}」

### 卡片融合
两张「治疗」卡融合时，**治疗量会直接相加**（其他栏位以前者为准）。例如：「回 3 点」+「回 5 点」融合 → 「回 8 点」。

## 注意事项

*   **不要把「治疗」用在攻击牌上凑「攻击+回血」**：请改用「**组合效果**」包覆，把「攻击」和「治疗」分别作为子设定 — 两者语意正交，不该混在一起。
*   **治疗量为负值**：技术上不会 crash，但语意错误（治疗打负数变伤害？）。**要造成伤害请改用「攻击」设定**，不要钻这个漏洞。
*   **治疗目标选「敌方」**：合法但奇特（给敌方回血通常是 debuff 或玩笑卡）；确认这是设计意图再用。

---

## 附录：程式人员参考 (Programmer Reference)

> 此段以下使用程式内部术语，受众转为程式人员与 AI agent。前半段内容请优先采信。

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_HealSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_HealSetting` → 「治疗」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_HealType` | `HealType`（未本地化） | `HealType` (enum) | — | 列举值有 i18n：`HealType_Amount` (数值)、`HealType_Percentage` (百分比) |
| `m_HealAmount` | 治疗量 | `IntVariable` | `HealAmount` | `[UCL.Core.PA.UCL_FieldOnGUI]` |
| `m_HealTarget` | 治疗目标 | `RCG_SelectTargetData` | `HealTarget` | 取代旧的 `m_HealRange`（已淘汰） |

### A.3 重要 Method 摘要
*   **`AddAction(TriggerEffectData, AddActionMode)`**：
    1. `m_HealTarget.GetTargets(iData)` 解析目标。
    2. 非空时建立 `RCG_PlayerHealAction(m_HealType, User, targets, m_HealAmount.GetValue, iData)`。
    3. 以 `AddActionMode.InsertAction` 插入。
*   **`GetDescriptionFormat`**：
    *   `HealType.Amount` → i18n key `HealIcon_Des`，组合 `{Heal}` + `{Target}` + 心形图示。
    *   其他 → `m_HealType.GetLocalizeDes(...)` 拿对应格式串。
    *   `iTriggerEffectData.GetDescription` 套句尾修饰。
*   **`GetDescriptionShort`**：`Percentage` → `{Heal}%`；其他 → `{Heal}`。
*   **`Fusion(other)`**：必须对方也是 `RCG_HealSetting`；clone 自身并用 `IntVariable.FuseAdd` 合并 `m_HealAmount`。

### A.4 与其他系统的互动
*   **`RCG_PlayerHealAction`**：实际的 Action class；负责动画播放与生命值修改。
*   **`RCG_SelectTargetData`**：通用目标选择器，整个 BattleSetting 系统共用；`GetTargets(iData)` 是入口。
*   **`IntVariable`**：可序列化的数值容器，支援常数 / 变数 / 隐藏变数 三种型态。

### A.5 已知议题
*   旧的 `m_HealRange` 栏位与注解中的 `DeserializeFromJson` 处理已被遗弃；目前完全走 `m_HealTarget`。
*   旧版 `GetDescription` 注解保留作为对比参考（`//QWQ QWQ` 标记）。
