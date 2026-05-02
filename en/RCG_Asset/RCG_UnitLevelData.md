---
title: Unit Level Data (RCG_UnitLevelData)
description: Curves defining "how a unit's HP / attack scale with level" — combined with the difficulty system to control monster strength
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Unit Level Data

> Class name: `RCG_UnitLevelData`

## Purpose

Defines **curves for how a unit's HP and attack scale with level**. Each monster / character can reference an `RCG_UnitLevelData`; in-game, the level is computed from **current difficulty**, then this curve determines HP and attack multipliers at that level.

Inherits from `RCG_Asset<RCG_UnitLevelData>`.

## Editor Layout

```
RCG_UnitLevelData: <ID>
    LevelCurve  ← level curve: difficulty 0~1 → level growth
    HPCurve     ← HP curve: level 0~1 → HP multiplier
    AtkCurve    ← attack curve: level 0~1 → attack multiplier
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **LevelCurve** | yes | Level curve. Input `aDifficultyVal` (0~1), output is the level growth value added to `BaseLevel` |
| **HPCurve** | yes | HP multiplier curve. Input level (0~1), output HP multiplier |
| **AtkCurve** | yes | Attack multiplier curve, same as above |

Each `CurveData` contains:
*   **CurveType**: `Linear` (`Lerp(min, max, x)`) or `Power` (`pow(base, x * segment)`)
*   **MinVal / MaxVal** (Linear): start and max values
*   **PowerBase / PowerSegment** (Power): power base and segment count

## Behavior

### Level Computation (`GetLevel(BaseLevel)`)
1. Get **current difficulty**: `RCG_BigMapManager.Difficulty + RCG_DataService.Ins.Difficulty`.
2. Map difficulty to 0~1 (**piecewise linear**):
    *   0~10 → 0.00~0.10 (linear)
    *   10~50 → 0.10~0.30
    *   50~100 → 0.30~0.40
    *   100+ → 0.40~1.00
3. `LevelCurve.GetValue(0~1) → growth`, `Level = BaseLevel + round(growth)`, clamped to [1, 100].

### Stat Computation
*   `GetMaxHP(baseHP, level)` = `round(baseHP × HPCurve(level/99) × DifficultyData.HealthMult)`
*   `GetAtkMult(level)` = `AtkCurve(level/99) × DifficultyData.AtkMult`
Note: the global difficulty multipliers (`DifficultyData.HealthMult / AtkMult`) are applied here on top of the curve.

### Preview
The default editor preview only shows the ID; curves are not visualized, so checking shape requires reading the values.

## Caveats

*   **MaxLevel = 100**: hardcoded cap; raise it requires updating `m_PowerSegment` defaults.
*   **MaxDifficulty = 100** (theoretical): in practice, difficulty over 100 keeps growing linearly at 0.001/step, but `aDifficulty > 1` is set to `aDifficulty = 1` (**looks like a bug**: should set `aDifficultyVal`).
*   **`Debug.LogWarning` inside `GetMaxHP` / `GetAtkMult`**: prints on every call — log spam, must be removed before release.
*   **Global difficulty multiplier** is applied at this layer; total strength = level curve × global multiplier.
*   **Default `m_LevelCurve = Linear(0, 100)`** means "+100 levels at max difficulty" — combined with `BaseLevel = 1` it caps at MaxLevel.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_UnitLevelData.cs`
*   **Inherits**: `RCG_Asset<RCG_UnitLevelData>`
*   **AssetGroup**: `EditBattleSetting`
*   **Constants**: `MinLevel = 1` / `MaxLevel = 100` / `MaxDifficulty = 100`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_LevelCurve` | LevelCurve | `CurveData` | Default `Linear(0, 100)` |
| `m_HPCurve` | HPCurve | `CurveData` | Default `Linear(1, 10)` |
| `m_AtkCurve` | AtkCurve | `CurveData` | Default `Linear(1, 4)` |

### A.3 Key Methods

*   **`Difficulty` (static property)** — `BigMapManager.Difficulty + DataService.Difficulty`.
*   **`GetLevel(BaseLevel)`** — main entry; piecewise difficulty→0~1 mapping inside.
*   **`GetMaxHP(baseHP, level)` / `GetAtkMult(level)`** — apply curve + global difficulty multiplier.
*   **`GetLevelValue(level)`** (private) — `(level - 1) / (MaxLevel - 1f)`, mapping 1~100 to 0~1.

### A.4 System Interactions

*   **`RCG_BigMapManager.Difficulty`** — bigmap-layer difficulty.
*   **`RCG_DataService.Ins.Difficulty`** — runtime difficulty (gameplay / challenge).
*   **`RCG_DataService.Ins.m_DifficultyData.m_HealthMult / m_AtkMult`** — global difficulty multipliers.
*   **`RCG_UnitLevelGenData`** (in-file) — Asset Entry; default ID = `"Default"`.

### A.5 Known Issues

*   In `GetLevel`, when `aDifficultyVal > 1`, it sets `aDifficulty = 1` (wrong type; suspected bug — should set `aDifficultyVal = 1`).
*   `Debug.LogWarning` in `GetMaxHP` / `GetAtkMult` is log spam.
