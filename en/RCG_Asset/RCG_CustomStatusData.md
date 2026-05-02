---
title: Custom Status (RCG_CustomStatusData)
description: Statuses that can be applied to units in battle (buff / debuff / DoT, etc.) — layer changes, offsets, immunity, resistance, atk/def buffs
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Custom Status

> Class name: `RCG_CustomStatusData`

## Purpose

**Template for statuses applied to units in battle (buff / debuff)**. Examples: "Poison: -5 HP per turn", "Strength +2", "Immune to negatives", "Dodge: layer ≥ attack power negates the hit". Centralizes all status-related properties: layer change rules, offset relations, immunity / resistance, atk/def buff bonuses, special unit states (stun, chant, etc.).

Inherits from `RCG_Asset<RCG_CustomStatusData>`. Implements: `RCGI_Status`.

## Editor Layout

```
RCG_CustomStatusData: <ID>
    Name / StatusEffectType / StatusClass
    AtkBuff / DefBuff           ← attack/defense bonus settings
    IconSprite / Tags
    LayerIncreaseVFX / StatusPersistantVFX
    StatusOffsets / OffsetTags  ← offsetting statuses
    StatusAlterOn               ← which triggers alter the layer
    Effects                     ← trigger effects
    UnitStates                  ← corresponding special states (stun, dodge, etc.)
    StopDecayStatus             ← prevents decay of target statuses
    ImmuneStatus                ← immune statuses (won't be applied)
    Resistance                  ← resistance (1% chance per layer to nullify the target status)
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **Name** | yes | Status name (localized) |
| **StatusEffectType** | yes | `Buff` / `Debuff` / etc.; affects color and offset checks |
| **StatusClass** | — | `Normal` / `AtkBuff` / `DefBuff` / `Immune` (all-negative immunity) |
| **AtkBuff / DefBuff** | no | Multiple atk / def buff settings (damage multiplier, armor add, etc.) |
| **IconSprite** | yes | Status icon (`RCG_IconSprite` reference) |
| **Tags** | no | Status category tags (Buff / Debuff / DoT / Feature, etc.) |
| **LayerIncreaseVFX** | — | VFX on layer increase |
| **StatusPersistantVFX** | — | Persistent unit VFX (always shown while held) |
| **StatusOffsets** | no | Specific offsetting statuses (e.g., Strength ↔ Weakness) |
| **OffsetTags** | no | Offsets by **tag type** (e.g., a buff that offsets all Debuff tags) |
| **StatusAlterOn** | no | Which triggers alter the layer (dictionary: `trigger → DecreaseType`) |
| **Effects** | no | Trigger effects |
| **UnitStates** | no | Corresponding unit special states enum (`Stun` / `Guard` / `Dodge`, etc.) |
| **StopDecayStatus** | no | List of statuses that won't decay |
| **ImmuneStatus** | no | Immune statuses (won't be applied) |
| **Resistance** | no | 1% chance per layer to nullify target status |

## Behavior

### Layer Change (`GetDecreaseType`)
Looks up `m_StatusAlterOn` table, returns the `eStatusDecreaseType` for the trigger:
*   `None` no change / `DecreaseOneLayer` -1 / `Clear` clear all / `DecreaseHalf` halve
*   `AddOneLayer` +1 / `ClearImmediately` immediate clear / `Eliminate` remove (no end effect)

### Offset Check (`CheckIsOffset`)
*   `StatusClass = Immune` → all statuses with `StatusDebuff` Tag are offset.
*   `OffsetTags` matches target's Tag → offset.
*   `StatusOffsets` contains target ID → offset.

### Description Generation
Two sections (separated by blank line):
1. **Offsets section**: lists statuses this offsets.
2. **Effects section**: UnitStates description / AtkDefBuff description / StopDecay / Immune / Resistance / Effects trigger description.

### IconTMPKey
When `RCG_BattleSetting.IsShowOnUI = true` → returns IconSprite TMPKey (for TextMeshPro icons); otherwise returns LocalizedName.

### Trigger Effects (`OnTriggerEffect`)
Fetches effects from `m_Effects` for the trigger, fires in order.

## Caveats

*   **`Immune` class auto-offsets all Debuffs**: no need to manually list `OffsetTags = StatusDebuff`; logic is built in.
*   **`m_AtkBuff / m_DefBuff` are lists**: stack multiple (different conditions, different bonuses); legacy `m_AtkBuffData / m_DefBuffData` (single) replaced and commented out.
*   **`UnitStates` maps directly to enum**: stun, dodge, guard, etc. are 13 hardcoded special states; can't be added (requires code changes).
*   **Resistance is probabilistic**: 1% per layer; 10 layers = 10% resistance; doesn't stack to 100%.
*   **StatusFeature tag**: status with this tag is prefixed with "Feature: " in display.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_CustomStatusData.cs`
*   **Inherits**: `RCG_Asset<RCG_CustomStatusData>`
*   **Implements**: `RCGI_Status`
*   **AssetGroup**: `EditCharacter`

