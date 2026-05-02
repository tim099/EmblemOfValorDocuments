---
title: Acquire item
description: Adds items to the player's inventory; can flag as temporary (auto-removed after battle)
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Acquire item

> Class: `RCG_AddItemSetting`

## Purpose
Adds one or more items to the player's inventory. Used by "battle reward cards", "pickup events", and "temporary buff item grants".

## Inspector layout
```
▼ ✓ [Acquire item(AddItem)] (item icons)
    Items         ▶ (item list)
    IsTmpItem     [✓]
```

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **Items** | Yes | Items to grant; each is an `RCG_ItemGenData` (item template). |
| **IsTmpItem** | — | Default checked. Checked = auto-removed after battle (no permanent inventory effect); unchecked = kept permanently. |

## Behaviour
*   Adds each item to inventory in order, then pops up the "Acquire item" panel for the player.
*   Temp items are tagged accordingly and cleaned up at battle end.
*   Description: "**Acquire {item1, item2, ...}**"; if temp, appended `[Temp item]` line.

## Notes
*   **Save risk for temp items**: if the player saves mid-battle, temp item cleanup behavior depends on the inventory system; verify with persistence-affecting items.
*   **Empty Items list**: legal but the card grants nothing — treat as data error.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_AddItemSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_AddItemSetting` → "Acquire item"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_Items` | Items | `List<RCG_ItemGenData>` | `Items` | Item templates |
| `m_IsTmpItem` | IsTmpItem | `bool` | `IsTmpItem` | Default `true` |

### A.3 Key methods
*   **`Infos`**: aggregates each `m_Items[i].GetData()` info; appends `CardInfoData.TmpItemInfo` when temp.
*   **`GetDescriptionFormat`**: joins item names with `, `, appends `[TmpItem]` if temp, wraps with `AcquireItem` i18n key.
*   **`AddAction`**: wraps `AddAsyncActionTrigger`, calls `m_Items[i].GetData().AddItem()` for each, sets `m_TmpItem`, opens `RCG_AquireItemPanel`.

### A.4 Cross-system interactions
*   **`RCG_ItemGenData / RCG_Item`**: item template + instance.
*   **`UI.RCG_AquireItemPanel`**: pickup panel UI.
*   **`CardInfoData.TmpItemInfo`**: static "temp item" info block.
