---
title: Battle Preset Data (RCG_BattlePresetData)
description: A "full configuration" for quickly launching a test battle — monsters + characters + decks + equipment + difficulty, one click StartBattle
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Battle Preset Data

> Class name: `RCG_BattlePresetData`

## Purpose

**Quick configuration for test battles**. Bundles a complete "monsters + characters + deck + equipment + skills + difficulty" set into one Asset; click the in-Editor `StartBattle` button to immediately enter battle (skipping bigmap / character selection / etc.). **Not used in the regular game flow** — purely a debugging tool for QA / designers tuning numbers.

Inherits from `RCG_Asset<RCG_BattlePresetData>`.

## Editor Layout

```
RCG_BattlePresetData: <ID>
    BattleSetGenData   ← the BattleSet referenced
    Players            ← combat character list
    ExtraCards         ← extra starting cards (in addition to Deck)
    Items              ← starting items
    Deck               ← starting deck (replaces default)
    Equipments         ← per-character equipped items (dictionary)
    UnitSkills         ← per-character learned skills (dictionary)
    AllEquipments      ← unequipped items (inventory)
    EnemyType          ← enemy type
    DifficultyData     ← global difficulty data
    Difficulty         ← dynamic difficulty value
    [button] StartBattle ← one-click start in Editor playing
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **BattleSetGenData** | yes | Referenced `RCG_BattleSet` (monster configuration) |
| **Players** | yes | Combat character list |
| **ExtraCards** | no | Cards added to the deck (for testing card combos) |
| **Items** | no | Starting items (potions, scrolls) |
| **Deck** | no | Starting deck; replaces player's deck when not `Default` |
| **Equipments** | no | Character → equipment list dict (which character has which equipment) |
| **UnitSkills** | no | Character → skill list dict |
| **AllEquipments** | no | Inventory items (not equipped) |
| **EnemyType** | no | Enemy type |
| **DifficultyData** | no | Global difficulty (HP / Atk multipliers, enemy skill level) |
| **Difficulty** | no | Dynamic difficulty value (stacks on top of bigmap difficulty) |

## Behavior

### `StartBattleAsync`
Pressing `StartBattle` (visible only in Editor playing) will:
1. Create BigMapManager and enter bigmap.
2. Enter Quest (wrapped in EnterQuestSetting).
3. Apply `m_DifficultyData` and `m_Difficulty` to `RCG_DataService`.
4. Replace player deck (if `m_Deck.ID != Default`).
5. Add ExtraCards / Items / Equipments / UnitSkills, etc.
6. Enter battle scene.

### Preview
Shows underlying BattleSet preview, with a StartBattle button.

## Caveats

*   **For testing only**: not used by the production game flow — don't rely on this data for actual content design.
*   **`m_Deck.ID == DefaultID` skips deck replacement**, using player's current deck — confirm this Asset's `m_Deck` is set for replacement to take effect.
*   **InitActivePowers logic is commented out**: originally applied character init active powers; now superseded by UnitSkill system.
*   **Editor not playing → no StartBattle button**: must enter Play Mode and the Developer Page to use it.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattlePresetData.cs`
*   **Inherits**: `RCG_Asset<RCG_BattlePresetData>`
*   **AssetGroup**: `EditBattleSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_BattleSetGenData` | BattleSet | `RCG_BattleSetGenData` | |
| `m_Players` | Players | `List<RCG_CharacterGenData>` | |
| `m_ExtraCards` | ExtraCards | `List<RCG_CardGenData>` | |
| `m_Items` | Items | `List<RCG_ItemGenData>` | |
| `m_Deck` | Deck | `RCG_DeckGenData` | |
| `m_Equipments` | Equipments | `Dictionary<RCG_CharacterGenData, List<RCG_EquipmentGenData>>` | |
| `m_UnitSkills` | UnitSkills | `Dictionary<RCG_CharacterGenData, List<RCG_UnitSkillGenData>>` | |
| `m_AllEquipments` | AllEquipments | `List<RCG_EquipmentGenData>` | inventory |
| `m_EnemyType` | EnemyType | `RCG_EnemyTypeTagGenData` | |
| `m_DifficultyData` | DifficultyData | `RCG_DifficultyData` | |
| `m_Difficulty` | Difficulty | `int` | |

### A.3 Key Methods

*   **`StartBattleAsync(BigMap, Quest, CancellationToken)`** — main entry; build bigmap → enter Quest → apply difficulty → replace Deck → add Cards/Items/Equipments/UnitSkills.
*   **`Preview`** — shows BattleSet preview + StartBattle button in editor.

### A.4 System Interactions

*   **`RCG_BigMapManager`** / **`RCG_MapManager`** — entry flow core.
*   **`RCG_DataService.Ins.m_DifficultyData / Difficulty`** — applies difficulty.
*   **`RCG_DataService.Ins.m_DeckData`** — replacement deck target.
*   **`RCG_MapEventManager.Reset`** — clears event queue before entering a new battle.

### A.5 Known Issues

*   `// 初始角色能力` / "init character abilities" section commented out — legacy ActivePower-based init logic is deprecated.
