---
title: Animator state
description: Drives the unit's animator (state toggle or one-shot animation); pure visual, no description
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Animator state

> Class: `RCG_AnimatorStateSetting`

## Purpose
**Pure animation control** with no gameplay impact:
*   **Persistent state toggle** (e.g. enter/leave defense stance)
*   **One-shot animation** (e.g. wave, blink)

> [!NOTE]
> **Doesn't appear on card descriptions** (`GetDescriptionFormat` and `GetDescriptionShort` return empty) — purely visual.

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **ActionType** | Yes | • **SetState** — toggle a persistent animator bool (e.g. defense stance)<br>• **PlayAnim** — play a one-shot animation and wait for completion |
| **AnimType** (`UnitAnimType`) | Yes | Which animation to operate (Attack, Hit, Defense, ...). |
| **Enable** | When ActionType=SetState | bool — target value when toggling persistent state. Auto-hidden in PlayAnim mode. |
| **Target** | Yes | Target unit selector. |

## Behaviour
*   **SetState**: per target → `UnitAnimController.SetAnimState(AnimType, Enable)`.
*   **PlayAnim**: per target → `await PlayAnim(AnimType)` until done (**blocks battle flow**).
*   Targets without an animator controller are silently skipped.

## Notes
*   **PlayAnim delays subsequent actions**: it's awaited, lengthening battle pacing. Be careful inside loops.
*   **SetState pairs**: any `SetState(true)` should have a matching `SetState(false)` later, or the state persists till battle end.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_AnimatorStateSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_AnimatorStateSetting` → "Animator state"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_ActionType` | ActionType | `AnimActionType` (enum) | — | `SetState` / `PlayAnim` |
| `m_AnimType` | AnimType | `UnitAnimType` (enum) | — | |
| `m_Enable` | Enable | `bool` | `Enable` | `[Conditional(nameof(m_ActionType), false, AnimActionType.SetState)]` |
| `m_Target` | Target | `RCG_SelectTargetData` | `Target` | |

### A.3 Key methods
*   **`GetDescriptionFormat / GetDescriptionShort`**: return `string.Empty` (intentionally hidden).
*   **`AddAction`**: `AddAsyncActionTrigger` per target — branch on `ActionType`.

### A.4 Cross-system interactions
*   **`UnitAnimController.SetAnimState / PlayAnim`**: actual animation entry points.
*   **`UnitAnimType` enum**: matches Spine animation names.
