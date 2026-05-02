---
title: BattleTagCombine（标签组合）说明
description: 「组合效果」的特化版；以前缀战斗标签包覆主效果，并可指定触发时机
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# BattleTagCombine（标签组合）

> 程式类别名称：`RCG_BattleTagCombineSetting`

## 用途
「**组合效果**」的特化版。差别在于：**前面多绑了一组「战斗标签」作为前缀条件**，搭配指定的触发时机（`OnPlay` / `OnDraw` 等），让主效果只在「**特定触发时机 + 带有指定标签**」下生效。

实务上的用法：「若这张卡带有【消耗】标签，打出时触发 X 效果」「若带有【敏捷】标签，抽牌时触发 Y 效果」。

继承自：`RCG_CombineSetting`（所以也有「描述」「OverrideDescription」「组合效果」清单；见「组合效果」说明）

## 主要栏位（在「组合效果」之外多两个）

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **TriggerOn** | 是 | 触发时机：`OnPlay`（打出时，预设）/ `OnDraw`（抽牌时）/ ... 详见 `TriggerOn` enum。 |
| **PrefixBattleTagSetting** | 是 | 前缀的「战斗标签」设定 — 主效果只在这些标签被触发时跑。 |

加上「组合效果」基底的：**OverrideDescription** / **描述** / **组合效果（子设定清单）**。

## 行为说明
*   `CombineSettings` 会把 `PrefixBattleTagSetting` 放在子设定清单**最前面**作为「条件」，后面接原本的 `组合效果` 清单。
*   描述显示为「**若 {前缀标签}，则 {主效果}**」（i18n key `IfCardConditions2` + `ConditionFitThenDes`）。
*   触发时：先对 `PrefixBattleTagSetting` 的每个标签呼叫 `OnTriggerEffect(TriggerOn)`，**再执行**剩下的子设定。
*   `GetBattleTags()` **忽略** prefix 标签 — 避免标签被当作主效果的一部分聚合。

## 注意事项
*   **TriggerOn 必须与卡片实际触发点对齐**：若卡片本身在 `OnPlay` 触发，但这里 TriggerOn 设成 `OnDraw`，主效果永远不会跑。
*   **PrefixBattleTagSetting 为空**：等于「无前置条件」，描述会变成「若 ，则 …」 — 属于资料错误。
*   **与一般「条件判断」的差别**：「条件判断」是依 `RCG_Condition` 判断是否符合；本设定则是**让特定触发时机 + 标签**作为前置条件，语意较精准但用途较窄。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_BattleTagCombineSetting.cs`
*   **继承自**：`RCG_CombineSetting`
*   **无 i18n 类别名 key**：编辑器显示 stripped name `BattleTagCombine`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_TriggerOn` | `TriggerOn` | `TriggerOn` (enum) | — | 预设 `OnPlay` |
| `m_PrefixBattleTagSetting` | `PrefixBattleTagSetting` | `RCG_BattleTagSetting` | — | 嵌套的标签设定 |
| (继承) `m_OverrideDescription` / `m_Description` / `m_CombineSettings` | 同「组合效果」 | — | — | 见父类 |

### A.3 重要 Method 摘要
*   **`CombineSettings` (override)**：把 `m_PrefixBattleTagSetting` 插在最前 + base.CombineSettings。
*   **`GetBattleTags`** (override)：**跳过** prefix，只聚合 `m_CombineSettings.GetEnableBattleSettings()` 的标签。
*   **`AddAction`**：对 `i==0` 的元素（prefix）改呼叫各 BattleTag 的 `OnTriggerEffect(m_TriggerOn, iData)`；其余正常 `AddAction(InsertInOrder)`。
*   **`GetDescriptionFormat / GetDescription / GetDescriptionShort`**：以 `IfCardConditions2` + `ConditionFitThenDes` 包装 prefix 描述与主描述。

### A.4 与其他系统的互动
*   **`RCG_BattleTagSetting`**：作为 prefix 容器。
*   **`RCG_EffectTriggerOn.GetEffectTriggerOn(TriggerOn)`**：把 `TriggerOn` enum 转成可触发的物件。
*   **i18n keys**：`IfCardConditions2` / `ConditionFitThenDes`。

### A.5 已知议题
*   未本地化于 `AllTypes` 之外，子类选单显示为 stripped name。
