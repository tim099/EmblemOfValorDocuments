---
title: Leveled Monster Action (RCG_MonsterLevelActionData)
description: Bundles different level versions of the same action — auto-switches to stronger versions as difficulty rises
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Leveled Monster Action

> Class name: `RCG_MonsterLevelActionData`

## Purpose

**Bundles "multiple level variants of the same action" together**. For example: "Attack_Close_x1" is `Lv0`, "Attack_Close_x2" is `Lv1`, "Attack_Close_AOE" is `Lv2`... When monster level rises, the system auto-switches to a stronger version by index. Alternatively, use **single-action mode** (`m_UseLevelAsIndex = false`): one Action that adjusts itself via internal HiddenVariable reading the level.

Inherits from `RCG_Asset<RCG_MonsterLevelActionData>`.

## Editor Layout

```
RCG_MonsterLevelActionData: <ID>
    UseLevelAsIndex (bool)
    MaxSkillLevel
    SkillLevelOffest
    ▼ Actions (UseLevelAsIndex = true)   ← index mode: list per-level Actions
    ▼ SingleAction (UseLevelAsIndex = false) ← single mode: one Action only
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **UseLevelAsIndex** | yes | `true`: use `Actions[level]`; `false`: always use `SingleAction`, with internal level handling |
| **MaxSkillLevel** | yes | Level cap; clamped on overflow (default 100) |
| **SkillLevelOffest** | — | Level base offset (only effective when > 0); used to **uniformly raise** the effective level |
| **SingleAction** | when UseLevelAsIndex=false | The single Action (uses `m_SkillLevel` variable to adjust strength) |
| **Actions** | when UseLevelAsIndex=true | Per-level Action list; `Actions[i]` corresponds to level i |

## Behavior

### `GetAction(level)`
1. Add **global difficulty skill level bonus**: `level += DifficultyData.m_EnemySkillLevel`.
2. If `SkillLevelOffest > 0`, add the offset.
3. Clamp to `[0, MaxSkillLevel]`.
4. **Index mode** (`UseLevelAsIndex = true`):
    *   `Actions` empty → return `Idle`.
    *   Else `result = Actions[clamp(level, 0, Actions.Count - 1)]`.
5. **Single mode**: `result = SingleAction`.
6. Write `level` into `result.m_Action.m_SkillLevel`, return.

### Preview
*   Index mode: lists all Actions (each individually expandable).
*   Single mode: one row showing SingleAction.

## Caveats

*   **`Actions[i]` corresponds to level i**: list index is the level; levels above the list size clamp to the last entry.
*   **When `MaxSkillLevel = 100` exceeds `Actions.Count`**: levels beyond `Actions.Count - 1` all use the last Action (strength capped).
*   **`SkillLevelOffest` only effective when > 0**: negatives ignored; lowering strength must go through global difficulty settings.
*   **Single mode's `m_SkillLevel`**: written into the returned clone at runtime; the original Asset is unaffected.
*   **Single mode's HiddenVariable** is the design extension point: binding `SkillLevel` to a variable inside the Action enables "linear scaling without 10 separate Actions".

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_MonsterLevelActionData.cs`
*   **Inherits**: `RCG_Asset<RCG_MonsterLevelActionData>`
*   **AssetGroup**: `EditBattleSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_UseLevelAsIndex` | UseLevelAsIndex | `bool` | Default `true` |
| `m_MaxSkillLevel` | MaxSkillLevel | `int` | Default 100 |
| `m_SkillLevelOffest` | SkillLevelOffest | `int` | Default 0; only added when > 0 |
| `m_SingleAction` | SingleAction | `RCG_MonsterActionGenData` | `Conditional(UseLevelAsIndex == false)` |
| `m_Actions` | Actions | `List<RCG_MonsterActionGenData>` | `Conditional(UseLevelAsIndex == true)` |

### A.3 Key Methods

*   **`GetAction(int level)`** — main entry; includes difficulty bonus / offset / clamp / level write-back to Action.
*   **`Preview`** — lists all Actions (index mode) or one Action (single mode).

### A.4 System Interactions

*   **`RCG_MonsterActionData`** — Action templates this drops.
*   **`RCG_MonsterActionGenData`** — Asset Entry; default `IdleID = "Idle"`.
*   **`RCG_MonsterLevelActionGenData`** (in-file) — type used externally to reference this data, with `m_Level` (`IntVariable`); `GetAction()` uses this level to query the data.
*   **`RCG_DataService.Ins.m_DifficultyData.m_EnemySkillLevel`** — global skill level bonus.

### A.5 Known Issues

*   `RCG_MonsterLevelActionGenData` LogError + falls back to `Idle` on construction failure; can hide bugs.
