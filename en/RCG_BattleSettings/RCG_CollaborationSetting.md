---
title: Collaboration
description: Triggers an effect when enough party members match a skill requirement; or uses the count as a variable
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Collaboration

> Class: `RCG_CollaborationSetting`

## Purpose
**Co-op mechanics**: counts party members matching a skill requirement, then either triggers an effect (threshold mode) or feeds the count to a variable. Examples:
*   "If 2+ Mages in party, deal extra damage"
*   "Heal scales with the count of matching members"

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **CollaborationType** | Yes | Mode:<br>• **Default** — fires only when count ≥ `CollaborationCount`<br>• **Variable** — always fires; count goes to a variable |
| **CollaborationCount** | Default mode | Threshold (default 2). |
| **RequireSkills** | No | Skill list; empty = no restriction (any party member counts). |
| **CollaborationVariable** | Variable mode | Variable name (default `X`); receives the count. |
| **CollaborationSetting** | Yes | The **Combine effect** to run when triggered. |

## Behaviour
*   **Default mode**: count matching allies vs `CollaborationCount`. Threshold met → run `CollaborationSetting`; not met → nothing.
*   **Variable mode**: always runs; writes the matching count to `CollaborationVariable` for use inside `CollaborationSetting` (e.g. "deal X damage").
*   Description includes condition info via i18n `CollaborationTypeInfo_*`, plus child setting description.

## Notes
*   **Empty RequireSkills + Default mode**: every party member counts — verify intent.
*   **Variable name collisions**: multiple settings using `X` overwrite each other; name semantically.
*   **Inner Combine can stack effects**: `CollaborationSetting` is a Combine, but mind the **nesting depth** for description readability.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CollaborationSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_CollaborationSetting` → "Collaboration"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_CollaborationType` | CollaborationType | enum | — | `Default` / `Variable` |
| `m_CollaborationCount` | CollaborationCount | `int` | — | Default 2 |
| `m_RequireSkills` | RequireSkills | `List<RCG_SkillTagGenData>` | — | |
| `m_CollaborationVariable` | CollaborationVariable | `string` | — | `[Conditional("m_CollaborationType", false, Variable)]`; default `"X"` |
| `m_CollaborationSetting` | CollaborationSetting | `RCG_CombineSetting` | — | `[AlwaysExpendOnGUI]` |

### A.3 Key methods
*   **`Infos`**: own description (with `GetCollaborationDes()` + `CollaborationTypeInfo_*` i18n) + `m_CollaborationSetting.Infos`.
*   **`GetCollaborationRequireSkillDes()` (private)**: empty list → `CollaborationRequireSkills_Any`; otherwise `CollaborationRequireSkills_Specify`.
*   **`AddAction`** (beyond shown range): resolves matching count, branches per CollaborationType.

### A.4 Cross-system interactions
*   **`RCG_SkillTagGenData`**: skill tag template.
*   **`RCG_CombineSetting`**: child container.
