---
title: 设定目标 说明
description: 自动依规则重新选取目标（不弹 UI）；会覆盖原本的目标清单
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 设定目标

> 程式类别名称：`RCG_SetTargetSetting`

## 用途
**自动依规则重新选定目标**（不开 UI）。常见用途：
*   「将攻击转移到血量最低的敌人」
*   「重定向到后排目标」
*   AI 触发效果时自动指定目标

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **UseOldSelectUnitRule** | — | 预设打勾（向前兼容）。打勾使用旧版 `RCG_SelectTargetData`；不打勾用新版 `RCG_TargetSelectRule`。 |
| **SelectTarget** | UseOldSelectUnitRule=true 时 | 旧版目标规则（`RCG_SelectTargetData`）。 |
| **SelectUnitRule** | UseOldSelectUnitRule=false 时 | 新版目标规则（`RCG_TargetSelectRule`）。 |

## 行为说明
*   依设定的规则自动解析目标清单，覆盖 `iData.Targets`。
*   不开 UI、不需玩家输入；适合 AI 与自动化效果。
*   描述显示对应规则的描述（短名）。

## 注意事项
*   **与「选取目标」的差别**：本设定**不出 UI**；「选取目标」要玩家点击。
*   **新规则 vs 旧规则**：新版 `RCG_TargetSelectRule` 表达能力更强；旧版 `RCG_SelectTargetData` 仅为兼容保留。新资料请用新版（取消打勾 `UseOldSelectUnitRule`）。
*   **同附录中 TODO**：程式内有 `// TODO: Test QWQ2`，新版规则实际使用尚需验证。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_SetTargetSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_SetTargetSetting` → 「设定目标」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_UseOldSelectUnitRule` | `UseOldSelectUnitRule` | `bool` | — | 预设 `true`（向前兼容） |
| `m_SelectUnitRule` | `SelectUnitRule` | `RCG_TargetSelectRule` | — | `[Conditional("m_UseOldSelectUnitRule", false)]` |
| `m_SelectTarget` | `SelectTarget` | `RCG_SelectTargetData` | — | `[Conditional("m_UseOldSelectUnitRule", true)]` |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：依 `m_UseOldSelectUnitRule` 走 `m_SelectUnitRule.SelectTargets(iData)` 或 `m_SelectTarget.GetTargets(iData)`，赋值到 `iData.Targets`。
*   **`GetDescription`**：代理到对应规则的 `Description / GetShortName`。

### A.4 与其他系统的互动
*   **`RCG_TargetSelectRule`**：新版目标规则容器。
*   **`RCG_SelectTargetData`**：旧版目标选择器；广泛被其他 setting 使用。

### A.5 已知议题
*   `// TODO: Test QWQ2` — 新版规则尚需测试验证。
