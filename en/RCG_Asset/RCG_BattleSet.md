---
title: Battle Set (RCG_BattleSet)
description: A specific battle's monster configuration + tags + enemy type — used to spawn the battle when a map node triggers
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Battle Set

> Class name: `RCG_BattleSet`

## Purpose

**A specific battle's monster configuration**. Examples: "3 Goblins + 1 Goblin Chef", "Boss Whale solo", "Elite encounter: Dragon + Mage" are different BattleSets. When a map node triggers a battle, an `RCG_BattleSet` is drawn from `RCG_BattleSetDropPool` to spawn the battle.

Inherits from `RCG_Asset<RCG_BattleSet>`.

## Editor Layout

```
RCG_BattleSet: <ID>
    EnemyType  ← enemy type (Normal / Elite / Boss)
    Tags       ← BattleSet tags (used by DropPool filtering)
    MonsterSet ← monster configuration (positions, IDs, reward resources)
    Preview    ← live preview of the configuration
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **MonsterSet** | yes | Full monster configuration (`RCG_MonsterSet`: which monster goes in each slot, what reward resources drop) |
| **Tags** | no | BattleSet tags (chapter / scene type, etc.) — used by DropPool filtering |
| **EnemyType** | yes | Enemy type (`Normal` / `Elite` / `Boss`); affects rewards, BGM, UI title |

## Behavior

### Default Fetch
`RCG_BattleSet.DefaultBattleSet` directly takes the data for `RCG_BattleSetGenData.DefaultID`; fallback when missing.

### Preview
Shows ID + EnemyType, tag list, and `MonsterSet.Preview` monster position visualization.

## Caveats

*   **EnemyType is a Tag object**, not an enum: was originally an enum (`m_EnemyType = EnemyType.Normal`); now uses `RCG_EnemyTypeTagGenData` with the legacy field commented out. To add types, edit Tag assets directly.
*   **Reward resources inside MonsterSet**: there used to be a `m_MonsterSet.m_RewardResources.Clear()` cleanup on deserialize (now commented out); reference for special cleanup logic.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSet.cs`
*   **Inherits**: `RCG_Asset<RCG_BattleSet>`
*   **AssetGroup**: `EditBattleSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_MonsterSet` | MonsterSet | `RCG_MonsterSet` | Main configuration container |
| `m_Tags` | Tags | `List<RCG_BattleSetTagGenData>` | DropPool filtering |
| `m_EnemyType` | EnemyType | `RCG_EnemyTypeTagGenData` | Replaces legacy enum |

### A.3 Key Methods

*   **`Preview` / `OnGUI`** — editor rendering.
*   **`DefaultBattleSet`** (static) — `Util.GetData(RCG_BattleSetGenData.DefaultID)`.

### A.4 System Interactions

*   **`RCG_MonsterSet`** — detailed container for positions + rewards.
*   **`RCG_BattleSetDropPool`** — drop pool for battle setups.
*   **`RCG_BattleSetGenData`** — Asset Entry wrapper.

### A.5 Known Issues

*   Legacy `m_EnemyType` (enum) commented out, marking the migration to the Tag type.