### A.2 Field Mapping (selected)

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_StatusEffectType` | StatusEffectType | `StatusEffectType` enum | `Buff` / `Debuff` |
| `m_StatusClass` | StatusClass | `StatusClass` enum | `Normal` / `AtkBuff` / `DefBuff` / `Immune` |
| `m_AtkBuff` | AtkBuff | `List<RCG_AtkBuffData>` | |
| `m_DefBuff` | DefBuff | `List<RCG_DefBuffData>` | |
| `m_IconSprite` | IconSprite | `RCG_IconSpriteGenData` | |
| `m_Tags` | Tags | `List<RCG_StatusTagGenData>` | |
| `m_LayerIncreaseVFX` | LayerIncreaseVFX | `RCG_CommonVFXGenData` | Default `VFX_StatusLayerID` |
| `m_StatusPersistantVFX` | StatusPersistantVFX | `RCG_CommonVFXGenData` | |
| `m_StatusOffsets` | StatusOffsets | `List<RCG_CustomStatusGenData>` | |
| `m_OffsetTags` | OffsetTags | `List<RCG_StatusTagGenData>` | |
| `m_StatusAlterOn` | StatusAlterOn | `Dictionary<RCG_EffectTriggerOn, eStatusDecreaseType>` | |
| `m_Effects` | Effects | `List<RCG_CommonEffect>` | |
| `m_UnitStates` | UnitStates | `List<UnitState>` | enum: Stun / Guard / Dodge / etc. |
| `m_StopDecayStatus` | StopDecayStatus | `List<RCG_CustomStatusGenData>` | |
| `m_ImmuneStatus` | ImmuneStatus | `List<RCG_CustomStatusGenData>` | |
| `m_Resistance` | Resistance | `List<RCG_CustomStatusGenData>` | |

### A.3 Key Methods

*   **`GetDecreaseType(triggerOn)`** — looks up m_StatusAlterOn table.
*   **`CheckIsOffset(targetStatus)`** — three-layer check: Immune class / OffsetTags / StatusOffsets.
*   **`OnTriggerEffect(triggerOn, data)`** — fire effects.
*   **`TriggerOnUnitState(triggerOn)`** — quick check (will it change or trigger anything).
*   **`Description` (property)** — large StringBuilder composition: Offsets + UnitStates + AtkBuff + DefBuff + StopDecay + Immune + Resistance + Effects.
*   **`IconTMPKey`** — UI mode → TMPKey, else LocalizedName.
*   **`CreateLayerIncreaseVFX / CreateStatusPersistantVFX`** — async VFX creation.

### A.4 System Interactions

*   **`RCG_StatusGenData`** — Asset Entry; `Status` (property) returns `new RCG_StatusGenData(ID)`.
*   **`RCG_CustomStatusGenData`** — direct reference type; includes `s_Default` / `s_ChargeUp` static instances.
*   **`RCG_AtkBuffData` / `RCG_DefBuffData`** — atk/def buff sub-data.
*   **`UnitState` (enum)** — 13 hardcoded special states.
*   **`RCG_VFXManager`** — VFX creation.

### A.5 Known Issues

*   Legacy `m_AtkBuffData` / `m_DefBuffData` (single) replaced by lists; deserialize migration is commented out.
*   `Init()` is empty; reserved for bigmap entry initialization.
