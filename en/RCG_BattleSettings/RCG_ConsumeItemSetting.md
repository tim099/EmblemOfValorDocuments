---
title: Consume item
description: Consume specified items from inventory; can act as a "must have item to play" condition
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Consume item

> Class: `RCG_ConsumeItemSetting`

## Purpose
**Consume items from the player's inventory**. Doubles as a **playability condition** — if the items aren't owned, the card can't be played. Examples:
*   "Consume 1 Gunpowder to deal AOE fire damage"
*   "Consume 2 Herbs to heal the party"

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **Items** | Yes | List of `RCG_ItemGenData` to consume; matched by ID, **duplicates require multiple owned**. |

## Behaviour
*   **Playability**: inventory must contain all listed items (matching by ID, including duplicates). Any miss → unplayable.
*   **Trigger**: removes one matching item per list entry.
*   Description: lists item names.

## Notes
*   **TODO**: source notes "**resource changes don't immediately refresh card playability**" — players may see stale "playable" state briefly.
*   **Empty list**: legal but a no-op (always playable, consumes nothing).
*   **Multiple of same ID**: list with same ID twice means consume 2 copies.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ConsumeItemSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_ConsumeItemSetting` → "Consume item"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_Items` | Items | `List<RCG_ItemGenData>` | `Items` | Source comment "獲得的資源" is stale (copy-paste); current use is consumption |

### A.3 Key methods
*   **`CheckPlayable`**: clones `RCG_DataService.Ins.m_ItemsData.GetItems()`, iteratively matches each entry by ID and `RemoveAt`; any miss → false.
*   **`Infos`**: base + each `m_Items[i].GetData()` info (`AddIfNotRepeat`).
*   **`AddAction`** (beyond shown range): subtracts the matched items from inventory.

### A.4 Cross-system interactions
*   **`RCG_DataService.Ins.m_ItemsData`**: inventory data source.
*   **`RCG_ItemGenData / RCG_Item`**: item template + instance.

### A.5 Known issues
*   Source comment `Todo: 資源變化時 要刷新卡牌才能正確判斷目前是否能打出` — playability state isn't reactively refreshed.
*   Field comment "獲得的資源" is a copy-paste leftover; field is actually for consumption.
