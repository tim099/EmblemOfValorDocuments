---
title: RCG_CharacterData 說明
description: <!-- TODO: 一句話功能摘要 -->
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# RCG_CharacterData

> 程式類別名稱：`RCG_CharacterData`

## 用途

<!-- TODO: 描述這個 Asset 在遊戲裡負責什麼、什麼情境會用、舉 1-2 個範例。 -->

繼承自 `RCG_Asset<RCG_CharacterData>`，實作介面：`UCLI_ShortName`, `RCGI_Unloackable`

## 編輯器中的樣貌

```
<!-- TODO: 描繪此 Asset 在編輯器內的版面 -->
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Name** | — | <!-- TODO: 說明欄位用途 --> |
| **CharacterIntro** | — | <!-- TODO: 說明欄位用途 --> |
| **PortraitAnim** | — | <!-- TODO: 說明欄位用途 --> |
| **Portrait** | — | <!-- TODO: 說明欄位用途 --> |
| **Avatar** | — | <!-- TODO: 說明欄位用途 --> |
| **Illustrator** | — | <!-- TODO: 說明欄位用途 --> |
| **MaxHp** | — | <!-- TODO: 說明欄位用途 --> |
| **TutorialDeck** | — | <!-- TODO: 說明欄位用途 --> |
| **Deck** | — | <!-- TODO: 說明欄位用途 --> |
| **JoinDeck** | — | <!-- TODO: 說明欄位用途 --> |
| **UnitGenData** | — | <!-- TODO: 說明欄位用途 --> |
| **SkillTags** | — | <!-- TODO: 說明欄位用途 --> |
| **InitActivePowers** | — | <!-- TODO: 說明欄位用途 --> |
| **UnitSkillDatas** | — | <!-- TODO: 說明欄位用途 --> |
| **UnitSkillPool** | — | <!-- TODO: 說明欄位用途 --> |
| **InitItems** | — | <!-- TODO: 說明欄位用途 --> |
| **AdditionalDecks** | — | <!-- TODO: 說明欄位用途 --> |
| **AdditionalJoinDecks** | — | <!-- TODO: 說明欄位用途 --> |
| **Unlock** | — | <!-- TODO: 說明欄位用途 --> |
| **Order** | — | <!-- TODO: 說明欄位用途 --> |
| **Index** | — | <!-- TODO: 說明欄位用途 --> |
| **Hp** | — | <!-- TODO: 說明欄位用途 --> |
| **IsMainCharacter** | — | <!-- TODO: 說明欄位用途 --> |
| **SkillTags** | — | <!-- TODO: 說明欄位用途 --> |
| **UnitSkillEntities** | — | <!-- TODO: 說明欄位用途 --> |
| **Equipments** | — | <!-- TODO: 說明欄位用途 --> |
| **Data** | — | <!-- TODO: 說明欄位用途 --> |
| **RuntimeData** | — | <!-- TODO: 說明欄位用途 --> |
| **PortraitAnim** | — | <!-- TODO: 說明欄位用途 --> |
| **Equipments** | — | <!-- TODO: 說明欄位用途 --> |

## 行為說明

<!-- TODO: 戰鬥 / 載入 / 解鎖時的觸發時機與順序。 -->

## 注意事項

<!-- TODO: 常見的設計反模式 / 容易踩到的坑。 -->

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊

*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_CharacterData.cs`
*   **繼承自**：`RCG_Asset<RCG_CharacterData>, UCLI_ShortName, RCGI_Unloackable`
*   **實作介面**：`UCLI_ShortName`, `RCGI_Unloackable`

### A.2 欄位對照（自動產生，需人工複核）

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | `Name` | |
| `m_CharacterIntro` | CharacterIntro | `RCG_LocalizeData` | `CharacterIntro` | |
| `m_PortraitAnim` | PortraitAnim | `RCG_PrefabResData` | `PortraitAnim` | |
| `m_Portrait` | Portrait | `RCG_SpriteData` | `Portrait` | |
| `m_Avatar` | Avatar | `RCG_SpriteData` | `Avatar` | |
| `m_Illustrator` | Illustrator | `string` | `Illustrator` | |
| `m_MaxHp` | MaxHp | `int` | `MaxHp` | |
| `m_TutorialDeck` | TutorialDeck | `RCG_DeckGenData` | `TutorialDeck` | |
| `m_Deck` | Deck | `RCG_DeckGenData` | `Deck` | |
| `m_JoinDeck` | JoinDeck | `RCG_DeckGenData` | `JoinDeck` | |
| `m_UnitGenData` | UnitGenData | `RCG_UnitGenDataWithPosition` | `UnitGenData` | |
| `m_SkillTags` | SkillTags | `List<RCG_SkillTagGenData>` | `SkillTags` | |
| `m_InitActivePowers` | InitActivePowers | `List<RCG_ActivePowerGenData>` | `InitActivePowers` | |
| `m_UnitSkillDatas` | UnitSkillDatas | `List<RCG_UnitSkillGenData>` | `UnitSkillDatas` | |
| `m_UnitSkillPool` | UnitSkillPool | `RCG_UnitSkillDropSetting` | `UnitSkillPool` | |
| `m_InitItems` | InitItems | `List<RCG_ItemGenData>` | `InitItems` | |
| `m_AdditionalDecks` | AdditionalDecks | `List<RCG_DeckGenData>` | `AdditionalDecks` | |
| `m_AdditionalJoinDecks` | AdditionalJoinDecks | `List<RCG_DeckGenData>` | `AdditionalJoinDecks` | |
| `m_Unlock` | Unlock | `RCG_UnlockEntry` | `Unlock` | |
| `m_Order` | Order | `int` | `Order` | |
| `m_Index` | Index | `int` | `Index` | |
| `m_Hp` | Hp | `int` | `Hp` | |
| `m_IsMainCharacter` | IsMainCharacter | `bool` | `IsMainCharacter` | |
| `m_SkillTags` | SkillTags | `List<RCG_SkillTagGenData>` | `SkillTags` | |
| `m_UnitSkillEntities` | UnitSkillEntities | `List<RCG_UnitSkillPointer>` | `UnitSkillEntities` | |
| `m_Equipments` | Equipments | `Dictionary<EquipmentType, RCG_EquipmentPointer>` | `Equipments` | |
| `m_Data` | Data | `UnitData` | `Data` | |
| `m_RuntimeData` | RuntimeData | `UnitRuntimData` | `RuntimeData` | |
| `m_PortraitAnim` | PortraitAnim | `GameObject` | `PortraitAnim` | |
| `m_Equipments` | Equipments | `Dictionary<EquipmentType, RCG_EquipmentPointer>` | `Equipments` | |

### A.3 重要 Method 摘要

<!-- TODO: 補上影響行為的關鍵 method（OnGUI / Preview / 序列化覆寫等）。 -->

### A.4 與其他系統的互動

<!-- TODO: 列出依賴 / 被依賴的類別與系統。 -->

### A.5 已知議題（選填）

<!-- TODO: TODO/FIXME 摘錄、待重構點。 -->
