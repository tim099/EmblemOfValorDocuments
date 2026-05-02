---
title: 玩家能量变化 说明
description: 改变玩家当前能量（费用）— 加 / 减 / 设定为指定值
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 玩家能量变化

> 程式类别名称：`RCG_CostAlterSetting`

## 用途
**改变玩家当前的能量值（费用 / Cost）**。常见用途：
*   「补 1 能量」
*   「消耗 2 能量触发强力效果」
*   「将能量设为 0」（过载）

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **CostAlter** | 是 | 变化量（变数可变）。 |
| **CostAlterType** | 是 | 变化模式：<br>• **Add** — 增加能量<br>• **Sub** — 减少能量（**会卡可玩判定**）<br>• **Set** — 设为指定值 |

## 行为说明
*   **可玩判定**：`Sub` 模式 + 纯数值时，玩家当前能量必须 ≥ `CostAlter` 才能打出（避免出现负能量）。其他模式不检查。
*   **触发**：依模式呼叫 `RCG_Player.Ins.AlterCost(...)` 或 `SetCost(...)`。
*   **VFX**：增加会播 `ChargeEffect`，减少会播 `OverloadEffect`，并在角色身上跳出能量变化数字。
*   描述格式依模式套不同 i18n key（`AddCostIcon_Des` / `SubCostIcon_Des` / `SetCostIcon_Des`）+ 能量图示。

### 卡片融合
仅同 `CostAlterType` 之间可融合（变化量加总）。

## 注意事项
*   **旧资料反序列化**：以前用「`CostAlter` 为负值」表示减少；现在用 `CostAlterType.Sub`。`DeserializeFromJson` 会自动把旧资料的负值转换为 `Sub` + 正值。
*   **变数型攻击力的 Sub 不检查**：可玩判定只在 `m_VariableType == Value`（纯数值）时生效；变数型减能量**不会挡打出**，可能让能量短暂变负。
*   **Set 不受可玩判定限制**：永远可打出，请小心避免「Set 0」型卡导致玩家失去战术空间。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CostAlterSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_CostAlterSetting` → 「玩家能量变化」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_CostAlter` | `CostAlter` | `IntVariable` | — | |
| `m_CostAlterType` | `CostAlterType` | enum (档内) | — | `Add` / `Sub` / `Set` |

### A.3 重要 Method 摘要
*   **`CheckPlayable`**：`Sub` + 纯数值时 `RCG_Player.Ins.Cost >= m_CostAlter`；其他模式 true。
*   **`DeserializeFromJson`**：旧资料负值 → 自动转为 `Sub` + 正值（内部翻转 `m_Value` 或 `m_Mult`）。
*   **`AddAction`** (async)：依模式呼叫 `RCG_Player.Ins.AlterCost(aValue, isCardCost: false)` 或 `SetCost(aValue)`，并播放对应 VFX (`ChargeEffect` / `OverloadEffect` + `VFX_Cost`)。
*   **`Fusion`**：要求同 `m_CostAlterType`；clone + `IntVariable.FuseAdd`。

### A.4 与其他系统的互动
*   **`RCG_Player.Ins.AlterCost / SetCost`**：能量修改入口。
*   **`RCG_CommonVFXGenData.s_ChargeEffect / s_OverloadEffect`**：VFX 来源。
*   **`CommonVFX.VFX_Cost`** (透过 `aCostVFX.SetAlterHP`)：能量变化数字浮动 UI。
