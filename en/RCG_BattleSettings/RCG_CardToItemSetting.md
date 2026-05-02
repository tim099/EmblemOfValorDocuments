---
title: Card to item
description: Itemize selected hand cards (creates "play this card" items); supports temp / permanent
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Card to item

> Class: `RCG_CardToItemSetting`

## Purpose
**Convert hand cards into inventory items**. The item's use-effect is "**generate this card to hand and play it**". Examples:
*   "Store an attack card as a potion"
*   "Card-as-item" specialty mechanics

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **SelectHandCardSetting** | Yes | Selector for cards to itemize. |
| **IsTmpItem** | — | Default checked. Checked = auto-removed after battle; unchecked = kept permanently. |

## Behaviour
*   Player chooses cards via selector.
*   Per chosen card:
    1. Clone a new `RCG_ItemData` (copies card name and icon).
    2. Wires the use-effect to `RCG_CardCreateSetting` (generates this card to hand).
    3. Chant cards carry `IsChanted` over to avoid double-chanting.
    4. Registered as `BattleRuntime` data, added to inventory.
*   Originals are banished.
*   Description: `CardToTmpItem_Des` (when temp) or `CardToItem_Des` (permanent).

## Notes
*   **Temp vs permanent persistence**: temp items clear at battle end; permanents save to inventory. Mass-permanent itemization can clutter inventory — design caps.
*   **No fusion after itemization**: originals are banished. To "fuse + itemize", fuse first then itemize.
*   **vs Card fusion**: Card fusion combines multiple cards into one new card; this setting is **card → item** one-way.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardToItemSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_CardToItemSetting` → "Card to item"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_SelectHandCardSetting` | SelectHandCardSetting | `RCG_SelectHandCardSetting` | — | |
| `m_IsTmpItem` | IsTmpItem | `bool` | `IsTmpItem` | Default `true` |

### A.3 Key methods
*   **`AddAction`**: per chosen card:
    1. Build `RCG_ItemData` (`Data.m_Name = m_LocalizeName`, `Data.m_Icon = m_CardIcon`, `ID = aCard.ID`).
    2. Wire `RCG_CommonEffect(OnPlay) → RCG_CardCreateSetting(AddToHandCard, m_RuntimeCardData = new RCG_CardBattleDataPointer(aCard))`.
    3. Chant cards → `aSetting.m_IsChanted = true`.
    4. `RCG_DataService.Ins.AddRuntimeData(aItemData, DataType.BattleRuntime)`.
    5. `new RCG_Item(aItemData, ItemType.Runtime)`, set `m_TmpItem = m_IsTmpItem`, `AddItem()`.
*   **`Info`**: `m_IsTmpItem ? CardInfoData.TmpItemInfo : null`.

### A.4 Cross-system interactions
*   **`RCG_CardCreateSetting + RCG_CardBattleDataPointer`**: the item's use-effect "play stored card" pattern.
*   **`DataType.BattleRuntime`**: end-of-battle cleanup container.
*   **`RCG_AquireItemPanel`**: UI feedback (in code beyond shown range).
