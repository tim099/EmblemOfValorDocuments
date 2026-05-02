---
title: Unlock Data (RCG_UnlockData)
description: Generic unlock condition (cards / equipment / items / skills / system features) — supports blessing-shop unlock and Steam achievement bridge
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Unlock Data

> Class name: `RCG_UnlockData`

## Purpose

**Generic unlock condition setting**. Multiple Assets (cards / equipment / items / skills / characters / bigmaps / decks / active powers) can reference the same `RCG_UnlockData` to share an unlock condition — e.g., "at level 5, unlock a starting set of cards and equipment" only needs one UnlockData; related Assets set `m_Unlock.ID = "Lv5_Unlock"`.

Also supports "blessing shop unlock" (after meeting the condition, still must purchase with blessings) and "system feature unlock" (after unlock, enables features like Final Level / Ban Card / Camp Fire Enhancement, etc.).

Inherits from `RCG_Asset<RCG_UnlockData>`.

## Editor Layout

```
RCG_UnlockData: <ID>
    UnlockSetting       ← unlock condition body (UnlockType: Level / SkillTagLevel / None / Tutorial / Achievement / Never)
    IsBlessingShop      ← whether unlocked through blessing shop
    IsHiddenInCodex     ← hide from codex
    SystemUnlockItems   ← system features enabled when this Asset unlocks (FinalLevel / BanCard / Inheritance...)
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **UnlockSetting** | yes | Main condition setting |
| **IsBlessingShop** | — | true: condition met but still requires blessing-shop purchase to take effect |
| **IsHiddenInCodex** | — | Codex hidden (avoid leaking unlock targets' names) |
| **SystemUnlockItems** | no | List of system features to enable (`SystemUnlockItem` enum) |

`UnlockSetting` contains:

| Sub-field | Description |
|---|---|
| **UnlockType** | `Level` (player level) / `SkillTagLevel` (class level) / `None` (auto-unlock) / `Tutorial` (clear tutorial) / `Achievement` (specific achievement) / `Never` |
| **SkillTag** | when UnlockType=SkillTagLevel: the class |
| **UnlockLevel** | when UnlockType=Level/SkillTagLevel: level threshold |
| **Achievement** | when UnlockType=Achievement: achievement ID |

## Behavior

### `CheckUnlock()` (static)
Scans all UnlockData:
1. Already in `RCG_GameRecord.UnlockRecords` → skip.
2. `UnlockType.None` → directly record unlock + apply `SystemUnlockItems`, but no popup notification.
3. Else: `!IsLocked()` → record + apply SystemUnlockItems; non-blessing-shop items added to return set (for upper-layer popup).
Returns "set of newly unlocked, non-blessing-shop IDs" for UI to display "newly unlocked items".

### `GetUnloackables<T>(id)` (generic static)
Queries all `RCGI_Unloackable` Assets where `UnlockEntry.ID == id`; used to find which cards / equipment / items / skills / characters / bigmaps / decks / active powers correspond to this UnlockData.

### `IsLocked()` (UnlockSetting)
*   `Level` → player level < `m_UnlockLevel`.
*   `SkillTagLevel` → corresponding class level < `m_UnlockLevel`.
*   `None` → false (already unlocked).
*   `Tutorial` → true (this layer always returns true; actual unlock check happens in GameRecord's Tutorial set).
*   `Achievement` → that achievement not unlocked.
*   `Never` → true.

### Editor Preview
Shows four categories of unlockables: CardData / ItemData / EquipmentData / UnitSkillData, paginated. BigMap / Character omitted.

## Caveats

*   **`Tutorial` UnlockType always returns true in IsLocked**: actual unlock path goes through `RCG_GameRecord.Ins.UnlockedCharacters` etc., not `IsLocked`.
*   **`IsBlessingShop = true` unlocks have two stages**: condition met → unlocks shop listing; player purchase → actually usable (recorded in `RCG_GameRecord.UnlockedCards / UnlockedItems / UnlockedEquipments`).
*   **`SystemUnlockItems` is a hardcoded enum**: FinalLevel / BanCard / Inheritance LV1/2 / RestPointEnhancement LV1/2 / Blessing_Shop LV1/2/3 / Secret_Base; adding features requires changing enum + program code.
*   **Disabled unlock**: `UnlockEditor` provides a toggle to disable unlocked records, adding to the `DisabledUnlockRecord` set (debug / test use).

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_UnlockData.cs`
*   **Inherits**: `RCG_Asset<RCG_UnlockData>`
*   **AssetGroup**: `EditGameSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_UnlockSetting` | UnlockSetting | `UnlockSetting` | main condition setting |
| `m_IsBlessingShop` | IsBlessingShop | `bool` | |
| `m_IsHiddenInCodex` | IsHiddenInCodex | `bool` | |
| `m_SystemUnlockItems` | SystemUnlockItems | `List<SystemUnlockItem>` | enum: 10 system features |

### A.3 Key Methods

*   **`CheckUnlock()` (static)** — scans all UnlockData and updates unlock records.
*   **`GetUnloackables<T>(id)` (generic static)** — finds Assets referencing this UnlockData.
*   **`GetLockedCards / GetLockedEquipments / ... ` (instance)** — quick queries by type.
*   **`UnlockSetting.IsLocked()`** — condition check.
*   **`UnlockSetting.GetShortName()`** — auto-generates unlock description string.
*   **`RCG_UnlockEntry.Unlocked / IsShopItem / Name / UnlockType`** — entry helpers for referencing this data.

### A.4 System Interactions

*   **`RCG_GameRecord.Ins.UnlockRecords`** — unlocked record set.
*   **`RCG_GameRecord.UnlockedCards / UnlockedItems / UnlockedEquipments / UnlockedCharacters`** — shop purchase records.
*   **`RCG_GameAchievement.Ins.IsAchievementUnlocked`** — Achievement type unlock check.
*   **`RCG_GameRecord.DisabledUnlockRecord`** — debug-disabled unlocks.
*   **`RCG_DataService.Ins.m_GameData.m_UnlockedSystemUnlocks`** — system feature unlock record.
*   **`UCLI_Asset.GetUtilByType`** — reflective fetch of corresponding type's Asset Util.

### A.5 Known Issues

*   `// TODO: Add Deck QWQ234!!` — preview UI omits Deck / Character categories; enumeration incomplete.
*   `RCG_ShopUnlockData` / `CommonItemGenData` are same-file helpers used with RCG_UnlockData.
