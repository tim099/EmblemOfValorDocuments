---
title: RCG_GameChallengeData 說明
description: <!-- TODO: 一句話功能摘要 -->
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# RCG_GameChallengeData

> 程式類別名稱：`RCG_GameChallengeData`

## 用途

<!-- TODO: 描述這個 Asset 在遊戲裡負責什麼、什麼情境會用、舉 1-2 個範例。 -->

繼承自 `RCG_Asset<RCG_GameChallengeData>`，實作介面：`RCGI_Unloackable`

## 編輯器中的樣貌

```
<!-- TODO: 描繪此 Asset 在編輯器內的版面 -->
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Name** | — | <!-- TODO: 說明欄位用途 --> |
| **Unlock** | — | <!-- TODO: 說明欄位用途 --> |
| **ChallengeGoals** | — | <!-- TODO: 說明欄位用途 --> |

## 行為說明

<!-- TODO: 戰鬥 / 載入 / 解鎖時的觸發時機與順序。 -->

## 注意事項

<!-- TODO: 常見的設計反模式 / 容易踩到的坑。 -->

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊

*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_GameChallengeData.cs`
*   **繼承自**：`RCG_Asset<RCG_GameChallengeData>, RCGI_Unloackable`
*   **實作介面**：`RCGI_Unloackable`

### A.2 欄位對照（自動產生，需人工複核）

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | `Name` | |
| `m_Unlock` | Unlock | `RCG_UnlockEntry` | `Unlock` | |
| `m_ChallengeGoals` | ChallengeGoals | `List<RCG_QuestGoalData>` | `ChallengeGoals` | |

### A.3 重要 Method 摘要

<!-- TODO: 補上影響行為的關鍵 method（OnGUI / Preview / 序列化覆寫等）。 -->

### A.4 與其他系統的互動

<!-- TODO: 列出依賴 / 被依賴的類別與系統。 -->

### A.5 已知議題（選填）

<!-- TODO: TODO/FIXME 摘錄、待重構點。 -->
