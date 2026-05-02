---
title: Resource alter
description: Gain or consume resources (gold, soul power, ...) during battle
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Resource alter

> Class: `RCG_ResourceAlterSetting`

## Purpose
**Modify the player's persistent resources** (gold, soul power, ...) during battle. Examples:
*   "Gain 50 gold mid-battle"
*   "Spend 1 soul power to enable a powerful card"

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **ResourceAlterMode** | Yes | Mode:<br>• **Add** — gain (default)<br>• **Consume** — spend (**gates playability**) |
| **AcquiredResources** | Yes | Resource list; each is an `RCG_ResourceGenData` (resource type + amount). |

## Behaviour
*   **Playability**: in Consume mode, every listed resource must satisfy current ≥ amount. Any miss → unplayable.
*   **Trigger**: per mode adds / subtracts the resource on `RCG_DataService.Ins`.
*   Description: lists resources; Consume → `ConsumeItem` i18n; Add → `AcquireItem` i18n.

## Notes
*   **TODO**: "**resource changes don't immediately refresh card playability**" — same caveat as ConsumeItemSetting.
*   **Always expanded**: `[AlwaysExpendOnGUI]`.
*   **Empty list**: legal but a no-op.
*   **vs Energy alter**: energy is a per-turn resource within battle; this setting is **persistent global resources** (gold, soul power) — don't conflate.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ResourceAlterSetting.cs`
*   **Inherits**: `RCG_BattleSetting`, `[System.Serializable] + [AlwaysExpendOnGUI]`
*   **i18n class key**: `RCG_ResourceAlterSetting` → "Resource alter"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_ResourceAlterMode` | ResourceAlterMode | enum (file) | — | `Add` / `Consume` |
| `m_AcquiredResources` | AcquiredResources | `List<RCG_ResourceGenData>` | — | |

### A.3 Key methods
*   **`CheckPlayable`**: Consume mode iterates resources, compares `RCG_DataService.Ins.GetResource(type)` to `m_Amount.GetValue`; any short → false. Add mode is always true.
*   **`AddAction`** (beyond shown range): per resource, calls add/sub API.
*   **`Infos`**: base + each resource type info (`AddIfNotRepeat`).
*   **`GetDescriptionFormat / Params`**: joins names with `, `, branches per mode (Consume / Add) for i18n key.

### A.4 Cross-system interactions
*   **`RCG_DataService.Ins.GetResource(type)`**: current resource query.
*   **`RCG_ResourceGenData / RCG_ResourceTypeGenData`**: resource template.
