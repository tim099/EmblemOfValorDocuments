---
title: Battle Start Action (RCG_BattleStartActionData)
description: Special behaviors triggered at battle start — ambush damage, stun, initial buffs
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Battle Start Action

> Class name: `RCG_BattleStartActionData`

## Purpose

**Special behaviors triggered at the very start of a battle**. Examples: "Ambush" can deal damage to enemies before they act; "Prepared" gives the party initial buffs; "Magic Circle" can release an AOE on turn 0. Each battle can reference one `RCG_BattleStartActionData`, which contains a `User` (who executes) + a sequence of `RCG_BattleSetting` (what to do).

Inherits from `RCG_Asset<RCG_BattleStartActionData>`.

## Editor Layout

```
RCG_BattleStartActionData: <ID>
    User                  ← executor selection rule
    BattleStartActions    ← sequence of BattleSettings to run
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **User** | yes | Executor selection rule (`RCG_SelectTargetData`). Default selects Ally / All range; if empty → falls back to player's first unit |
| **BattleStartActions** | yes | Sequence of `RCG_BattleSetting` to run on battle start (damage, buff, draw, etc.) |

## Behavior

### `AddAction(TriggerEffectData)`
1. Use `m_User` to get the target list.
2. Take the first target as `iData.User` (if none → player's first unit).
3. Add all `m_BattleStartActions` to the action queue in order (`PushBack` mode).

### Default ID
`RCG_BattleStartActionGenData.DefaultID = "None"`: signifies "no battle start action".

## Caveats

*   **`User` is the "executor", not the "target"**: each BattleSetting has its own target selection internally; `m_User` only decides **whose perspective the action originates from**.
*   **Empty target list fallback** to `RCG_BattleManager.PlayerBattleUnits[0]`: ensures the start action doesn't fail when User rule misses.
*   **`OnGUI` is fully commented out**: currently uses base class default rendering for editing.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_BattleStartActionData.cs`
*   **Inherits**: `RCG_Asset<RCG_BattleStartActionData>`
*   **AssetGroup**: `EditBattleSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_User` | User | `RCG_SelectTargetData` | Default `SelectTargetType.Range`, range `Ally + All` |
| `m_BattleStartActions` | BattleStartActions | `List<RCG_BattleSetting>` | Start action sequence |

### A.3 Key Methods

*   **`AddAction(TriggerEffectData)`** — main entry; compute User → set iData.User → push all actions to queue.
*   **`DefaultAction`** (static) — `Util.GetData(RCG_BattleStartActionGenData.DefaultID)`, "None" by default.

### A.4 System Interactions

*   **`RCG_SelectTargetData`** — executor selection rule.
*   **`RCG_BattleSetting`** — each action node.
*   **`RCG_BattleManager.PlayerBattleUnits`** — target fallback source.
*   **`AddActionMode.PushBack`** — action queue insertion mode.

### A.5 Known Issues

*   Full `OnGUI` implementation is commented out; uses base class default rendering.
