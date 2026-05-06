---
title: Cmd_ExportNotes API
description: Exports Card / Equipment / Item / Story Note / Description to Markdown based on the targets parameter. Consolidates the legacy Cmd_ExportAllNotes / Cmd_ExportCardNotes / Cmd_ExportEquipmentNotes / Cmd_ExportItemNotes commands into one, and adds story support (2026-05-06)
source_file: CardGame/Assets/Scripts/RCG_Scripts/RCG_AgentCommands/Cmd_ExportNotes.cs
namespace: RCG.AgentCommands
last_updated: 2026-05-06
target_audience: [AI_Agent, Tools_Maintainer, Game_Designer]
---

# Cmd_ExportNotes

> Class name: `RCG.AgentCommands.Cmd_ExportNotes`
> Source path: [`CardGame/Assets/Scripts/RCG_Scripts/RCG_AgentCommands/Cmd_ExportNotes.cs`](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_AgentCommands/Cmd_ExportNotes.cs)

## 1. Overview

`Cmd_ExportNotes` is an RCG project-level Agent Command that exports the Note / Description of Card / Equipment / Item / Story data as Markdown files (written to `docs/Catalogs/<Type>_Notes_Export.md`), dispatched by the `targets` parameter.

> [!NOTE]
> **Story target behavior** (added 2026-05-06):
> Story has no `m_Note` field; the export mirrors the Editor Preview content — including each Story's Tags / SubStory structure, plus per-SubStory Description / Image / Options table (with Transition + MapEvents summary). Output file: `docs/Catalogs/Story_Notes_Export.md`.

> [!IMPORTANT]
> **Replaces 4 legacy commands** (consolidated 2026-05-06):
>
> | Legacy command | Equivalent new call |
> |---|---|
> | `ExportAllNotes` | `Type:ExportNotes`, `Args:{}` or `targets=all` |
> | `ExportCardNotes` | `Type:ExportNotes`, `Args:{"targets":"card"}` |
> | `ExportEquipmentNotes` | `Type:ExportNotes`, `Args:{"targets":"equipment"}` |
> | `ExportItemNotes` | `Type:ExportNotes`, `Args:{"targets":"item"}` |

Typical use cases:
- **Bulk-refresh all Notes documents** in editor / agent (all 3 types in one shot)
- **Refresh a single type** (just edited Card data and want to inspect the catalog diff)
- **Refresh a custom subset** (just balanced Card + Item, leave Equipment alone)

## 2. Args Schema

| Param | Required | Default | Description |
|---|:-:|---|---|
| `targets` | ❌ | `all` | Comma-separated export targets; tokens are `card` / `equipment` / `item` / `story` / `all` (case-insensitive). Empty string, omitted, or containing `all` exports all four types |

**Valid usage examples**:

| `targets` value | What gets exported |
|---|---|
| (omit Args entirely) | Card + Equipment + Item + Story |
| `all` | Card + Equipment + Item + Story |
| `card` | Card only |
| `story` | Story only (with SubStory structure and Options table) |
| `card,item` | Card + Item |
| `card,story` | Card + Story |
| `Card, Story` | Card + Story (case / whitespace tolerant) |
| `all,card` | Card + Equipment + Item + Story (`all` expands; `card` becomes redundant; deduped) |
| `card,card` | Card only (auto-deduped) |
| `""` or `" , "` | Card + Equipment + Item + Story (empty falls back to all) |

**Invalid input** throws `ArgumentException` (Runner records into `LastRunError`):

| Invalid input | Error message |
|---|---|
| `cards` (extra s) | `Unknown targets: [cards]. Valid: all, card, equipment, item, story` |
| `equipments` | `Unknown targets: [equipments]. Valid: all, card, equipment, item, story` |
| `stories` (extra ies) | `Unknown targets: [stories]. Valid: all, card, equipment, item, story` |
| `random` | `Unknown targets: [random]. Valid: all, card, equipment, item, story` |

## 3. Internal Behavior

Each target maps to a static method call on the corresponding `EditorPage` (order: card → equipment → item):

