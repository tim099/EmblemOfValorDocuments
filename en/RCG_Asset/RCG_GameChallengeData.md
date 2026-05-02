---
title: Game Challenge (RCG_GameChallengeData)
description: Global game challenge goal — completing this is what counts as "challenge achieved" (different from Achievement)
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Game Challenge

> Class name: `RCG_GameChallengeData`

## Purpose

**Global "clear-game challenge goal" setting**. Examples: "Defeat the final boss", "Clear without using items", "End the battle within 3 turns". Challenges are at the **clear-game goal** level, different from `RCG_AchievementAsset` — challenges determine "whether this run is considered cleared".

Inherits from `RCG_Asset<RCG_GameChallengeData>`. Implements: `RCGI_Unloackable`.

## Editor Layout

```
RCG_GameChallengeData: <ID>
    Name             ← challenge name (localized; default "GameChallenge")
    Unlock           ← unlock condition
    ChallengeGoals   ← completion conditions (multiple RCG_QuestGoalData together)
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **Name** | yes | Challenge display name (localized) |
| **Unlock** | no | Unlock condition; empty = default unlocked |
| **ChallengeGoals** | yes | Completion conditions (`RCG_QuestGoalData`); meeting all = cleared |

## Behavior

### Unlock Check (`IsUnlocked`)
*   `m_Unlock.IsEmpty || m_Unlock.Unlocked`: no condition or condition met → unlocked.

### Completion Check
This file itself doesn't handle "complete" judgment; that's done by an external quest manager / challenge manager checking `m_ChallengeGoals` progress.

## Caveats

*   **Default ID `Challenge_FinalBoss`** is the standard "defeat the final boss" challenge; most route clears use this.
*   **Difference from `RCG_AchievementAsset`**: achievements are "extra honors" (condition met → reward / display); challenges are **clear-game judgment** (met → run ends / triggers settlement).
*   **`m_Name` defaults to `"GameChallenge"`**: must be renamed for the UI to show a meaningful title.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_GameChallengeData.cs`
*   **Inherits**: `RCG_Asset<RCG_GameChallengeData>`
*   **Implements**: `RCGI_Unloackable`
*   **AssetGroup**: `EditGameSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | Default `"GameChallenge"` |
| `m_Unlock` | Unlock | `RCG_UnlockEntry` | |
| `m_ChallengeGoals` | ChallengeGoals | `List<RCG_QuestGoalData>` | |

### A.3 Key Methods

*   **`IsUnlocked` (property)** — `m_Unlock.IsEmpty || m_Unlock.Unlocked`.
*   **`UnlockEntry`** — `m_Unlock`.
*   **`LocalizedName`** — `m_Name.Name`.

### A.4 System Interactions

*   **`RCG_QuestGoalData`** — clear condition element.
*   **`RCG_GameChallengeGenData`** — Asset Entry; default `Challenge_FinalBoss`.
*   **`RCG_UnlockEntry`** — unlock system.

### A.5 Known Issues

*   No `Preview` override; uses base default rendering.
