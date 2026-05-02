---
title: RCG_DifficultyData 說明
description: <!-- TODO: 一句話功能摘要 -->
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# RCG_DifficultyData

> 程式類別名稱：`RCG_DifficultyData`

## 用途

<!-- TODO: 描述這個 Asset 在遊戲裡負責什麼、什麼情境會用、舉 1-2 個範例。 -->

繼承自 `RCG_Asset<RCG_DifficultyData>`。

## 編輯器中的樣貌

```
<!-- TODO: 描繪此 Asset 在編輯器內的版面 -->
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Name** | — | <!-- TODO: 說明欄位用途 --> |
| **IconSprite** | — | <!-- TODO: 說明欄位用途 --> |
| **IsHidden** | — | <!-- TODO: 說明欄位用途 --> |
| **BaseLevel** | — | <!-- TODO: 說明欄位用途 --> |
| **FieldEffects** | — | <!-- TODO: 說明欄位用途 --> |
| **HealthMult** | — | <!-- TODO: 說明欄位用途 --> |
| **AtkMult** | — | <!-- TODO: 說明欄位用途 --> |
| **PriceMult** | — | <!-- TODO: 說明欄位用途 --> |
| **SoulPriceMult** | — | <!-- TODO: 說明欄位用途 --> |
| **DarkMistMult** | — | <!-- TODO: 說明欄位用途 --> |
| **InitResources** | — | <!-- TODO: 說明欄位用途 --> |
| **CampFireDecreaseRate** | — | <!-- TODO: 說明欄位用途 --> |
| **CampFireHealPercentageMult** | — | <!-- TODO: 說明欄位用途 --> |
| **ItemUsageLimit** | — | <!-- TODO: 說明欄位用途 --> |
| **AdditionalLength** | — | <!-- TODO: 說明欄位用途 --> |
| **SellPriceMult** | — | <!-- TODO: 說明欄位用途 --> |
| **AdditionalQuestProgress** | — | <!-- TODO: 說明欄位用途 --> |
| **EnemySkillLevel** | — | <!-- TODO: 說明欄位用途 --> |
| **RequiredCompletedDifficulty** | — | <!-- TODO: 說明欄位用途 --> |
| **SortOrder** | — | <!-- TODO: 說明欄位用途 --> |

## 行為說明

<!-- TODO: 戰鬥 / 載入 / 解鎖時的觸發時機與順序。 -->

## 注意事項

<!-- TODO: 常見的設計反模式 / 容易踩到的坑。 -->

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊

*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_DifficultyData.cs`
*   **繼承自**：`RCG_Asset<RCG_DifficultyData>`
*   **實作介面**：（無）

### A.2 欄位對照（自動產生，需人工複核）

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | `Name` | |
| `m_IconSprite` | IconSprite | `RCG_SpriteData` | `IconSprite` | |
| `m_IsHidden` | IsHidden | `bool` | `IsHidden` | |
| `m_BaseLevel` | BaseLevel | `int` | `BaseLevel` | |
| `m_FieldEffects` | FieldEffects | `List<RCG_FieldEffectGenData>` | `FieldEffects` | |
| `m_HealthMult` | HealthMult | `float` | `HealthMult` | |
| `m_AtkMult` | AtkMult | `float` | `AtkMult` | |
| `m_PriceMult` | PriceMult | `float` | `PriceMult` | |
| `m_SoulPriceMult` | SoulPriceMult | `float` | `SoulPriceMult` | |
| `m_DarkMistMult` | DarkMistMult | `float` | `DarkMistMult` | |
| `m_InitResources` | InitResources | `List<RCG_ResourceGenData>` | `InitResources` | |
| `m_CampFireDecreaseRate` | CampFireDecreaseRate | `float` | `CampFireDecreaseRate` | |
| `m_CampFireHealPercentageMult` | CampFireHealPercentageMult | `float` | `CampFireHealPercentageMult` | |
| `m_ItemUsageLimit` | ItemUsageLimit | `int` | `ItemUsageLimit` | |
| `m_AdditionalLength` | AdditionalLength | `float` | `AdditionalLength` | |
| `m_SellPriceMult` | SellPriceMult | `float` | `SellPriceMult` | |
| `m_AdditionalQuestProgress` | AdditionalQuestProgress | `int` | `AdditionalQuestProgress` | |
| `m_EnemySkillLevel` | EnemySkillLevel | `int` | `EnemySkillLevel` | |
| `m_RequiredCompletedDifficulty` | RequiredCompletedDifficulty | `List<RCG_DifficultyGenData>` | `RequiredCompletedDifficulty` | |
| `m_SortOrder` | SortOrder | `int` | `SortOrder` | |

### A.3 重要 Method 摘要

<!-- TODO: 補上影響行為的關鍵 method（OnGUI / Preview / 序列化覆寫等）。 -->

### A.4 與其他系統的互動

<!-- TODO: 列出依賴 / 被依賴的類別與系統。 -->

### A.5 已知議題（選填）

<!-- TODO: TODO/FIXME 摘錄、待重構點。 -->