| target | Calls | Output file |
|---|---|---|
| `card` | `RCG_CardDataEditorPage.ExportNotesToMarkdown()` | `docs/Catalogs/Card_Notes_Export.md` |
| `equipment` | `RCG_EquipmentDataEditorPage.ExportNotesToMarkdown()` | `docs/Catalogs/Equipment_Notes_Export.md` |
| `item` | `RCG_ItemDataEditorPage.ExportNotesToMarkdown()` | `docs/Catalogs/Item_Notes_Export.md` |
| `story` | `RCG_StoryDataEditorPage.ExportNotesToMarkdown()` | `docs/Catalogs/Story_Notes_Export.md` |

> [!NOTE]
> **Error handling**: If any EditorPage throws during execution, **the remaining targets do NOT continue** (fail fast). Runner records `LastRunResult: Failed` plus the exception message; user can re-run after fixing the underlying issue.

## 4. Calling from queue.json

**Export everything (most common)**:
```json
{
  "Id": "20260506-export-all",
  "Type": "ExportNotes",
  "Mode": "OneShot",
  "Args": {},
  "Description": "Refresh Card / Equipment / Item Notes (all 3 types)"
}
```

**Export Card only**:
```json
{
  "Id": "20260506-export-card",
  "Type": "ExportNotes",
  "Mode": "OneShot",
  "Args": { "targets": "card" },
  "Description": "Refresh Card Notes only (just rebalanced Card data)"
}
```

**Export Card + Item combo**:
```json
{
  "Id": "20260506-export-cardItem",
  "Type": "ExportNotes",
  "Mode": "OneShot",
  "Args": { "targets": "card,item" },
  "Description": "Card + Item dual refresh (Equipment unchanged)"
}
```

## 5. Python Wrapper Invocation

```bash
# Export everything (default = all)
python CardGame/Assets/UCL/UCL_Core/Tools~/AgentCommands/run_cmd.py run ExportNotes

# Card only
python CardGame/Assets/UCL/UCL_Core/Tools~/AgentCommands/run_cmd.py run ExportNotes \
    --arg targets=card

# Card + Item
python CardGame/Assets/UCL/UCL_Core/Tools~/AgentCommands/run_cmd.py run ExportNotes \
    --arg targets=card,item
```

## 6. Design Rationale

### 6.1 Why consolidate 4 commands into 1?

- **Less Registry noise**: The Available Commands dropdown in `UCL_AgentCommandsPage` shrinks from 4 rows to 1
- **Future target additions stay in one place**: e.g. adding `Status` Notes export only touches `Cmd_ExportNotes`'s `KnownTargets` set and `RunSingleTarget` switch — no new file to register
- **Composition flexibility**: The legacy "Card + Item but not Equipment" case required two separate commands; now it's a single `targets=card,item` call
- **Typo defense**: Whitelist check + clear error message (legacy typos in command names also failed at the Registry level, but with vaguer errors)

### 6.2 Why `targets` (plural) instead of `target` (singular)?

`targets` implicitly suggests "multi-select", consistent with the comma-list syntax `card,item`. `target=all` would mislead users into thinking it's a single-value enum that can't be combined.

### 6.3 Why is `all` an explicit string token rather than just empty default?

Both are supported (empty string falls back to `all`), but **explicit `all` is more readable**:
```json
{ "targets": "all" }    // ✅ Intent is unambiguous
{ "targets": "" }       // ⚠ Looks like a forgotten value
{ }                     // ✅ Acceptable (relies on docs noting default = all)
```

## 7. Related Documents

- [docs/Workflows/AgentCommands_Workflow.md](../../../docs/Workflows/AgentCommands_Workflow.md) — Project-level Agent Commands workflow
- [docs/Workflows/HelpURL_Workflow.md](../../../docs/Workflows/HelpURL_Workflow.md) — How this doc's path is resolved (`eov_docs:` prefix)
- [Cmd_ExportNotes.cs source](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_AgentCommands/Cmd_ExportNotes.cs)
