---
title: RCG_BattlePresetData 說明
description: <!-- TODO: 一句話功能摘要 -->
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# RCG_BattlePresetData

> 程式類別名稱：`RCG_BattlePresetData`

## 用途

<!-- TODO: 描述這個 Asset 在遊戲裡負責什麼、什麼情境會用、舉 1-2 個範例。 -->

繼承自 `RCG_Asset<RCG_BattlePresetData>`。

## 編輯器中的樣貌

```
<!-- TODO: 描繪此 Asset 在編輯器內的版面 -->
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **BattleSetGenData** | — | <!-- TODO: 說明欄位用途 --> |
| **Players** | — | <!-- TODO: 說明欄位用途 --> |
| **ExtraCards** | — | <!-- TODO: 說明欄位用途 --> |
| **Items** | — | <!-- TODO: 說明欄位用途 --> |
| **Deck** | — | <!-- TODO: 說明欄位用途 --> |
| **Equipments** | — | <!-- TODO: 說明欄位用途 --> |
| **UnitSkills** | — | <!-- TODO: 說明欄位用途 --> |
| **AllEquipments** | — | <!-- TODO: 說明欄位用途 --> |
| **EnemyType** | — | <!-- TODO: 說明欄位用途 --> |
| **DifficultyData** | — | <!-- TODO: 說明欄位用途 --> |
| **Difficulty** | — | <!-- TODO: 說明欄位用途 --> |

## 行為說明

<!-- TODO: 戰鬥 / 載入 / 解鎖時的觸發時機與順序。 -->

## 注意事項

<!-- TODO: 常見的設計反模式 / 容易踩到的坑。 -->

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊

*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattlePresetData.cs`
*   **繼承自**：`RCG_Asset<RCG_BattlePresetData>`
*   **實作介面**：（無）

### A.2 欄位對照（自動產生，需人工複核）

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_BattleSetGenData` | BattleSetGenData | `RCG_BattleSetGenData` | `BattleSetGenData` | |
| `m_Players` | Players | `List<RCG_CharacterGenData>` | `Players` | |
| `m_ExtraCards` | ExtraCards | `List<RCG_CardGenData>` | `ExtraCards` | |
| `m_Items` | Items | `List<RCG_ItemGenData>` | `Items` | |
| `m_Deck` | Deck | `RCG_DeckGenData` | `Deck` | |
| `m_Equipments` | Equipments | `Dictionary<RCG_CharacterGenData, List<RCG_EquipmentGenData>>` | `Equipments` | |
| `m_UnitSkills` | UnitSkills | `Dictionary<RCG_CharacterGenData, List<RCG_UnitSkillGenData>>` | `UnitSkills` | |
| `m_AllEquipments` | AllEquipments | `List<RCG_EquipmentGenData>` | `AllEquipments` | |
| `m_EnemyType` | EnemyType | `RCG_EnemyTypeTagGenData` | `EnemyType` | |
| `m_DifficultyData` | DifficultyData | `RCG_DifficultyData` | `DifficultyData` | |
| `m_Difficulty` | Difficulty | `int` | `Difficulty` | |

### A.3 重要 Method 摘要

<!-- TODO: 補上影響行為的關鍵 method（OnGUI / Preview / 序列化覆寫等）。 -->

### A.4 與其他系統的互動

<!-- TODO: 列出依賴 / 被依賴的類別與系統。 -->

### A.5 已知議題（選填）

<!-- TODO: TODO/FIXME 摘錄、待重構點。 -->
