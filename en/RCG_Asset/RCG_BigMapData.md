---
title: 大地圖資料 (RCG_BigMapData) 說明
description: 一張完整大地圖的設定（章節骨架）：任務生成模式、狀態機、暗霧、解鎖、起始裝備、進度規則
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-en
---

> [!WARNING]
> Translation pending — this file needs an English translation.
The original zh-Hant content is included below for reference.


# 大地圖資料

> 程式類別名稱：`RCG_BigMapData`

## 用途

**一張完整大地圖的章節骨架設定**。每個 BigMap 是一段完整的遊戲歷程：從起點走到 Boss、中間踩各種任務節點、依任務 tag 推進進度。本資料定義：
*   任務生成模式（預設規則 / 任務池 / 池+混合）
*   多階段狀態機（`MapState`：每階段有自己的任務池、獎勵池、進入下階段條件）
*   暗霧設定、解鎖條件、起始裝備、可用挑戰目標
*   常駐 + 最終任務池

繼承自 `RCG_Asset<RCG_BigMapData>`，實作介面：`RCGI_Unloackable`。

## 編輯器中的樣貌

```
RCG_BigMapData: <ID>
    Name / Description / QuestIcon / BGM / BigMap (prefab)
    BigMapBackground
    TopMenuState
    QuestGenerateType         ← Default / QuestPool / QuestPoolMix
    MaxShowingQuestCount      ← 同時最多顯示多少個任務（Default 模式）
    QuestProgressToComplete   ← 解鎖最終關所需任務進度（0 = 忽略）
    QuestPoolSetting          ← (QuestPool/QuestPoolMix) 多階段狀態機
    PermanentQuests           ← (QuestPoolMix) 常駐任務
    FinalQuests               ← (QuestPoolMix) 最終任務
    QuestProgressTags         ← Boss 任務的 Tag 標籤（Forest/Mountain/Boss⋯）
    DetailSetting             ← 隱藏旗標 / 教學設定 / 解鎖 / 起始裝備等
```

## 主要欄位（節選）

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **QuestGenerateType** | 是 | `Default`（預設規則）/ `QuestPool`（純任務池）/ `QuestPoolMix`（池+常駐+最終混合） |
| **MaxShowingQuestCount** | Default 時 | 同時可選任務數（預設 3） |
| **QuestProgressToComplete** | — | 解鎖最終關所需任務進度數；> 0 才會顯示進度指標 |
| **QuestPoolSetting** | QuestPool/Mix | 多階段狀態機（每個 `MapState` 有自己的任務池 / 獎勵池 / 進入條件） |
| **PermanentQuests** | QuestPoolMix | 常駐任務（如休息點）：總是出現且**不計算顯示數量限制** |
| **FinalQuests** | QuestPoolMix | 最終任務（達成 Final 條件後才出現） |
| **QuestProgressTags** | Default/Mix | 哪些 Tag 算「進度任務」（通常是 Boss 標籤） |
| **DetailSetting** | 是 | 細節設定：HideThisBigMap / RandomQuestOrder / Difficulty / EnterBigMapTutorial / ShowMode / IsTutorial / BigMapType (Default/Loop) / Unlock / OverridingCharacters / StartingEquipments / AvailableChallenges 等 |

`MapState` 內含每階段資料：
*   `MaxShowingQuestCount` / `QuestDropSettings`（含條件式權重的任務池清單）
*   `EquipmentRewardPools / UnitSkillRewardPools / CardRewardPools / ItemRewardPools`：階段專屬獎勵池
*   `NextStateConditions`（OR）/ `FinalConditions`（OR）

## 行為說明

### `EnterBigMap(newLoop)`
新一局時重置進度與通過任務記錄。

### `GenerateQuests()`
依 `QuestGenerateType` 分流：
*   **Default**：不在這裡，由其他 manager 處理。
*   **QuestPool**：純抽當前 MapState 的任務池。
*   **QuestPoolMix**：複雜邏輯——
    1. 起始祝福事件（首次進入時）
    2. 上個任務若為常駐/特殊 → 生成常駐任務 + 跳過選項
    3. 上個任務為 Boss → 重新隨機生成（按 ProgressTag 篩選）
    4. 滿足最終條件 → 只顯示最終任務
    5. 一般情況：依環境 Tag 去重、Boss 任務 + 一般任務按比例混合

### `QuestStart(questData)`
進入任務時若 `CurState.CheckFinal()` 為 true → 標記為最終地圖（讓勝利後彈通關 UI）。

### `PassedQuest(questId, progress)`
通關任務時：
1. `m_PassedQuests` 加入此 ID。
2. 寫入 `MapStatePassQuestCount` / `PassQuestCount` Tag（GameTag 系統）。
3. 檢查 `CurState.CheckTransition()` → 進入下一個 MapState（並重置 MapStatePassQuestCount）。

