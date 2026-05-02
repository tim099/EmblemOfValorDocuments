---
title: Reinforcement
description: Summons reinforcement units (currently enemy-only); normal battles random-pick, elite/boss use template
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Reinforcement

> Class: `RCG_ReinforcementSetting`

## Purpose
**Summons enemy reinforcements** to the battlefield. Currently **monster-only** (cannot summon player allies). Examples:
*   Boss summons minions
*   Auto-spawn extras at certain turns

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **DefaultUnit** | Yes | Default reinforcement unit (**Elite / Boss battles only**). Normal battles ignore this and randomly pick from `MonsterSet`. |

## Behaviour
*   Reads `RCG_BattleField.Ins.MonsterSet` for available monsters of the current enemy type.
*   **Normal battle**: random pick from list.
*   **Elite / Boss battle**: uses `DefaultUnit` directly — prevents repeated elite stacking.
*   Prefers spawning at `iData.TargetPositions` empty slots; otherwise auto-locates an empty slot.
*   Description: "**Summon monster reinforcement**" (i18n `ReinforcementDes_Monster`).

## Notes
*   **Monster-only**: source has `bool aIsMonster = true` hardcoded; for player reinforcements use **Summon**.
*   **Silent failure on no slot**: `Debug.LogError` but no player feedback — card may seem ineffective.
*   **Elite/Boss limitation is intentional**: avoids "reinforcement card + Elite" chain-summoning a horde of elites. Source has `// TODO: should move to Condition` for future refactor.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ReinforcementSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_ReinforcementSetting` → "Reinforcement"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_DefaultUnit` | DefaultUnit | `RCG_UnitGenData` | — | Used for Elite / Boss battles |

### A.3 Key methods
*   **`LocalizeKey` (private)** → hardcoded `"ReinforcementDes_Monster"`.
*   **`AddAction`** (async):
    1. Random pick from `MonsterSet.GetUnitSpawnDatas(EnemyType)` → `aTargetUnit`.
    2. Override to `m_DefaultUnit.GetData()` if EnemyType ≠ Normal.
    3. Try `iData.TargetPositions` for placement; otherwise auto-locate (beyond shown range).

### A.4 Cross-system interactions
*   **`RCG_BattleField.Ins.MonsterSet.GetUnitSpawnDatas`**: candidate monster list.
*   **`RCG_BattleManager.Ins.EnemyType`** / **`RCG_EnemyTypeTagGenData.s_EnemyType_Normal`**: battle type check.
*   **`UCL_Random.Instance.Next`**: random pick source.

### A.5 Known issues
*   `// TODO: 目前只有普通戰鬥獲得增援 (應該移到Condition)` — should refactor to a condition rather than baking into reinforcement logic.
*   `bool aIsMonster = true` hardcoded; can't summon player units.
