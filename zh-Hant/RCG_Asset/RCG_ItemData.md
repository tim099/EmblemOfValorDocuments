---
title: RCG_ItemData 說明
description: <!-- TODO: 一句話功能摘要 -->
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# RCG_ItemData

> 程式類別名稱：`RCG_ItemData`

## 用途

<!-- TODO: 描述這個 Asset 在遊戲裡負責什麼、什麼情境會用、舉 1-2 個範例。 -->

繼承自 `RCG_Asset<RCG_ItemData>`，實作介面：`RCGI_Item`, `RCGI_Unloackable`

## 編輯器中的樣貌

```
<!-- TODO: 描繪此 Asset 在編輯器內的版面 -->
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Name** | — | <!-- TODO: 說明欄位用途 --> |
| **Description** | — | <!-- TODO: 說明欄位用途 --> |
| **TargetType** | — | <!-- TODO: 說明欄位用途 --> |
| **Price** | — | <!-- TODO: 說明欄位用途 --> |
| **Icon** | — | <!-- TODO: 說明欄位用途 --> |
| **Rarity** | — | <!-- TODO: 說明欄位用途 --> |
| **ItemType** | — | <!-- TODO: 說明欄位用途 --> |
| **ItemUseType** | — | <!-- TODO: 說明欄位用途 --> |
| **Tags** | — | <!-- TODO: 說明欄位用途 --> |
| **UseTimes** | — | <!-- TODO: 說明欄位用途 --> |
| **Unlock** | — | <!-- TODO: 說明欄位用途 --> |
| **HideInCodex** | — | <!-- TODO: 說明欄位用途 --> |
| **Data** | — | <!-- TODO: 說明欄位用途 --> |
| **ItemEffects** | — | <!-- TODO: 說明欄位用途 --> |

## 行為說明

<!-- TODO: 戰鬥 / 載入 / 解鎖時的觸發時機與順序。 -->

## 注意事項

<!-- TODO: 常見的設計反模式 / 容易踩到的坑。 -->

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊

*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_ItemData.cs`
*   **繼承自**：`RCG_Asset<RCG_ItemData>, RCGI_Item, RCGI_Unloackable`
*   **實作介面**：`RCGI_Item`, `RCGI_Unloackable`

### A.2 欄位對照（自動產生，需人工複核）

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | `Name` | |
| `m_Description` | Description | `RCG_LocalizeData` | `Description` | |
| `m_TargetType` | TargetType | `TargetType` | `TargetType` | |
| `m_Price` | Price | `int` | `Price` | |
| `m_Icon` | Icon | `RCG_SpriteData` | `Icon` | |
| `m_Rarity` | Rarity | `RCG_RarityTagGenData` | `Rarity` | |
| `m_ItemType` | ItemType | `ItemType` | `ItemType` | |
| `m_ItemUseType` | ItemUseType | `ItemUseType` | `ItemUseType` | |
| `m_Tags` | Tags | `List<RCG_ItemTagGenData>` | `Tags` | |
| `m_UseTimes` | UseTimes | `int` | `UseTimes` | UCL.Core.PA.Conditional("m_ItemUseType", false, ItemUseType.Consume, ItemUseType.OncePerTurn, ItemUseType.OncePerBattle) |
| `m_Unlock` | Unlock | `RCG_UnlockEntry` | `Unlock` | |
| `m_HideInCodex` | HideInCodex | `bool` | `HideInCodex` | |
| `m_Data` | Data | `ItemData` | `Data` | |
| `m_ItemEffects` | ItemEffects | `List<RCG_CommonEffect>` | `ItemEffects` | |

### A.3 重要 Method 摘要

<!-- TODO: 補上影響行為的關鍵 method（OnGUI / Preview / 序列化覆寫等）。 -->

### A.4 與其他系統的互動

<!-- TODO: 列出依賴 / 被依賴的類別與系統。 -->

### A.5 已知議題（選填）

<!-- TODO: TODO/FIXME 摘錄、待重構點。 -->
