---
title: Player Character Data (RCG_CharacterData)
description: Template for player-selectable / recruitable characters — HP, starting deck, starting skills, unlock conditions, join bonus
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Player Character Data

> Class name: `RCG_CharacterData`

## Purpose

**Template for characters the player can select or recruit during the story**. Each hero, party member, or recruitable NPC is an `RCG_CharacterData`. Contains: basic info, starting HP, starting deck, starting skills/classes, unlock conditions, "join deck" (the deck this character brings when joining mid-game), etc.

Inherits from `RCG_Asset<RCG_CharacterData>`. Implements: `UCLI_ShortName` / `RCGI_Unloackable` (unlock system).

## Editor Layout

```
RCG_CharacterData: <ID>
    Data (UnitData)            ← editing target (HP / Deck / Skills / Order / Unlock)
    RuntimeData                ← runtime-only data (equipment, current skills, etc.)
    Preview (right side)       ← avatar / name / max HP / skills / decks
```

## Main Fields (within Data)

| Editor Display | Required | Description |
|---|---|---|
| **Name** | yes | Character display name (localized) |
| **MaxHp** | yes | Max health |
| **Avatar** | yes | Avatar / portrait sprite |
| **Intro** | no | Character introduction text (used in selection screen) |
| **Order** | yes | Sort index in selection screen |
| **Deck** | yes | Starting deck (`RCG_DeckData`) |
| **JoinDeck** | no | Deck brought when joining mid-game (different progression from Deck) |
| **UnitSkillDatas** | no | Skills already learned at start |
| **SkillTags** | yes | Character classes (Warrior / Mage / Cleric ...), used for card skill checks |
| **Unlock** | no | Unlock condition (`RCG_UnlockEntry`). `UnlockType.Tutorial` means tutorial-unlocked; shop-unlocked also requires GameRecord purchase entry |

## Behavior

### Selection Categorization
The class provides four static methods to fetch character lists (sorted by `m_Order`):
*   `GetAllTutorialCharacters()` — tutorial-unlocked characters
*   `GetAllUnlockedCharacters()` — already unlocked
*   `GetAllLockedCharacters()` — locked
*   `GetAllJoinCharacters()` — tutorial + unlocked (the recruitable ones)

### Unlock Check (`Unlocked`)
*   `UnlockEntry.Unlocked == true`: unlock condition met → for shop-unlock characters, also check `RCG_GameRecord.UnlockedCharacters` for purchase record.
*   `UnlockType.None`: always unlocked.
*   `UnlockType.Tutorial`: requires tutorial completion record in GameRecord.

### Editor Page
*   **OnGUI** shows full Data editor on the left + preview on the right.
*   **TestDropSkills** button (built-in test tool) previews skill drop combos.

## Caveats

*   **`Order` affects selection screen sort**: omitted → stacks at the top.
*   **`JoinDeck` and `Deck` are separate**: starting characters use `Deck`; mid-story recruits use `JoinDeck` (usually leaner to avoid disrupting the deck balance).
*   **Two-stage unlock**: `UnlockEntry` + shop purchase record; check both when modifying unlock logic.
*   **`m_RuntimeData`** is runtime-mutated state (equipment, current skills) — don't edit it.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_CharacterData.cs`
*   **Inherits**: `RCG_Asset<RCG_CharacterData>`
*   **Implements**: `UCLI_ShortName` / `RCGI_Unloackable`
*   **AssetGroup**: `EditCharacter`

### A.2 Field Mapping (outer)

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_Data` | Data | `UnitData` (nested) | Editing-time data |
| `m_RuntimeData` | RuntimeData | `UnitRuntimData` | Runtime-only data |

`UnitData` contains `m_LocalizeName` / `m_Avatar` / `m_Intro` / `m_MaxHp` / `m_Deck` / `m_JoinDeck` / `m_UnitSkillDatas` / `m_SkillTags` / `m_Unlock` / `m_Order`, etc.

### A.3 Key Methods

*   **`Unlocked` (property)** — two-stage unlock check (UnlockEntry → GameRecord).
*   **`GetAllTutorialCharacters / GetAllUnlockedCharacters / GetAllLockedCharacters / GetAllJoinCharacters`** (static) — selection screen.
*   **`Preview` / `OnGUI`** — editor rendering.
*   **`Data.TestDropSkills(...)`** — built-in drop testing tool.
*   **`Init(string ID)` / `Init(CharacterID)`** — multi-form constructors.

### A.4 System Interactions

*   **`RCG_DeckData`** — referenced by `m_Deck` / `m_JoinDeck`.
*   **`RCG_UnitSkillData`** — referenced by `m_UnitSkillDatas`.
*   **`RCG_SkillTagGenData`** — class system.
*   **`RCG_GameRecord.UnlockedCharacters`** — shop / tutorial unlock record.
*   **`RCG_UnlockEntry`** — unlock conditions.
*   **`UnitRuntimData`** — runtime character state (HP, equipment).

### A.5 Known Issues

*   `Unlocked` has `// QWQ23!!` comment in the `UnlockType.Tutorial` branch indicating legacy logic changes.
*   `m_ActivePowers` was once planned ("active abilities" field), now superseded by the UnitSkill system.
