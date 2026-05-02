---
title: Item Data (RCG_ItemData)
description: Template for player consumable / quest items (potions, scrolls, keys, etc.) — rarity, effects, use times, unlock
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Item Data

> Class name: `RCG_ItemData`

## Purpose

**Template for items the player can hold / use**. Potions, scrolls, food, quest keys, healing items — all are instances of this class. Each item has its own icon, effects, rarity, use limit, unlock conditions.

Inherits from `RCG_Asset<RCG_ItemData>`. Implements: `RCGI_Item` (can be added to inventory) / `RCGI_Unloackable` (unlockable).

## Editor Layout

```
RCG_ItemData: <ID>
    Data (m_Data)              ← main settings (nested class)
    Effects (m_ItemEffects)    ← item effects (OnPlay trigger)
    Preview                    ← live preview
```

## Main Fields (within Data)

| Editor Display | Required | Description |
|---|---|---|
| **Name** | yes | Item name (localized) |
| **Description** | no | Flavor description (localized; appended after auto description) |
| **TargetType** | yes | Target selection range when using (None / Friend / Enemy, etc.) |
| **Price** | yes | Merchant sell price; default 20, can be reset by `Auto Price` button (3 × rarity value) |
| **Icon** | yes | Item icon |
| **Rarity** | yes | Rarity tag |
| **ItemType** | yes | `Normal` (regular) or `QuestItem` (quest-only, story use) |
| **ItemUseType** | yes | `Consume` (consumed on use) / `OncePerBattle` / `OncePerTurn` / `Infinite` |
| **UseTimes** | depends | Use count (effective in Consume / OncePer* modes) |
| **Unlock** | no | Unlock conditions |
| **HideInCodex** | — | Hide in codex (test items) |

Outer class also has **m_ItemEffects** = `List<RCG_CommonEffect>`, describing effects triggered on use (OnPlay is the main trigger).

## Behavior

### Use Flow
1. Player selects an item from inventory → enters `RCG_ItemInfoPanel`.
2. `CheckUsable(data)`: runs `CheckPlayable` on each effect; usable only when all are true.
3. `TriggerEffect(data)`: fetches all `OnPlay` effects and triggers them in order (with try-catch).
4. Per `ItemUseType`, decides whether to consume / lock to next turn / lock to next battle.

### Description Generation
*   Auto-composed from `Effects` (one per line).
*   Appends: non-Normal ItemType marker, non-Consume UseType marker.
*   When `UseTimes > 1`, appends "Remaining N/M" (only at runtime).
*   When `m_Description` is set → appends flavor text after the auto description.

### Auto Price (Editor Tool)
The editor's `RCG_ItemDataEditorPage` has an `Auto Price` button: applies `Price = 3 × Rarity.m_Value` to all ItemData. **Overwrites manual prices.**

## Caveats

*   **`UseTime` returns 0 when `ItemUseType = Infinite`**: runtime doesn't consume / lock.
*   **`m_Description` stacks with auto description**: hand-written text shows after effects.
*   **QuestItem won't be consumed by general use flow**: usually triggered through story events.
*   **Auto Price overwrites manual prices** — confirm before pressing.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_ItemData.cs`
*   **Inherits**: `RCG_Asset<RCG_ItemData>`
*   **Implements**: `RCGI_Item` / `RCGI_Unloackable`
*   **AssetGroup**: `EditItems`

### A.2 Field Mapping (outer)

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_Data` | Data | `ItemData` (nested) | `[SerializeField] protected` |
| `m_ItemEffects` | Effects | `List<RCG_CommonEffect>` | |

`ItemData` nested fields: `m_Name` / `m_Description` / `m_TargetType` / `m_Price` / `m_Icon` / `m_Rarity` / `m_ItemType` / `m_ItemUseType` / `m_UseTimes` / `m_Unlock` / `m_HideInCodex`.

### A.3 Key Methods

*   **`AddItem()`** — player acquisition: `RCG_Item.Create(ID)` → `m_ItemsData.AddItem`.
*   **`CheckUsable(TriggerEffectData)`** — AND check on all enabled effects' `CheckPlayable`.
*   **`TriggerEffect(TriggerEffectData)`** — fetches and fires `OnPlay` effects (try-catch).
*   **`Description` / `FullDescription` / `GetDescription`** — three-tier description generation.
*   **`Effects` (property)** — `m_ItemEffects.GetEnableEffects()`.
*   **`Infos` (property)** — aggregates effect infos + ItemType description.
*   **`CreateSelectAssetPage`** — `RCG_ItemDataEditorPage.Create()`.

### A.4 System Interactions

*   **`RCG_Item`** — runtime item instance.
*   **`RCG_DataService.Ins.m_ItemsData`** — player inventory storage.
*   **`RCG_ItemDataEditorPage`** — editor main page (with Auto Price tool).
*   **`RCG_ItemInfoPanel`** — detail UI.

### A.5 Known Issues

*   `m_UseTimes`'s "暫時未完成 先隱藏" / "temporarily incomplete, hidden" comment hints variable-uses isn't fully implemented.
*   `SerializeToJson` / `DeserializeFromJson` overrides are commented out; legacy auto-pricing migration logic was once planned.
