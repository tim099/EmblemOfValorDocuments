---
title: 资源变化 说明
description: 战斗中获得或消耗资源（金币、魂力等）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 资源变化

> 程式类别名称：`RCG_ResourceAlterSetting`

## 用途
**战斗中改变玩家持有的资源**（金币、魂力等）。常见用途：
*   「战斗中获得 50 金币」
*   「消耗 1 魂力触发强力卡牌」

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **ResourceAlterMode** | 是 | 变化模式：<br>• **Add** — 获得（预设）<br>• **Consume** — 消耗（**会卡可玩判定**） |
| **AcquiredResources** | 是 | 资源清单；每项是 `RCG_ResourceGenData`（资源类型 + 数量）。 |

## 行为说明
*   **可玩判定**：`Consume` 模式下，每项资源的当前值必须 ≥ 消耗量。任一不足整张卡不可打出。
*   **触发**：依模式对 `RCG_DataService.Ins` 的对应资源加 / 扣值。
*   描述：列出每项资源；`Consume` 套 `ConsumeItem` i18n、`Add` 套 `AcquireItem` i18n。

## 注意事项
*   **TODO 提醒**：资源变化时不会即时刷新卡牌的可玩状态 — 同档案有 `Todo:资源变化时 要刷新卡牌才能正确判断目前是否能打出` 标记。
*   **永远展开**：`[AlwaysExpendOnGUI]`，Inspector 中**永远不会折叠**。
*   **空清单**：合法，但等于无变化。
*   **与「能量变化」的差别**：能量是战斗内每回合刷新的资源；本设定是**跨战斗的全域资源**（金币、魂力）— 两者不要混淆。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ResourceAlterSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **`[System.Serializable] + [AlwaysExpendOnGUI]`** 标记
*   **i18n 类别名 key**：`RCG_ResourceAlterSetting` → 「资源变化」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_ResourceAlterMode` | `ResourceAlterMode` | enum (档内) | — | `Add` / `Consume` |
| `m_AcquiredResources` | `AcquiredResources` | `List<RCG_ResourceGenData>` | — | |

### A.3 重要 Method 摘要
*   **`CheckPlayable`**：`Consume` 模式逐一以 `RCG_DataService.Ins.GetResource(type)` 比对 `m_Amount.GetValue`，任一不足回 false；`Add` 模式恒为 true。
*   **`AddAction`**（档案 100 行外）：依模式对每项资源呼叫加 / 减 API。
*   **`Infos`**：base + 每项资源的类型资讯（`AddIfNotRepeat`）。
*   **`GetDescriptionFormat`** / **`GetDescriptionParams`**（档内）：用 `, ` 串接资源名，`Consume`/`Add` 模式套不同 i18n key。

### A.4 与其他系统的互动
*   **`RCG_DataService.Ins.GetResource(type)`**：当前资源值查询。
*   **`RCG_ResourceGenData / RCG_ResourceTypeGenData`**：资源模板。
