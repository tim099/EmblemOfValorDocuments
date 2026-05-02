---
title: RCG_AchievementAsset 說明
description: <!-- TODO: 一句話功能摘要 -->
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# RCG_AchievementAsset

> 程式類別名稱：`RCG_AchievementAsset`

## 用途

<!-- TODO: 描述這個 Asset 在遊戲裡負責什麼、什麼情境會用、舉 1-2 個範例。 -->

繼承自 `RCG_Asset<RCG_AchievementAsset>`。

## 編輯器中的樣貌

```
<!-- TODO: 描繪此 Asset 在編輯器內的版面 -->
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **IsClassAchievement** | — | <!-- TODO: 說明欄位用途 --> |
| **SkillTag** | — | <!-- TODO: 說明欄位用途 --> |
| **IsCharacterAchievement** | — | <!-- TODO: 說明欄位用途 --> |
| **Character** | — | <!-- TODO: 說明欄位用途 --> |
| **OverrideDescription** | — | <!-- TODO: 說明欄位用途 --> |
| **OrderIndex** | — | <!-- TODO: 說明欄位用途 --> |
| **HasSteamAchievement** | — | <!-- TODO: 說明欄位用途 --> |
| **SteamAchievement** | — | <!-- TODO: 說明欄位用途 --> |
| **Conditions** | — | <!-- TODO: 說明欄位用途 --> |
| **HasPrerequisiteAchievement** | — | <!-- TODO: 說明欄位用途 --> |
| **PrerequisiteAchievement** | — | <!-- TODO: 說明欄位用途 --> |

## 行為說明

<!-- TODO: 戰鬥 / 載入 / 解鎖時的觸發時機與順序。 -->

## 注意事項

<!-- TODO: 常見的設計反模式 / 容易踩到的坑。 -->

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊

*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_AchievementAsset.cs`
*   **繼承自**：`RCG_Asset<RCG_AchievementAsset>`
*   **實作介面**：（無）

### A.2 欄位對照（自動產生，需人工複核）

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_IsClassAchievement` | IsClassAchievement | `bool` | `IsClassAchievement` | |
| `m_SkillTag` | SkillTag | `RCG_SkillTagGenData` | `SkillTag` | UCL.Core.PA.Conditional(nameof(m_IsClassAchievement), false, true) |
| `m_IsCharacterAchievement` | IsCharacterAchievement | `bool` | `IsCharacterAchievement` | |
| `m_Character` | Character | `RCG_CharacterGenData` | `Character` | UCL.Core.PA.Conditional(nameof(m_IsCharacterAchievement), false, true) |
| `m_OverrideDescription` | OverrideDescription | `RCG_LocalizeData` | `OverrideDescription` | |
| `m_OrderIndex` | OrderIndex | `int` | `OrderIndex` | |
| `m_HasSteamAchievement` | HasSteamAchievement | `bool` | `HasSteamAchievement` | |
| `m_SteamAchievement` | SteamAchievement | `UCL_SteamAchievementEntry` | `SteamAchievement` | UCL.Core.PA.Conditional(nameof(m_HasSteamAchievement), false, true) |
| `m_Conditions` | Conditions | `List<RCG_Condition>` | `Conditions` | |
| `m_HasPrerequisiteAchievement` | HasPrerequisiteAchievement | `bool` | `HasPrerequisiteAchievement` | |
| `m_PrerequisiteAchievement` | PrerequisiteAchievement | `RCG_AchievementEntry` | `PrerequisiteAchievement` | UCL.Core.PA.Conditional(nameof(m_HasPrerequisiteAchievement), false, true) |

### A.3 重要 Method 摘要

<!-- TODO: 補上影響行為的關鍵 method（OnGUI / Preview / 序列化覆寫等）。 -->

### A.4 與其他系統的互動

<!-- TODO: 列出依賴 / 被依賴的類別與系統。 -->

### A.5 已知議題（選填）

<!-- TODO: TODO/FIXME 摘錄、待重構點。 -->
