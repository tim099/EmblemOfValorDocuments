---
title: RCG_UnlockData 說明
description: <!-- TODO: 一句話功能摘要 -->
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# RCG_UnlockData

> 程式類別名稱：`RCG_UnlockData`

## 用途

<!-- TODO: 描述這個 Asset 在遊戲裡負責什麼、什麼情境會用、舉 1-2 個範例。 -->

繼承自 `RCG_Asset<RCG_UnlockData>`。

## 編輯器中的樣貌

```
<!-- TODO: 描繪此 Asset 在編輯器內的版面 -->
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Price** | — | <!-- TODO: 說明欄位用途 --> |
| **Item** | — | <!-- TODO: 說明欄位用途 --> |
| **UnlockSetting** | — | <!-- TODO: 說明欄位用途 --> |
| **IsBlessingShop** | — | <!-- TODO: 說明欄位用途 --> |
| **IsHiddenInCodex** | — | <!-- TODO: 說明欄位用途 --> |
| **SystemUnlockItems** | — | <!-- TODO: 說明欄位用途 --> |
| **Price** | — | <!-- TODO: 說明欄位用途 --> |
| **UnlockData** | — | <!-- TODO: 說明欄位用途 --> |
| **UnlockEditor** | — | <!-- TODO: 說明欄位用途 --> |
| **Dic** | — | <!-- TODO: 說明欄位用途 --> |
| **UnlockType** | — | <!-- TODO: 說明欄位用途 --> |
| **SkillTag** | — | <!-- TODO: 說明欄位用途 --> |
| **UnlockLevel** | — | <!-- TODO: 說明欄位用途 --> |
| **Achievement** | — | <!-- TODO: 說明欄位用途 --> |
| **ItemType** | — | <!-- TODO: 說明欄位用途 --> |
| **Card** | — | <!-- TODO: 說明欄位用途 --> |
| **Item** | — | <!-- TODO: 說明欄位用途 --> |
| **Equipment** | — | <!-- TODO: 說明欄位用途 --> |
| **Character** | — | <!-- TODO: 說明欄位用途 --> |

## 行為說明

<!-- TODO: 戰鬥 / 載入 / 解鎖時的觸發時機與順序。 -->

## 注意事項

<!-- TODO: 常見的設計反模式 / 容易踩到的坑。 -->

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊

*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_UnlockData.cs`
*   **繼承自**：`RCG_Asset<RCG_UnlockData>`
*   **實作介面**：（無）

### A.2 欄位對照（自動產生，需人工複核）

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_Price` | Price | `int` | `Price` | |
| `m_Item` | Item | `CommonItemGenData` | `Item` | |
| `m_UnlockSetting` | UnlockSetting | `UnlockSetting` | `UnlockSetting` | |
| `m_IsBlessingShop` | IsBlessingShop | `bool` | `IsBlessingShop` | |
| `m_IsHiddenInCodex` | IsHiddenInCodex | `bool` | `IsHiddenInCodex` | |
| `m_SystemUnlockItems` | SystemUnlockItems | `List<SystemUnlockItem>` | `SystemUnlockItems` | |
| `m_Price` | Price | `int` | `Price` | UCL.Core.PA.Conditional(nameof(IsShopItem)) |
| `m_UnlockData` | UnlockData | `UnlockData` | `UnlockData` | |
| `m_UnlockEditor` | UnlockEditor | `UnlockEditor` | `UnlockEditor` | |
| `m_Dic` | Dic | `UCL_ObjectDictionary` | `Dic` | |
| `m_UnlockType` | UnlockType | `UnlockType` | `UnlockType` | |
| `m_SkillTag` | SkillTag | `RCG_SkillTagGenData` | `SkillTag` | UCL.Core.PA.Conditional(nameof(m_UnlockType), false, UnlockType.SkillTagLevel) |
| `m_UnlockLevel` | UnlockLevel | `int` | `UnlockLevel` | UCL.Core.PA.Conditional(nameof(m_UnlockType), false, UnlockType.Level, UnlockType.SkillTagLevel) |
| `m_Achievement` | Achievement | `RCG_AchievementEntry` | `Achievement` | UCL.Core.PA.Conditional(nameof(m_UnlockType), false, UnlockType.Achievement) |
| `m_ItemType` | ItemType | `ItemType` | `ItemType` | |
| `m_Card` | Card | `RCG_CardGenData` | `Card` | UCL.Core.PA.Conditional(nameof(m_ItemType), false, ItemType.Card) |
| `m_Item` | Item | `RCG_ItemGenData` | `Item` | UCL.Core.PA.Conditional(nameof(m_ItemType), false, ItemType.Item) |
| `m_Equipment` | Equipment | `RCG_EquipmentGenData` | `Equipment` | UCL.Core.PA.Conditional(nameof(m_ItemType), false, ItemType.Equipment) |
| `m_Character` | Character | `RCG_CharacterGenData` | `Character` | UCL.Core.PA.Conditional(nameof(m_ItemType), false, ItemType.Character) |

### A.3 重要 Method 摘要

<!-- TODO: 補上影響行為的關鍵 method（OnGUI / Preview / 序列化覆寫等）。 -->

### A.4 與其他系統的互動

<!-- TODO: 列出依賴 / 被依賴的類別與系統。 -->

### A.5 已知議題（選填）

<!-- TODO: TODO/FIXME 摘錄、待重構點。 -->
