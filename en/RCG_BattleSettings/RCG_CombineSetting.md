---
title: Combine effect
description: Bundle multiple battle settings (attack, heal, draw, ...) into a single composite effect; the most common container type
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Combine effect

> Class: `RCG_CombineSetting`

## Purpose
Bundles **multiple** battle settings (Attack, Heal, Draw, Status, ...) into a single unit that fires **together**. The go-to container for composite cards:
*   "Deal 3 damage + draw 1 card"
*   "Grant 2 armor + add 1 Agile stack"
*   "Attack front row + self-damage 1 HP"

## Inspector layout
```
▼ ✓ [Combine effect(Combine)] [thumbnails]
    OverrideDescription   □
    Description           [hidden / shown]
    CombineSettings       ▶ (expand for child setting list)
```

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **OverrideDescription** | No | Whether to override the auto-concatenated child description. When checked, **Description** is used instead, avoiding bloated text. |
| **Description** | When OverrideDescription | Custom description body (`RCG_LocalizeData`); auto-hidden when OverrideDescription is unchecked. |
| **CombineSettings** | Yes | The list of child battle settings to run. Each item is a full battle setting (Attack / Heal / Draw / ...). |

## Behaviour

### How the description renders
*   **OverrideDescription off**: child descriptions joined by newlines (empty entries skipped).
*   **OverrideDescription on**: shows **Description** directly; children don't participate.

> [!TIP]
> If concatenated children become a wall of text, **try OverrideDescription** and write a clean summary yourself — but remember to keep it in sync manually.

### Playability
**All** children must be playable for the combine to be playable. Any failure short-circuits the whole card (AND logic).

### Preview damage
The UI's preview damage takes the **maximum** of all children (e.g. children of 5 / 8 / 2 → UI shows 8).

### Tags / card info
Children's tags and buff icons are **deduplicated** before display, so the same buff doesn't appear twice in the tooltip.

## Notes

*   **Keep child granularity tight**: each child should do one thing (damage / draw / status). Let Combine handle the assembly — easier to reuse.
*   **Avoid deep nesting**: Combine inside Combine works but makes preview damage and description rendering counter-intuitive. **Two levels max**.
*   **Empty list is legal but useless**: a Combine with no children fires nothing — treat it as a data error.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CombineSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_CombineSetting` → "Combine effect"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_OverrideDescription` | `OverrideDescription` | `bool` | `OverrideDescription` | Gates `m_Description` visibility |
| `m_Description` | `Description` | `RCG_LocalizeData` | `Description` | `[Conditional(nameof(m_OverrideDescription), false, true)]` |
| `m_CombineSettings` | `CombineSettings` | `List<RCG_BattleSetting>` | `CombineSettings` | `[AlwaysExpendOnGUI]` |

### A.3 Key methods
*   **`CombineSettings` (property)** → `m_CombineSettings.GetEnableBattleSettings()` filters `IsEnable=false`.
*   **`CheckPlayable`** → AND across children.
*   **`GetPreviewDamage`** → `Mathf.Max` over children, init `-1`.
*   **`Infos / GetBattleTags`** → `AppendIfNotRepeat` deduplication.
*   **`GetDescription`**: branches on `m_OverrideDescription` — direct `Name` or newline-joined children.
*   **`GetBattleSettings<T> / (Type)`** → self if matches + recurse `CombineSettings`.

### A.4 Cross-system interactions
*   **`RCG_LoopSetting`** wraps a `RCG_CombineSetting` as `m_LoopContent`; Loop is built atop Combine for description aggregation.
*   **`RCG_BattleTagCombineSetting`** inherits `RCG_CombineSetting` to specialize battle-tag handling.

### A.5 Known issues
*   Legacy `m_OverridingDescriptionKey` field is commented out; replaced by current `m_Description`.
