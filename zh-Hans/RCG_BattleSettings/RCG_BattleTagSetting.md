---
title: 战斗标签 说明
description: 为效果附加战斗标签（如「易碎」「敏捷」），不直接产生动作而是改变上层设定的描述与 Tag 聚合
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 战斗标签

> 程式类别名称：`RCG_BattleTagSetting`

## 用途
**附加战斗标签**到效果上 — 自身**不产生任何战斗动作**，纯粹做为「**标签载体**」。常用于：
*   让组合效果获得「**易碎**」「**敏捷**」「**消耗**」等属性标签
*   作为「卡片资讯」面板上的词条来源

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **BattleTags** | 是 | 战斗标签清单，每项是 `RCG_BattleTagGenData`（标签模板）。可含多个。 |

## 行为说明
*   **不产生 Action**：这个设定的 `AddAction` 是空实作 — 它只用来宣告标签。
*   **卡片资讯**：每个标签会在卡片悬停 Tooltip 上显示自己的名称与描述；可堆叠的标签（如 `m_IsStackable = true`）描述中会带层数。
*   **短描述**：直接串接所有标签的缩写名（没有额外修饰）。
*   **`GetBattleTags()`**：上层的「组合效果」「回圈」等容器会透过此方法**聚合所有子设定的标签**统一显示。

## 注意事项
*   **要产生实际效果请放在「组合效果」或「状态」**：战斗标签本身只是 metadata；想要「易碎」带来的减伤效果，得另外设定触发逻辑。
*   **空清单**：合法但无意义；会占资料空间又不影响任何行为。
*   **永远展开**：此设定有 `[AlwaysExpendOnGUI]` 属性，Inspector 中**永远不会折叠** — 直接展示标签清单。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_BattleTagSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **`[System.Serializable]` + `[UCL.Core.ATTR.AlwaysExpendOnGUI]`** 标记
*   **i18n 类别名 key**：`RCG_BattleTagSetting` → 「战斗标签」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_BattleTags` | `BattleTags` | `List<RCG_BattleTagGenData>` | — | |

### A.3 重要 Method 摘要
*   **`AddAction`** → 空实作（仅 `// base.AddAction(...)` 被注解掉）。
*   **`GetBattleTags()`** → `m_BattleTags.Clone()`；上层容器透过此方法聚合。
*   **`Infos`**：每个 tag 组出 `{LocalizedName}\n{Description}`，可堆叠类型加上 `StackedBattleTag` 模板（含 `eBattleVariable_BattleTagStackCount` 变数）。
*   **`GetShortName`** → 串接 `m_BattleTags[i].GetShortName()`；空清单回退到 base。
*   **`GetDescription`** → 永远回传 `string.Empty`（被刻意覆写）。

### A.4 与其他系统的互动
*   **`RCG_BattleTagGenData / RCG_BattleTag`**：标签模板与实例。
*   **i18n keys**：`StackedBattleTag` / `eBattleVariable_BattleTagStackCount` / tag 颜色 `RCG_BattleTag`。
*   **被谁呼叫 `GetBattleTags`**：所有容器类设定（CombineSetting / LoopSetting / ConditionalSetting / ForeachTargetSetting 等）。
