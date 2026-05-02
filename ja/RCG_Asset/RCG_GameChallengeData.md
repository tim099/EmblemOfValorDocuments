---
title: 遊戲挑戰 (RCG_GameChallengeData) 說明
description: 全局遊戲挑戰目標：完成此通關目標才視為「達成挑戰」（與 Achievement 不同）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-ja
---

> [!WARNING]
> 翻訳待機中 — このファイルは日本語翻訳が必要です。
参考用に zh-Hant 原文を以下に掲載しています。


# 遊戲挑戰

> 程式類別名稱：`RCG_GameChallengeData`

## 用途

**全局的「通關挑戰目標」設定**。例如「擊敗最終 Boss」「不使用任何道具通關」「3 回合內結束戰鬥」。挑戰是「**通關目標**」級別，與單純獲得的 `RCG_AchievementAsset` 不同——挑戰決定「這場遊戲是否已通關」。

繼承自 `RCG_Asset<RCG_GameChallengeData>`，實作介面：`RCGI_Unloackable`。

## 編輯器中的樣貌

```
RCG_GameChallengeData: <ID>
    Name             ← 挑戰名（多語系；預設 "GameChallenge"）
    Unlock           ← 解鎖條件
    ChallengeGoals   ← 完成條件（多個 RCG_QuestGoalData 共同構成）
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Name** | 是 | 挑戰顯示名（多語系） |
| **Unlock** | 否 | 解鎖條件；空 = 預設已解鎖 |
| **ChallengeGoals** | 是 | 完成條件清單（`RCG_QuestGoalData`），達成全部視為通關 |

## 行為說明

### 解鎖判斷 (`IsUnlocked`)
*   `m_Unlock.IsEmpty || m_Unlock.Unlocked`：沒設條件或條件已通過 → 已解鎖。

### 完成判斷
本檔本身不負責「是否完成」的判斷；交由外部的 quest manager 或 challenge manager 檢查 `m_ChallengeGoals` 的進度。

## 注意事項

*   **預設 ID `Challenge_FinalBoss`** 是「擊敗最終 Boss」的標準挑戰，多數路線通關都用這個。
*   **與 `RCG_AchievementAsset` 的差異**：成就是「附加榮譽」（解鎖條件達成 → 獲得獎勵 / 顯示）；挑戰是「**通關判定**」（達成 → 該局結束 / 觸發結算）。
*   **`m_Name` 預設值是 `"GameChallenge"`**：必須改名才會在 UI 顯示有意義的標題。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_GameChallengeData.cs`
*   **繼承自**：`RCG_Asset<RCG_GameChallengeData>`
*   **實作介面**：`RCGI_Unloackable`
*   **AssetGroup**：`EditGameSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | 預設 `"GameChallenge"` |
| `m_Unlock` | Unlock | `RCG_UnlockEntry` | |
| `m_ChallengeGoals` | ChallengeGoals | `List<RCG_QuestGoalData>` | |

### A.3 重要 Method

*   **`IsUnlocked` (property)** — `m_Unlock.IsEmpty || m_Unlock.Unlocked`。
*   **`UnlockEntry`** — `m_Unlock`。
*   **`LocalizedName`** — `m_Name.Name`。

### A.4 與其他系統的互動

*   **`RCG_QuestGoalData`** — 通關條件元素。
*   **`RCG_GameChallengeGenData`** — Asset Entry 包裝；預設 `Challenge_FinalBoss`。
*   **`RCG_UnlockEntry`** — 解鎖系統。

### A.5 已知議題

*   無 `Preview` 實作；用基底類預設繪製。