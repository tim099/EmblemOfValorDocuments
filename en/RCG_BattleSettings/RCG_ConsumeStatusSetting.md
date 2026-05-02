---
title: Consume status
description: Spend self status stacks to trigger an effect; insufficient stacks can either skip or block playability
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Consume status

> Class: `RCG_ConsumeStatusSetting`

## Purpose
**Spend self status stacks to trigger an effect**. Example: "Consume 1 Sword Energy → deal 20 physical damage". Useful for "build up + payoff" card patterns.

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **Status** | Yes | Status type to consume (`RCG_CustomStatusGenData`). |
| **Amount** | Yes | Stacks to consume; supports variables. |
| **Setting** | Yes | The **Combine effect** to run after a successful consume. |
| **DetailSetting** | No | Shared with Status setting's DetailSetting (anim / tags). |
| **CheckPlayable** | — | Default off. Off = playable but skips when short on stacks; on = unplayable when short. |

## Behaviour
*   **Stack check**: against `iData.User`'s current stacks of the chosen status.
*   **CheckPlayable mode**:
    *   On: card greys out when stacks insufficient.
    *   Off: card plays but `m_Setting` skips silently.
*   **On success**: subtract stacks (`-amount`) → run `m_Setting`.
*   **Preview damage**: passes through `m_Setting` when condition holds; **0** when not (note: not `-1`, so it's still treated as "attack card" with 0 damage).
*   Description: "**Consume {Amount} {Status icon}**" + child description (i18n `ConsumeStatusDes`).

## Notes
*   **Preview accuracy**: if stacks are dynamic (e.g. "draw card then consume"), preview may be off.
*   **Status must be stackable**: behaviour on non-stackable statuses is undefined.
*   **vs Status setting**: Status grants stacks; this setting **consumes self stacks**.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ConsumeStatusSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_ConsumeStatusSetting` → "Consume status"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_Status` | Status | `RCG_CustomStatusGenData` | `Status` | |
| `m_Amount` | Amount | `IntVariable` | `Amount` | Default 1 |
| `m_Setting` | Setting | `RCG_CombineSetting` | — | |
| `m_DetailSetting` | DetailSetting | `RCG_StatusSetting.DetailSetting` | `DetailSetting` | Reuses Status's DetailSetting |
| `m_CheckPlayable` | CheckPlayable | `bool` | — | Default `false` |

### A.3 Key methods
*   **`CheckCondition(iData, out amount)` (public)**: compares `iData.User.m_UnitStatus.GetStatusLayer(m_Status)` vs `m_Amount.GetValue(iData)`.
*   **`CheckPlayable`**: `m_CheckPlayable = false` → always true; otherwise runs `CheckCondition`.
*   **`AddAction`**: condition met → `CreateAction.StatusAction(User, Status, -amount, ..., DetailSetting)` + `m_Setting.AddAction(InsertInOrder)`.
*   **`GetBattleSettings<T> / (Type)`**: self + recurse `m_Setting`.
*   **`GetDescriptionFormat`**: temp `m_FullSentence = false` → i18n `ConsumeStatusDes` + child format → restore.
*   **`GetPreviewDamage`**: condition false → 0 (**not -1**); true → delegates to `m_Setting`.
*   **`PreloadData`**: `await m_Setting.PreloadData`.

### A.4 Cross-system interactions
*   **`RCG_UnitStatus.GetStatusLayer(status)`**: stack query.
*   **`CreateAction.StatusAction`**: actual stack mutation builder.
*   **`RCG_CustomStatusGenData.IconTMPKey`**: description icon.
