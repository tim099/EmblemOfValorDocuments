---
title: Runtime Script (RCG_RuntimeScriptData)
description: Bundles a logic script (RCG_RuntimeScript) into an Asset, reusable from many places
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Runtime Script

> Class name: `RCG_RuntimeScriptData`

## Purpose

**Bundles an `RCG_RuntimeScript` logic script into an Asset**. Lets multiple places reuse the same logic without copy-paste (e.g., "common death sequence", "conditional check script").

Inherits from `RCG_Asset<RCG_RuntimeScriptData>`.

## Editor Layout

```
RCG_RuntimeScriptData: <ID>
    RuntimeScript    ← actual script content (RCG_RuntimeScript)
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **RuntimeScript** | yes | Actual script (`RCG_RuntimeScript`, with nodes, variables, execution order, etc.) |

## Behavior

This file is just a container; all logic is in `RCG_RuntimeScript`.

## Caveats

*   **Preview shows only ID**: viewing actual content requires opening the script editor via the edit button.
*   **`RCG_RuntimeScript`'s exact structure** is in the program code (not this Asset's responsibility).

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RuntimeData/RCG_RuntimeScriptData.cs`
*   **Inherits**: `RCG_Asset<RCG_RuntimeScriptData>`
*   **AssetGroup**: `Runtime`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_RuntimeScript` | RuntimeScript | `RCG_RuntimeScript` | |

### A.3 System Interactions

*   **`RCG_RuntimeScript`** — actual script content.
