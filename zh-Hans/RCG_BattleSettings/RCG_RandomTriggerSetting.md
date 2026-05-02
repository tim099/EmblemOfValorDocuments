---
title: 随机触发 说明
description: 随机选取效果触发、依机率触发、或依变数机率触发三种模式
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 随机触发

> 程式类别名称：`RCG_RandomTriggerSetting`

## 用途
**让效果以随机方式发生**。三种模式：
*   **随机选 N 个**：从清单中随机挑 N 个效果触发
*   **整批依机率**：以一个固定机率决定整批效果是否触发
*   **整批依变数机率**：与上同，但机率由变数决定（例如「依当前 HP 比例触发」）

常见用途：赌博类卡、机率型 Buff、多选一随机效果。

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **RandomTriggerMode** | 是 | 三种模式（见下）。 |
| **RandomPickCount** | RandomPick 模式 | 随机挑几个效果触发。`Conditional` 控制显示。 |
| **TriggerRate** | TriggerByRate 模式 | 触发机率（0~100，slider 控制）。 |
| **VariableRate** | TriggerByVariableRate 模式 | 触发机率变数（其值代表 0~100 的百分比）。 |
| **TriggerSettings** | 是 | 候选效果清单；只有 `IsEnable = true` 的会被纳入。 |

### 模式详细
*   **RandomPick** — 从 enable 的候选中**随机挑 N 个**触发；同一效果不重复触发。
*   **TriggerByRate** — 掷一次 0~99 的随机数，< `TriggerRate` 则**整批**触发；否则全跳过。
*   **TriggerByVariableRate** — 同 TriggerByRate，但机率值由 `VariableRate.GetValue` 决定。

## 行为说明
*   描述会列出所有候选效果，配合机率资讯呈现（例如「50% 机率触发：[A], [B]」或「随机触发 1 个：[A] [B] [C]」）。

## 注意事项
*   **RandomPickCount > 候选数**：`RandomPick` 内部使用 `RandomPick(list, count)`；超过候选数的部分会被工具函式处理（通常取上限）。
*   **空清单**：合法但什么也不会发生。
*   **TriggerByRate 机率 0 / 100**：分别等于「永不触发」「永远触发」 — 属于极端情况，多半是设计错误。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_RandomTriggerSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_RandomTriggerSetting` → 「随机触发」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_RandomTriggerMode` | `RandomTriggerMode` | enum | — | 3 种模式 |
| `m_RandomPickCount` | `RandomPickCount` | `IntVariable` | — | `[Conditional(nameof(m_RandomTriggerMode), false, RandomPick)]` |
| `m_TriggerRate` | `TriggerRate` | `int` | — | `[UCL_IntSlider(0, 100)]` + `[Conditional(... TriggerByRate)]`，预设 50 |
| `m_VariableRate` | `VariableRate` | `IntVariable` | — | `[Conditional(... TriggerByVariableRate)]` |
| `m_TriggerSettings` | `TriggerSettings` | `List<RCG_BattleSetting>` | — | 过滤后 `TriggerSettings = m_TriggerSettings.Where(IsEnable)` |

### A.3 重要 Method 摘要
*   **`AddAction`**：依 `m_RandomTriggerMode`：
    *   `RandomPick` → `RCG_GameManager.Random.RandomPick(allEnabled, count)`，逐一 `AddAction(InsertInOrder)`。
    *   `TriggerByRate` → `Random.Range(0, 100) < m_TriggerRate` → 若触发，全 enable 子设定逐一 AddAction。
    *   `TriggerByVariableRate` → 同上但机率取 `m_VariableRate.GetValue(iData)`。
*   **`GetFusionCandidateSettings`** → 对所有 enable 子的候选聚合（不含自身）。
*   **`GetFusionBaseSetting`**：clone + 重建 `m_TriggerSettings` 为 placeholder 化版本。

### A.4 与其他系统的互动
*   **`RCG_GameManager.Random.RandomPick / Range`**：种子控制的随机来源（影响重现性）。
