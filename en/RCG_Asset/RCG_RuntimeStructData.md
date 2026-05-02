---
title: Runtime Struct Data (RCG_RuntimeStructData)
description: "Type table" used by RCG_RuntimeData — defines Int / Float / String / Bool / Json / Enum / Struct / List / Dic structures
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Runtime Struct Data

> Class name: `RCG_RuntimeStructData`

## Purpose

**The "type definition table" used by `RCG_RuntimeData`**. When the system needs to dynamically define data structures (without C# classes, using data instead), this is where you build a Struct: which fields, what types each is. Example: a "player run state" Struct can contain `Score: Int / IsBoss: Bool / Inventory: List<Item>`.

Inherits from `RCG_Asset<RCG_RuntimeStructData>`.

## Editor Layout

```
RCG_RuntimeStructData: <ID>
    StructType  ▾ Int / Float / String / Bool / Json / Enum / Struct / Dic / List
    Name(localized)
    Fields       (StructType=Struct)  ← per-field dict: name → struct type ref
    ElementType  (StructType=List/Dic) ← element type ref
    Enums        (StructType=Enum)     ← enum value list
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **StructType** | yes | 9 types: 5 primitives (Int/Float/String/Bool/Json) + Enum + Struct + List + Dic |
| **Name** | no | Display name (localized) |
| **Fields** | when StructType=Struct | Struct field dict: name → reference to another RuntimeStructData |
| **ElementType** | when StructType=List/Dic | Element type reference |
| **Enums** | when StructType=Enum | List of enum value strings |

## Behavior

### Default Primitive Types (`PrimitiveDic`)
The 5 primitive types are lazily built on first access: `{ "Int": ..., "Float": ..., "String": ..., "Bool": ..., "Json": ... }`. **You don't need to manually create Assets for these** — just reference IDs `"Int"` / `"Float"` etc.

### Generic String Parsing (`CreateData`)
When the ID has angle brackets (e.g., `"List<Int>"` or `"Dic<String,Float>"`), it auto-parses:
*   Splits the StructType string (List / Dic).
*   Splits the element type string (after the last comma).
*   Builds and caches into `GenericDic`.

### Preview (`PreviewData`)
Renders data per StructType in the corresponding UI; recursively expands Struct / List / Dic.

### Editing (`DataOnGUI`)
Renders the corresponding edit UI per StructType (IntField / NumField / TextArea / BoolField / Popup / nested data, etc.).

### JSON Serialization (`SerializeDataToJson` / `DeserializeDataFromJson`)
Full structure supports JSON round-trip; recursively handles Struct/List/Dic.

### Data-Change Detection
`DataOnGUI` uses `iDataDic[PrevDataKey]` to compare current data; on mismatch, clears the GUI sub-dict (avoids stale IntField cache from wrong input).

## Caveats

*   **`Event` type commented out** (in code: `// Event,`): once planned, never implemented.
*   **Don't manually create Assets for primitive types**: `PrimitiveDic` lazy-builds them; manual creation conflicts with auto-generated.
*   **Generic syntax for List / Dic**: use `"List<ElementTypeID>"` / `"Dic<KeyID,ValueID>"` (Key only supports String currently; the first value is ignored, only the last is read).
*   **Recursion cap of 10**: `CreateData(layer)` prevents infinite recursion; deeply nested structs return null.
*   **Modifying Struct's Fields**: existing data isn't auto-migrated; after changing schema, old data dictionaries may have stale keys or missing keys.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RuntimeData/RCG_RuntimeStructData.cs`
*   **Inherits**: `RCG_Asset<RCG_RuntimeStructData>`
*   **AssetGroup**: `Runtime`
*   **Constants**: `s_PrimitiveTypes` = 5 types; `GenericTypeDic` = List/Dic (Dic's element type count)

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_StructType` | StructType | `StructType` enum | 9 types |
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Fields` | Fields | `Dictionary<string, RCG_RuntimeStructGenData>` | `Conditional(Struct)` |
| `m_ElementType` | ElementType | `RCG_RuntimeStructGenData` | `Conditional(List/Dic)` |
| `m_Enums` | Enums | `List<string>` | `Conditional(Enum)` |

### A.3 Key Methods

*   **`PrimitiveDic` / `GenericDic` (static)** — lazy-built caches.
*   **`GetAllIDs(useCache)`** — primitives + generics + base IDs.
*   **`CreateData(string)` (override)** — includes generic string parsing.
*   **`CreateData(int layer = 0)` (instance)** — build initial value (with recursion cap).
*   **`DataOnGUI(data, fieldName, dataDic)`** — main edit UI; branches by StructType.
*   **`PreviewData(data, fieldName)`** — main preview.
*   **`SerializeDataToJson / DeserializeDataFromJson`** — JSON round-trip.
*   **`GetRuntimeObject(obj, key/int)` / `SetData(obj, key/int, val)` / `GetList / GetDictionary / GetFields / GetFieldInfo`** — helpers for external data access.

### A.4 System Interactions

*   **`RCG_RuntimeData`** — Asset that references this type.
*   **`RCG_RuntimeObject`** — dynamic value container.
*   **`RCG_RuntimeStructGenData`** — Asset Entry; includes constants like `GenericTypeLeft = '<'`.

### A.5 Known Issues

*   `Event` in the enum is commented out; declared but unused.
*   `IsNone` only checks empty Fields for Struct type; other types never count as None.
*   `CreateData`'s generic parsing only takes the **last** type parameter (works for Dic, may need future extension).
