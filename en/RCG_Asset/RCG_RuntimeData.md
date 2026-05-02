---
title: Runtime Data (RCG_RuntimeData)
description: Generic runtime variable container — dynamic structure defined by RuntimeStructData (Int / Bool / Struct / List / Dic / Enum, etc.)
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Runtime Data

> Class name: `RCG_RuntimeData`

## Purpose

**Generic runtime variable container**. Anywhere the system needs "a slot with dynamic structure" uses this — e.g., battle phase state, card trigger timing counters, various temporary flags. The actual structure is defined by `RCG_RuntimeStructData` (the type table); `RCG_RuntimeData` is **one instance of that structure**, with default values.

Inherits from `RCG_Asset<RCG_RuntimeData>`. Implements: `UCLI_FieldOnGUI`.

## Editor Layout

```
RCG_RuntimeData: <ID>
    StructType   ← referenced RCG_RuntimeStructData ID (type definition)
    DefaultValue ← initial value of this instance (rendered dynamically by StructType)
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **StructType** | yes | Referenced `RCG_RuntimeStructData` ID (defines the data shape of this instance) |
| **DefaultValue** | yes | Initial value of this instance; auto-rebuilt when type changes |

## Behavior

### Auto-Rebuild on Type Change
The `DefaultValue` getter compares `m_DefaultValue.ID` with `m_StructType.ID`; if they differ, sets it to null and rebuilds. **Avoids stale-type data lingering.**

### Serialization
`SerializeToJson` outputs base + `DefaultValue` (key = `DefaultValueKey = "DefaultValue"`); deserialize only overwrites DefaultValue if the key is present.

### Default Fetch (`InGameData`)
`RCG_RuntimeData.InGameData` fetches the Asset for `RCG_RuntimeGenData.InGameDataID`, used as "global in-game variable storage".

### `RCG_RuntimeDataConst` (in-file subclass)
Skips m_StructType editing, displaying DefaultValue directly. Marked `[UCL_IgnoreAsset]` so it doesn't appear as a standalone Asset.

## Caveats

*   **`StructType` must exist first**: corresponding `RCG_RuntimeStructData` must be created before referencing here.
*   **Changing StructType clears existing DefaultValue**: type mismatch triggers rebuild; old data is lost.
*   **`DefaultType` enum lists common IDs**: `BattleState` / `CardTriggerTiming` are shortcuts for common built-in struct types.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RuntimeData/RCG_RuntimeData.cs`
*   **Inherits**: `RCG_Asset<RCG_RuntimeData>`
*   **Implements**: `UCLI_FieldOnGUI`
*   **AssetGroup**: `Runtime`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_StructType` | StructType | `RCG_RuntimeStructGenData` | type table reference |
| `m_DefaultValue` (private) | DefaultValue | `RCG_RuntimeObject` | dynamically built via property |

### A.3 Key Methods

*   **`InGameData` (static)** — `Util.GetData(InGameDataID)`, global in-game variable storage.
*   **`DefaultValue` (property)** — includes type-mismatch auto-rebuild logic.
*   **`SerializeToJson / DeserializeFromJson`** — serializes nested DefaultValue.
*   **`GetEnumIDs()`** — for Enum types, returns all enum string values.
*   **`OnGUIDefaultValue(field, dataDic)`** — renders the DefaultValue editor.
*   **`OnGUI(field, dataDic, params)`** — `UCLI_FieldOnGUI` implementation.

### A.4 System Interactions

*   **`RCG_RuntimeStructData`** — type definition source.
*   **`RCG_RuntimeObject`** — dynamic value container.
*   **`RCG_RuntimeGenData`** — Asset Entry.
*   **`RCG_RuntimeDataConst`** (same file) — non-standalone constant variant.

### A.5 Known Issues

*   `RCG_RuntimeDataConst` marked `[UCL_IgnoreAsset]` but still referenceable; avoid accidentally creating Assets.
