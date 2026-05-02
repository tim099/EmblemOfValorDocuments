---
title: 状态 说明
description: 给予 / 减少 / 移除 / 转换 / 吸收 / 随机抽取状态的全功能设定
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 状态

> 程式类别名称：`RCG_StatusSetting`

## 用途
**游戏中的「状态」核心设定**。给予层数、减少层数、移除整个状态、按类型批次操作、A 转 B 等多种模式。是除了攻击以外**最常用的设定之一**。

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **目标** (`Target`) | 是 | 状态套用的目标选择器。 |
| **StatusModifiedType** | 是 | 变动类型，**10 种模式**（见下表）。 |
| **状态** (`Status`) | Add/Decrease/Remove/Convert/Set 模式必填 | 要操作的状态类型（`RCG_CustomStatusGenData`）。 |
| **StatusDropPool** | StatusDropPool 模式必填 | 从哪个状态池随机抽。 |
| **ConvertStatus** | Convert 模式必填 | 要转换成的目标状态。 |
| **ConvertLayer** | Convert 模式 | 多少层触发一次转换（预设 10）。 |
| **ConvertRatio** | Convert 模式 | 转换比例（预设 1）。例如 ConvertLayer=10 + ConvertRatio=3 → 「每 10 层恍惚 → 3 层晕眩」。 |
| **StatusTags** | RemoveStatusOfTargetType/AbsorbStatusOfTargetType 模式 | 要操作的状态类型清单（如「全部 Debuff」）。 |
| **值** (`Amount`) | Add/Decrease/Set 模式 | 变动层数（变数可变）。 |
| **DescriptionType** | — | 描述句型：`Default` / `Then`。 |
| **详细设定** (`DetailSetting`) | — | 含 `SkipAnim`（跳过演出，直接设定层数）。 |

### StatusModifiedType 模式
| 值 | 行为 |
|---|---|
| **AddStatusLayer** | 新增层数（预设） |
| **DecreaseStatusLayer** | 减少层数 |
| **RemoveStatus** | 移除指定状态（不论层数） |
| **RemoveAllStatus** | 移除全部状态 |
| **RemoveAllDebuff** | 移除全部负面状态 |
| **RemoveStatusOfTargetType** | 依 StatusTags 清单批次移除 |
| **AbsorbStatusOfTargetType** | 把目标的某类状态吸到自身 |
| **ConvertStatus** | A 状态转换为 B 状态（依 Layer / Ratio） |
| **SetStatusLayer** | 设定层数为指定值 |
| **StatusDropPool** | 从状态池随机抽一个状态给予 |

## 行为说明
*   依模式对 `m_Target.GetTargets(iData)` 套用。
*   有 `Amount` 的模式支援变数绑定（如「依手牌数加敏捷」）。
*   `Convert` 模式：每 `ConvertLayer` 层消耗，产生 `ConvertRatio` 层的 ConvertStatus。
*   描述格式因模式而异 — UI 显示 i18n 名称 + 图示。

### 卡片融合
仅 `AddStatusLayer / DecreaseStatusLayer / SetStatusLayer` 三个模式可融合（Amount 加总）。其他模式不可融合。

## 注意事项
*   **AbsorbStatusOfTargetType**：吸到「**自身**」（不是目标选择器）— 注意是固定行为。
*   **状态池随机**：`StatusDropPool` 受战斗种子影响；同种子下重玩会抽到一样的状态。
*   **SkipAnim**：跳过演出可加快结算；但玩家可能来不及看到状态变化。重要状态建议不打勾。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_StatusSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **`[System.Serializable]`** 标记
*   **i18n 类别名 key**：`RCG_StatusSetting` → 「状态」
*   **同档内巢状类**：`DetailSetting`（含 `m_SkipAnim`）/ `DescriptionType` enum (`Default`, `Then`)
*   **同档顶层 enum**：`StatusModifiedType` (10 个值)

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_Target` | 目标 | `RCG_SelectTargetData` | `Target` | |
| `m_StatusModifiedType` | `StatusModifiedType` | `StatusModifiedType` | — | 预设 `AddStatusLayer` |
| `m_Status` | 状态 | `RCG_CustomStatusGenData` | `Status` | `[Conditional(...5 种模式)]` |
| `m_StatusDropPool` | `StatusDropPool` | `RCG_StatusDropPoolGenData` | — | `[Conditional(... StatusDropPool)]` |
| `m_ConvertStatus` | `ConvertStatus` | `RCG_CustomStatusGenData` | — | `[Conditional(... ConvertStatus)]` |
| `m_ConvertLayer` | `ConvertLayer` | `IntVariable` | — | `[Conditional(... ConvertStatus)]`，预设 10 |
| `m_ConvertRatio` | `ConvertRatio` | `IntVariable` | — | `[Conditional(... ConvertStatus)]`，预设 1 |
| `m_StatusTags` | `StatusTags` | `List<RCG_StatusTagGenData>` | — | `[Conditional(... 两个 OfTargetType 模式)]` |
| `m_Amount` | 值 | `IntVariable` | `Amount` | `[UCL_FieldOnGUI]` + `[Conditional(... 3 种需要 Amount 的模式)]` |
| `m_DescriptionType` | `DescriptionType` | `DescriptionType` (档内 enum) | `DescriptionType` | |
| `m_DetailSetting` | 详细设定 | `DetailSetting` (档内巢状类) | `DetailSetting` | 含 `m_SkipAnim` |

### A.3 重要 Method 摘要
*   **`Fusion`**：要求 `m_StatusModifiedType` 同为三种 Layer 操作之一；clone + `IntVariable.FuseAdd(m_Amount)`。
*   **`GetShortName`**：依 StatusModifiedType 走分支：直接 i18n / `GetDescription()` / `m_StatusDropPool.GetData().GetShortName()`。
*   **`AddAction`** / **`Infos`**（档案 200 行外）：依模式套用实际逻辑与资讯聚合。

### A.4 与其他系统的互动
*   **`RCG_UnitStatus`**：实际的状态容器（`GetStatusLayer` / `m_Status` 字典等）。
*   **`RCG_StatusTagGenData.StatusDebuff / StatusDot`**：类型标记，用于批次移除。
*   **`CreateAction.StatusAction`**：实际的 Action 建构工具（被 `RCG_ConsumeStatusSetting` 等共用）。

### A.5 已知议题
*   `CanEnhence / Enhence` 已注解（强化系统重构中）。
