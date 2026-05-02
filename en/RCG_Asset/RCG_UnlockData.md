---
title: 解鎖資料 (RCG_UnlockData) 說明
description: 通用解鎖條件設定（卡牌 / 裝備 / 道具 / 技能 / 系統功能）；含商店解鎖與 Steam 成就串接
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-en
---

> [!WARNING]
> Translation pending — this file needs an English translation.
The original zh-Hant content is included below for reference.


# 解鎖資料

> 程式類別名稱：`RCG_UnlockData`

## 用途

**通用的解鎖條件設定**。多個資料（卡牌 / 裝備 / 道具 / 技能 / 角色 / 大地圖 / 牌組 / 主動能力）可以引用同一個 `RCG_UnlockData` 共享解鎖條件——例如「達到 5 級時一次解鎖一組初級卡牌與裝備」只需建一個 UnlockData，相關資產 `m_Unlock.ID = "Lv5_Unlock"` 即可。

也支援「祝福商店解鎖」（解鎖後仍要花祝福購買）與「系統功能解鎖」（解鎖後啟用最終關 / 禁卡 / 篝火強化等）。

繼承自 `RCG_Asset<RCG_UnlockData>`。

## 編輯器中的樣貌

```
RCG_UnlockData: <ID>
    UnlockSetting       ← 解鎖條件本體（UnlockType: Level / SkillTagLevel / None / Tutorial / Achievement / Never）
    IsBlessingShop      ← 是否透過祝福商店解鎖
    IsHiddenInCodex     ← 圖鑑中隱藏
    SystemUnlockItems   ← 解鎖此 Asset 時順便啟用的系統功能（FinalLevel / BanCard / Inheritance...）
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **UnlockSetting** | 是 | 解鎖條件主設定 |
| **IsBlessingShop** | — | true：解鎖條件達成後仍要在祝福商店購買才生效 |
| **IsHiddenInCodex** | — | 圖鑑隱藏（不暴露未解鎖的物件名） |
| **SystemUnlockItems** | 否 | 一同啟用的系統功能清單（`SystemUnlockItem` enum） |

`UnlockSetting` 內含：

| 子欄位 | 說明 |
|---|---|
| **UnlockType** | `Level`（玩家等級）/ `SkillTagLevel`（職業等級）/ `None`（直接解鎖）/ `Tutorial`（通教學）/ `Achievement`（特定成就）/ `Never`（永不解鎖） |
| **SkillTag** | UnlockType=SkillTagLevel 時的職業 |
| **UnlockLevel** | UnlockType=Level/SkillTagLevel 時的等級門檻 |
| **Achievement** | UnlockType=Achievement 時的成就 ID |

## 行為說明

### `CheckUnlock()` (static)
掃所有 UnlockData：
1. 已在 `RCG_GameRecord.UnlockRecords` 內 → 跳過。
2. `UnlockType.None` → 直接記錄解鎖 + 套用 `SystemUnlockItems`，但不彈窗顯示。
3. 其他類型：`!IsLocked()` → 記錄 + 套用 SystemUnlockItems；非祝福商店物品則加入回傳 set（讓上層彈解鎖通知）。
回傳「本次新解鎖且非祝福商店的 ID 集合」，供 UI 顯示「新解鎖物品」。

### `GetUnloackables<T>(id)` (generic static)
查所有實作 `RCGI_Unloackable` 的 Asset 中 `UnlockEntry.ID == id` 的；用於本 UnlockData 對應到哪些卡牌 / 裝備 / 道具 / 技能 / 角色 / 大地圖 / 牌組 / 主動能力。

### `IsLocked()` (UnlockSetting)
*   `Level` → 玩家等級 < `m_UnlockLevel`。
*   `SkillTagLevel` → 對應職業等級 < `m_UnlockLevel`。
*   `None` → false（已解鎖）。
*   `Tutorial` → true（這層永遠回 true，實際解鎖判斷在 GameRecord 的 Tutorial 集合）。
*   `Achievement` → 該成就未解鎖。
*   `Never` → true。

### Editor 預覽
分四類顯示對應的 unlockable：CardData / ItemData / EquipmentData / UnitSkillData，可分頁切換。BigMap / Character 略過。

## 注意事項

*   **`Tutorial` UnlockType 在 IsLocked 永遠回 true**：實際解鎖路徑走 `RCG_GameRecord.Ins.UnlockedCharacters` 等記錄，不靠 `IsLocked`。
*   **`IsBlessingShop = true` 的解鎖**有兩階段：條件達成 → 解鎖商店上架；玩家購買 → 真正可用（記錄在 `RCG_GameRecord.UnlockedCards / UnlockedItems / UnlockedEquipments`）。
*   **`SystemUnlockItems` 是 enum 寫死**：FinalLevel / BanCard / Inheritance LV1/2 / RestPointEnhancement LV1/2 / Blessing_Shop LV1/2/3 / Secret_Base，新增功能要改 enum + 程式對應點。
*   **Disabled 解鎖**：`UnlockEditor` 提供把已解鎖的記錄「禁用」的開關，加入 `DisabledUnlockRecord` 集合（debug / 測試用）。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_UnlockData.cs`
*   **繼承自**：`RCG_Asset<RCG_UnlockData>`
*   **AssetGroup**：`EditGameSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_UnlockSetting` | UnlockSetting | `UnlockSetting` | 條件主設定 |
| `m_IsBlessingShop` | IsBlessingShop | `bool` | |
| `m_IsHiddenInCodex` | IsHiddenInCodex | `bool` | |
| `m_SystemUnlockItems` | SystemUnlockItems | `List<SystemUnlockItem>` | enum：10 種系統功能 |

### A.3 重要 Method

*   **`CheckUnlock()` (static)** — 掃描全 UnlockData 並更新解鎖紀錄。
*   **`GetUnloackables<T>(id)` (generic static)** — 對應引用此 UnlockData 的所有 Asset。
*   **`GetLockedCards / GetLockedEquipments / ... ` (instance)** — 各類型的快捷查詢。
*   **`UnlockSetting.IsLocked()`** — 條件判斷。
*   **`UnlockSetting.GetShortName()`** — 自動生成解鎖描述字串。
*   **`RCG_UnlockEntry.Unlocked / IsShopItem / Name / UnlockType`** — 引用此資料的 entry 的 helper。

### A.4 與其他系統的互動

*   **`RCG_GameRecord.Ins.UnlockRecords`** — 已解鎖紀錄集。
*   **`RCG_GameRecord.UnlockedCards / UnlockedItems / UnlockedEquipments / UnlockedCharacters`** — 商店購買紀錄。
*   **`RCG_GameAchievement.Ins.IsAchievementUnlocked`** — Achievement 類型解鎖檢查。
*   **`RCG_GameRecord.DisabledUnlockRecord`** — debug 禁用解鎖。
*   **`RCG_DataService.Ins.m_GameData.m_UnlockedSystemUnlocks`** — 系統功能解鎖紀錄。
*   **`UCLI_Asset.GetUtilByType`** — 反射取對應型別的 Asset Util。

### A.5 已知議題

*   `// TODO: Add Deck QWQ234!!` — 預覽 UI 漏了 Deck / Character 兩類，列舉不完整。
*   `RCG_ShopUnlockData` / `CommonItemGenData` 是同檔的輔助結構，與 RCG_UnlockData 配合使用。