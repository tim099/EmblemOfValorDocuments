---
title: Field Effect (RCG_FieldEffectData)
description: Battlefield-wide environmental effect template (fire field, darkness shroud, per-turn -HP, etc.)
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Field Effect

> Class name: `RCG_FieldEffectData`

## Purpose

**Template for environmental effects applied to the entire battlefield**. Unlike statuses (CustomStatus) which are attached to units, field effects are **field-wide** — e.g., "Fire field: all units take -3 HP per turn", "Darkness: all attacks have -20% accuracy", "Holy: all heals +50%".

Inherits from `RCG_Asset<RCG_FieldEffectData>`. Implements: `RCGI_Infos` / `UI.RCGI_StatusInfo`.

## Editor Layout

```
RCG_FieldEffectData: <ID>
    Name(localized)
    Icon                 ← field effect icon
    Effects              ← trigger effects (applied to the entire field by trigger)
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **Name** | yes | Field effect name (localized) |
| **Icon** | yes | Icon, default `FieldEffectIcons_volcano.png` |
| **Effects** | yes | Effects to fire on each trigger (`RCG_CommonEffect` list) |

## Behavior

### Triggering
`OnUnitState(triggerOn, data)` → fetches effects from `m_Effects` for the trigger, fires in order. **`OnUnitState` naming is legacy** — field effects aren't necessarily tied to unit states; they can fire on any trigger (OnBattleStart / OnTurnStart / OnTurnEnd, etc.).

### Description
Auto-composed from `Effects` (with BattleTag explanations).

## Caveats

*   **No layer mechanic**: field effects are binary (active / not active), unlike CustomStatus which has layers.
*   **Multiple field effects** can stack (managed by BattleManager's active list); UI shows all currently active.
*   **Source**: `RCG_BattleSceneData.m_FieldEffectDrops` references `RCG_FieldEffectDropPool`; on battle start, the pool draws this battle's field effects.
*   **`RCGI_Status` interface not implemented**: the class declaration has `//, RCGI_Status` commented out — was once planned but unimplemented.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_FieldEffectData.cs`
*   **Inherits**: `RCG_Asset<RCG_FieldEffectData>`
*   **Implements**: `RCGI_Infos` / `UI.RCGI_StatusInfo`
*   **AssetGroup**: `EditBattleSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Icon` | Icon | `RCG_SpriteData` | Default `FieldEffectIcons_volcano.png` |
| `m_Effects` | Effects | `List<RCG_CommonEffect>` | |

### A.3 Key Methods

*   **`OnUnitState(triggerOn, data)`** — fire effects (legacy name).
*   **`TriggerOnUnitState(triggerOn)`** — quick check.
*   **`Description` (property)** — `m_Effects.GetDescription(showBattleTags = true)`.
*   **`Infos` (property)** — `m_Effects.GetInfos()`.

### A.4 System Interactions

*   **`RCG_BattleSceneData.m_FieldEffectDrops`** — pool referenced by battle scenes.
*   **`RCG_FieldEffectDropPool`** — random pool of field effects.
*   **`RCG_BattleManager.TriggerFieldEffect`** — runtime entry where effects are fired.
*   **`RCG_FieldEffectGenData`** / **`RCG_FieldEffectDropPoolGenData`** — Asset Entry wrappers.

### A.5 Known Issues

*   Class declaration has `, RCGI_Status` commented out — once planned, never implemented.
*   `m_AcquireSkillEvents` etc. commented out in code — likely dead code copied from UnitSkill.
