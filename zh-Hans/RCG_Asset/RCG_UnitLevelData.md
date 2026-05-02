---
title: 单位等级资料 (RCG_UnitLevelData) 说明
description: 定义「单位的 HP / 攻击力如何随等级成长」的曲线；难度系统与此搭配影响怪物强度
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 单位等级资料

> 程式类别名称：`RCG_UnitLevelData`

## 用途

定义「**单位的 HP 与攻击力如何随等级成长**」的曲线。每个怪物 / 角色可以引用一份 `RCG_UnitLevelData`；游戏中根据**当前难度**换算等级，再透过此曲线决定该等级下的 HP 倍率与攻击力倍率。

继承自 `RCG_Asset<RCG_UnitLevelData>`。

## 编辑器中的样貌

```
RCG_UnitLevelData: <ID>
    LevelCurve  ← 等级曲线：难度 0~1 → 等级成长值
    HPCurve     ← HP 曲线：等级 0~1 → HP 倍率
    AtkCurve    ← 攻击曲线：等级 0~1 → 攻击倍率
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **LevelCurve** | 是 | 等级曲线。输入 `aDifficultyVal`(0~1)，回传的是要加到 `BaseLevel` 上的等级值 |
| **HPCurve** | 是 | HP 倍率曲线。输入等级值 (0~1)，回传 HP 倍率 |
| **AtkCurve** | 是 | 攻击倍率曲线。同上 |

每条 `CurveData` 内部：
*   **CurveType**：`Linear`（线性 `Lerp(min, max, x)`）或 `Power`（次方 `pow(base, x * segment)`）
*   **MinVal / MaxVal**（Linear）：起始值与最大值
*   **PowerBase / PowerSegment**（Power）：次方底数与分段数

## 行为说明

### 等级换算 (`GetLevel(BaseLevel)`)
1. 取**当前难度**：`RCG_BigMapManager.Difficulty + RCG_DataService.Ins.Difficulty`。
2. 把难度映射到 0~1 范围（**分段函式**）：
    *   0~10 → 0.00~0.10（线性）
    *   10~50 → 0.10~0.30
    *   50~100 → 0.30~0.40
    *   100+ → 0.40~1.00
3. `LevelCurve.GetValue(0~1) → 加成`，`Level = BaseLevel + round(加成)`，clamp 在 [1, 100]。

### 属性计算
*   `GetMaxHP(baseHP, level)` = `round(baseHP × HPCurve(level/99) × DifficultyData.HealthMult)`
*   `GetAtkMult(level)` = `AtkCurve(level/99) × DifficultyData.AtkMult`
注意「全局难度倍率」(`DifficultyData.HealthMult / AtkMult`) 会在这层再乘一次。

### 预览
编辑器内预设 Preview 只显示 ID 标签，不绘制曲线视觉化（要看曲线形状只能去看数值）。

## 注意事项

*   **MaxLevel = 100**：上限写死，超过会被 clamp。修改要连动 `m_PowerSegment` 预设值。
*   **MaxDifficulty = 100**（理论值）：实际 100+ 的难度会继续按 0.001 线性成长，但 `aDifficulty > 1` 时会把 `aDifficulty` 设为 1（**这看起来是 bug**：应该设 `aDifficultyVal`）。
*   **`Debug.LogWarning` 在 `GetMaxHP` / `GetAtkMult` 内**：每次计算都会印 log，会洗版；release 前要拿掉。
*   **全局难度倍率**（`DifficultyData.HealthMult / AtkMult`）会在这层再乘一次；总体强度 = 等级曲线 × 全局倍率。
*   **`m_LevelCurve` 预设 `Linear(0, 100)`** 表示「满难度时加 100 级」，搭配 `BaseLevel = 1` 就会抵到 MaxLevel。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_UnitLevelData.cs`
*   **继承自**：`RCG_Asset<RCG_UnitLevelData>`
*   **AssetGroup**：`EditBattleSetting`
*   **常数**：`MinLevel = 1` / `MaxLevel = 100` / `MaxDifficulty = 100`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_LevelCurve` | LevelCurve | `CurveData` | 预设 `Linear(0, 100)` |
| `m_HPCurve` | HPCurve | `CurveData` | 预设 `Linear(1, 10)` |
| `m_AtkCurve` | AtkCurve | `CurveData` | 预设 `Linear(1, 4)` |

### A.3 重要 Method 摘要

*   **`Difficulty` (static property)** — `BigMapManager.Difficulty + DataService.Difficulty`。
*   **`GetLevel(BaseLevel)`** — 主入口；内含分段难度→0~1 的映射。
*   **`GetMaxHP(baseHP, level)` / `GetAtkMult(level)`** — 套曲线 + 全局难度倍率。
*   **`GetLevelValue(level)`** (private) — `(level - 1) / (MaxLevel - 1f)`，把 1~100 对应到 0~1。

### A.4 与其他系统的互动

*   **`RCG_BigMapManager.Difficulty`** — 大地图层级的难度。
*   **`RCG_DataService.Ins.Difficulty`** — 玩法 / 挑战 runtime 难度。
*   **`RCG_DataService.Ins.m_DifficultyData.m_HealthMult / m_AtkMult`** — 全局难度倍率。
*   **`RCG_UnitLevelGenData`**（档内）— Asset Entry 包装，预设 ID = `"Default"`。

### A.5 已知议题

*   `GetLevel` 内 `aDifficultyVal > 1` 时改的是 `aDifficulty = 1`（型别不对，疑似 bug；应改 `aDifficultyVal = 1`）。
*   `GetMaxHP` / `GetAtkMult` 内 `Debug.LogWarning` 会洗版。
