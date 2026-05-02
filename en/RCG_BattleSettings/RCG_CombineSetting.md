---
title: RCG_CombineSetting
description: Responsibility, field semantics, and merge rules of the composite battle setting
last_updated: 2026-05-02
target_audience: [AI_Agent, Gameplay_Programmer, Designer]
---

# RCG_CombineSetting

## 1. Responsibility
Merges multiple `RCG_BattleSetting` instances into a **single** setting that runs them sequentially or simultaneously. This is the go-to container for composite behaviour (e.g. "deal damage + draw card + apply status").

Base class: `RCG_BattleSetting`

## 2. Key Fields

| Field | Type | Meaning |
|---|---|---|
| `m_OverrideDescription` | `bool` | Whether to override the concatenated child description. When `true`, `m_Description` is used instead, avoiding overly long stacked text. |
| `m_Description` | `RCG_LocalizeData` | The override description body; only effective when `m_OverrideDescription = true` (visibility gated by the `Conditional` attribute). |
| `m_CombineSettings` | `List<RCG_BattleSetting>` | List of child battle settings to combine. `AlwaysExpendOnGUI` keeps it expanded in the Inspector. |

## 3. Merge Rules

### 3.1 Playability `CheckPlayable`
**All** children must return `CheckPlayable == true` for the combine to be playable; any failure short-circuits to false (AND logic).

### 3.2 Preview Damage `GetPreviewDamage`
Returns the **maximum** of all children's `GetPreviewDamage` (**MAX logic**) — the UI shows the strongest hit among the children.

### 3.3 Tags / Card Infos
`Infos` / `GetBattleTags()` use **deduplicated concatenation** to prevent the same buff/tag from stacking visually across multiple children.

### 3.4 Description `GetDescription`
*   **No override**: each child's `GetDescription` joined with line breaks (empty strings skipped).
*   **Override**: returns `m_Description.Name` directly; children are not consulted.

## 4. Design Notes
*   **Granularity**: keep each child responsible for a single behaviour (damage, draw, buff, ...) and let CombineSetting do the assembly — this maximises reuse and unit-test coverage.
*   **Avoid deep nesting**: CombineSetting inside CombineSetting works but makes the MAX logic in `GetPreviewDamage` counter-intuitive. Two levels max is recommended.
