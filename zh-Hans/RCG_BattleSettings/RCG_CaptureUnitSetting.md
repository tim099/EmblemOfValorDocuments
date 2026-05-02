---
title: 捕获单位 说明
description: 捕获目标单位（从战斗中移除），并将其转化为可召唤的物品加入背包
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 捕获单位

> 程式类别名称：`RCG_CaptureUnitSetting`

## 用途
**捕获**指定目标单位 — 把它从战场上移除，**并生成一个对应的「召唤道具」**加入玩家背包。最常见的用途是「精灵球式」捕获卡：战斗后可以放出该怪物协助。

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **目标** (`Target`) | 是 | 要捕获的单位选择器（通常选「敌方单体」）。 |
| **CaptureUnitItem** | 是 | 捕获后生成的物品模板；预设使用 `Vivarium_Captured`（兽笼）。 |

## 行为说明
*   执行时：
    1. 对目标播放捕获特效（`VFX_Summon`）+ 缩小动画 0.6 秒。
    2. 目标被「逃跑」掉（`Flee()` — 从战场移除，**不算击杀**）。
    3. 以 `CaptureUnitItem` 为模板 clone 出新物品，名称改为 `{物品名}[{被捕怪物名}]` 格式。
    4. 物品 ID 变为 `{原 ID}_{被捕怪物 ID}`，避免同名冲突。
    5. 物品的「使用效果」自动写入「召唤（被捕单位）」逻辑。
    6. 物品加入背包并弹出「获得物品」面板。
*   描述格式：「**捕获 {目标}**」（i18n key `CaptureUnitDes`）。

## 注意事项
*   **被捕单位不算击杀**：透过 `Flee()` 移除战场，所以「击杀后触发」「击杀数」等条件**不会记录**这次捕获。设计时请留意。
*   **多目标选择器**：技术上选择器若回传多个目标，**只会捕获第一个**（程式码 `aTargets[0]`），其他目标被忽略。请选单体目标。
*   **CaptureUnitItem 不可空**：`Vivarium_Captured` 是预设值；若改成不存在的 ID 会在生成物品时例外。
*   **物品命名冲突**：两次捕获同一种怪物会产生相同 ID 的物品，后者会覆盖前者 — 这是已知行为。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CaptureUnitSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_CaptureUnitSetting` → 「捕获单位」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_Target` | 目标 | `RCG_SelectTargetData` | `Target` | 捕获目标选择器 |
| `m_CaptureUnitItem` | `CaptureUnitItem` | `RCG_ItemGenData` | — | 预设 `RCG_ItemGenData("Vivarium_Captured")` |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：
    1. `m_Target.GetTargets(iData)[0]` → `aTarget`。
    2. `RCG_VFXManager.CreateVFX(CommonVFX.VFX_Summon)` 并设置位置。
    3. `aTarget.UnitAnimController.ScaleUnitLocal(...)` 缩小演出。
    4. `aTarget.Flee()` 从战场移除。
    5. clone `m_CaptureUnitItem.GetData()` 为 `RCG_ItemData`：改名（`LocalizeDic`）、改 ID（后缀 unitID）。
    6. 内嵌 `RCG_CommonEffect`（`OnPlay`）+ `RCG_SummonSetting`（`SummonType.Default`, `IsMonster=false`, `Unit=被捕怪物`）。
    7. `RCG_DataService.Ins.AddRuntimeData(aItemData, DataType.InGameRuntime)` → 注册为 runtime 资料。
    8. 建立 `RCG_Item(aItemData, RCG_Item.ItemType.Runtime)` 并 `AddItem()` 进背包。
    9. `RCG_AquireItemPanel` 显示获得结果。
*   **`GetDescriptionFormat`** → i18n key `CaptureUnitDes`，仅一个 `{Target}` 参数。

### A.4 与其他系统的互动
*   **`RCG_VFXManager` / `CommonVFX.VFX_Summon`**：捕获特效。
*   **`RCG_BattleUnit.Flee`**：把单位以「逃跑」方式移除（不算击杀）。
*   **`RCG_DataService` / `DataType.InGameRuntime`**：runtime 物品的注册处。
*   **`RCG_SummonSetting`**：被组装进物品的使用效果。
*   **`RCG_AquireItemPanel`**：UI 呈现。