### `QuestCompleteAsync()`
若是最終地圖 → 顯示 `RCG_VictoryUI`，由玩家選擇是否繼續（繼續 = 進入下一輪 Loop）。

## 注意事項

*   **`QuestGenerateType.Default`** 是舊版規則，新地圖建議用 `QuestPoolMix`。
*   **`PermanentQuests` 不計數量限制**：要做休息點 / 永久商店這種「總是顯示」的任務節點放這裡。
*   **`m_QuestProgressToComplete` 與 `FinalQuests`** 一起使用：未達進度時 FinalQuests 不會出現。
*   **`HideThisBigMap`** 真值不止來自 `m_HideThisBigMap`：還有「`OverrideAvailableGameModes` 不含當前 GameVersion」也會隱藏。Demo 版鎖部分章節用此機制。
*   **`ShowMode`** 控制教學模式下的顯示：`ShowIfEnableTutorial` / `ShowIfDisableTutorial` 對應教學狀態切換。
*   **`Tutorial` 寫死 ID**：`BigMap_Challenge_Card`，是教學專用大地圖。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_MapScripts/RCG_BigMapData.cs`
*   **繼承自**：`RCG_Asset<RCG_BigMapData>`
*   **實作介面**：`RCGI_Unloackable`
*   **AssetGroup**：`EditQuestSetting`
*   **常數**：`TutorialName = "BigMap_Challenge_Card"`

### A.2 欄位對照（節選）

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_Name` / `m_Description` | Name / Description | `RCG_LocalizeData` | |
| `m_QuestIcon` | QuestIcon | `RCG_SpriteData` | |
| `m_BGM` | BGM | `RCG_BGMGenData` | |
| `m_BigMap` | BigMap | `RCG_PrefabResData` | 大地圖 prefab |
| `m_BigMapBackground` | BigMapBackground | `RCG_SpriteData` | |
| `m_TopMenuState` | TopMenuState | `TopMenuState` enum | |
| `m_QuestGenerateType` | QuestGenerateType | `QuestGenerateType` enum | |
| `m_MaxShowingQuestCount` | MaxShowingQuestCount | `int` | `Conditional(Default)` |
| `m_QuestProgressToComplete` | QuestProgressToComplete | `int` | |
| `m_QuestPoolSetting` | QuestPoolSetting | `QuestPoolSetting`（巢狀） | `Conditional(QuestPool / QuestPoolMix)` |
| `m_PermanentQuests` / `m_FinalQuests` | 常駐 / 最終 | `MapState` | `Conditional(QuestPoolMix)` |
| `m_QuestProgressTags` | QuestProgressTags | `List<RCG_QuestData.QuestTag>` | |
| `m_DetailSetting` | DetailSetting | `DetailSetting`（巢狀） | |

### A.3 重要 Method 摘要

*   **`EnterBigMap(newLoop)`** — 進入大地圖；新局重置進度。
*   **`GenerateQuests()`** — 主任務生成入口。
*   **`GenerateQuestsQuestPool()`** / **`GenerateQuestsQuestPoolMix()`** — 兩套生成實作。
*   **`QuestStart(questData)`** — 任務開始；標記最終地圖旗標。
*   **`PassedQuest(questId, progress)`** — 通關時更新 Tag、進入下個 MapState。
*   **`QuestCompleteAsync(token, map)`** — 任務完成；最終地圖會彈通關 UI。
*   **`AutoTriggerQuests()`** — 起始祝福事件 / 自動進入單一任務。
*   **`CurState`** — `m_QuestPoolSetting.m_MapStates[saveData.m_State]`。
*   **`HideThisBigMap`** / **`ShowThisBigMap`** — 多重判斷可見性。
*   **`MapState.GetQuestDropPool`** — 按條件式權重隨機抽池。
*   **`MapState.CheckTransition` / `CheckFinal`** — 狀態切換 / 最終判定（OR）。

### A.4 與其他系統的互動

*   **`RCG_BigMapManager`** — runtime 主入口。
*   **`RCG_QuestData`** / **`RCG_QuestDropPool`** — 任務系統。
*   **`RCG_BigMapGenData`** — Asset Entry；預設 `BigMap_Challenge_Card`。
*   **`BigMapSaveData`** — runtime 進度儲存。
*   **`RCG_DifficultyData.m_AdditionalQuestProgress`** — 難度附加進度。
*   **`RCG_GameSettingData.m_GameVersion`** — Demo 版本鎖定判斷。
*   **`RCG_TutorialService`** — 教學模式可見性判斷。

### A.5 已知議題

*   `// QWQ23` 標記散落多處，標示舊版資料格式遷移與待確認邏輯。
*   `GenerateQuestsQuestPoolMix` 邏輯極長（300+ 行單一方法），多處註解掉的 LogError，維護困難。