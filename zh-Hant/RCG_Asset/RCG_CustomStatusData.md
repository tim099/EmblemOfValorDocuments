---
title: RCG_CustomStatusData 說明
description: <!-- TODO: 一句話功能摘要 -->
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# RCG_CustomStatusData

> 程式類別名稱：`RCG_CustomStatusData`

## 用途

<!-- TODO: 描述這個 Asset 在遊戲裡負責什麼、什麼情境會用、舉 1-2 個範例。 -->

繼承自 `RCG_Asset<RCG_CustomStatusData>`，實作介面：`RCGI_Status`

## 編輯器中的樣貌

```
<!-- TODO: 描繪此 Asset 在編輯器內的版面 -->
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Name** | — | <!-- TODO: 說明欄位用途 --> |
| **StatusEffectType** | — | <!-- TODO: 說明欄位用途 --> |
| **StatusClass** | — | <!-- TODO: 說明欄位用途 --> |
| **AtkBuff** | — | <!-- TODO: 說明欄位用途 --> |
| **DefBuff** | — | <!-- TODO: 說明欄位用途 --> |
| **IconSprite** | — | <!-- TODO: 說明欄位用途 --> |
| **Tags** | — | <!-- TODO: 說明欄位用途 --> |
| **LayerIncreaseVFX** | — | <!-- TODO: 說明欄位用途 --> |
| **StatusPersistantVFX** | — | <!-- TODO: 說明欄位用途 --> |
| **StatusOffsets** | — | <!-- TODO: 說明欄位用途 --> |
| **OffsetTags** | — | <!-- TODO: 說明欄位用途 --> |
| **StatusAlterOn** | — | <!-- TODO: 說明欄位用途 --> |
| **Effects** | — | <!-- TODO: 說明欄位用途 --> |
| **UnitStates** | — | <!-- TODO: 說明欄位用途 --> |
| **StopDecayStatus** | — | <!-- TODO: 說明欄位用途 --> |
| **ImmuneStatus** | — | <!-- TODO: 說明欄位用途 --> |
| **Resistance** | — | <!-- TODO: 說明欄位用途 --> |

## 行為說明

<!-- TODO: 戰鬥 / 載入 / 解鎖時的觸發時機與順序。 -->

## 注意事項

<!-- TODO: 常見的設計反模式 / 容易踩到的坑。 -->

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊

*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_CustomStatusData.cs`
*   **繼承自**：`RCG_Asset<RCG_CustomStatusData>, RCGI_Status`
*   **實作介面**：`RCGI_Status`

### A.2 欄位對照（自動產生，需人工複核）

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | `Name` | |
| `m_StatusEffectType` | StatusEffectType | `StatusEffectType` | `StatusEffectType` | |
| `m_StatusClass` | StatusClass | `StatusClass` | `StatusClass` | |
| `m_AtkBuff` | AtkBuff | `List<RCG_AtkBuffData>` | `AtkBuff` | |
| `m_DefBuff` | DefBuff | `List<RCG_DefBuffData>` | `DefBuff` | |
| `m_IconSprite` | IconSprite | `RCG_IconSpriteGenData` | `IconSprite` | |
| `m_Tags` | Tags | `List<RCG_StatusTagGenData>` | `Tags` | |
| `m_LayerIncreaseVFX` | LayerIncreaseVFX | `RCG_CommonVFXGenData` | `LayerIncreaseVFX` | |
| `m_StatusPersistantVFX` | StatusPersistantVFX | `RCG_CommonVFXGenData` | `StatusPersistantVFX` | |
| `m_StatusOffsets` | StatusOffsets | `List<RCG_CustomStatusGenData>` | `StatusOffsets` | |
| `m_OffsetTags` | OffsetTags | `List<RCG_StatusTagGenData>` | `OffsetTags` | |
| `m_StatusAlterOn` | StatusAlterOn | `Dictionary<RCG_EffectTriggerOn, eStatusDecreaseType>` | `StatusAlterOn` | |
| `m_Effects` | Effects | `List<RCG_CommonEffect>` | `Effects` | |
| `m_UnitStates` | UnitStates | `List<UnitState>` | `UnitStates` | |
| `m_StopDecayStatus` | StopDecayStatus | `List<RCG_CustomStatusGenData>` | `StopDecayStatus` | |
| `m_ImmuneStatus` | ImmuneStatus | `List<RCG_CustomStatusGenData>` | `ImmuneStatus` | |
| `m_Resistance` | Resistance | `List<RCG_CustomStatusGenData>` | `Resistance` | |

### A.3 重要 Method 摘要

<!-- TODO: 補上影響行為的關鍵 method（OnGUI / Preview / 序列化覆寫等）。 -->

### A.4 與其他系統的互動

<!-- TODO: 列出依賴 / 被依賴的類別與系統。 -->

### A.5 已知議題（選填）

<!-- TODO: TODO/FIXME 摘錄、待重構點。 -->
