---
title: 難度資料 (RCG_DifficultyData) 說明
description: 難度等級的全局倍率設定：HP / 攻擊 / 商店價 / 暗霧 / 場地效果 / 解鎖前置難度
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-en
---

> [!WARNING]
> Translation pending — this file needs an English translation.
The original zh-Hant content is included below for reference.


# 難度資料

> 程式類別名稱：`RCG_DifficultyData`

## 用途

**難度等級的設定模板**——簡單 / 普通 / 困難 / 噩夢 / 教學等都是不同 ID 的 `RCG_DifficultyData`。本資料決定該難度下：
*   HP / 攻擊 / 商品價的倍率
*   敵人技能基礎等級加成
*   起始資源
*   篝火減少率
*   道具使用次數限制
*   永久套用的場地效果
*   進入此難度的前置條件（須完成哪些難度）

繼承自 `RCG_Asset<RCG_DifficultyData>`。

## 編輯器中的樣貌

```
RCG_DifficultyData: <ID>
    Name / IconSprite / IsHidden / SortOrder
    BaseLevel              ← 怪物基礎等級（再經 UnitLevelData 曲線換算）
    HealthMult / AtkMult / PriceMult / SoulPriceMult / DarkMistMult
    SellPriceMult          ← 賣物品/裝備時的價格倍率
    InitResources          ← 起始額外資源
    CampFireDecreaseRate / CampFireHealPercentageMult
    ItemUsageLimit         ← 道具使用次數限制
    AdditionalLength / AdditionalQuestProgress
    EnemySkillLevel        ← 敵人基礎技能等級
    FieldEffects           ← 永久場地效果
    RequiredCompletedDifficulty  ← 前置條件（OR：完成任一即可解鎖此難度）
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Name / IconSprite** | 是 | 顯示名與圖示 |
| **IsHidden** | — | 不在難度選單顯示（測試用） |
| **SortOrder** | 是 | 選單排序 |
| **BaseLevel** | 是 | 怪物基礎等級（會交給 `RCG_UnitLevelData` 曲線換算成最終等級） |
| **HealthMult / AtkMult** | 是 | HP / 攻擊倍率（套到 `UnitLevelData.GetMaxHP / GetAtkMult` 結果上） |
| **PriceMult / SoulPriceMult** | 是 | 商人 / 靈魂商店價格倍率 |
| **DarkMistMult** | 是 | 暗霧累積速度倍率 |
| **SellPriceMult** | — | 玩家出售物品的倍率（預設 0.5） |
| **InitResources** | 否 | 開局額外資源 |
| **CampFireDecreaseRate** | — | 篝火使用次數的衰減率（每次使用後減的比例） |
| **CampFireHealPercentageMult** | — | 篝火治療百分比倍率 |
| **ItemUsageLimit** | — | 每場戰鬥使用道具的上限（0 = 無限） |
| **AdditionalLength** | — | 大地圖長度加成 |
| **AdditionalQuestProgress** | — | Quest 進度加成 |
| **EnemySkillLevel** | — | 敵人基礎技能等級加成（影響 `MonsterLevelActionData.GetAction` 的 level） |
| **FieldEffects** | 否 | 永久套用的場地效果清單 |
| **RequiredCompletedDifficulty** | 否 | 前置難度（**OR**：完成任一即可解鎖此難度） |

## 行為說明

### 全局倍率作用點
這些 mult 值會在不同地方被乘上：
*   `UnitLevelData.GetMaxHP / GetAtkMult` 套用 `HealthMult / AtkMult`。
*   商店 UI 顯示售價時套用 `PriceMult / SoulPriceMult`。
*   出售物品計算售價時套用 `SellPriceMult`。
*   暗霧累積邏輯套用 `DarkMistMult`。
*   `MonsterLevelActionData.GetAction` 加上 `EnemySkillLevel`。

### `OnVictory()`
勝利並選擇繼續遊戲時呼叫，自動 `++m_EnemySkillLevel`——讓「無限模式」每勝一輪敵人技能升級一次。

### 描述
`LocalizedDescription` 從本地化系統取 key `<ID>_Description`，所以**描述要寫在 zh-Hant.txt / en.txt 裡**，而不是直接編在 Asset 上。

## 注意事項

*   **`RequiredCompletedDifficulty` 是 OR 關係**：滿足任一即解鎖。設計多階解鎖時要列出所有可接受的前置。
*   **`OnVictory` 永久修改 Asset**：`m_EnemySkillLevel` 是序列化欄位，勝利後會持久化。**這意味著每次勝利下次選同難度時都會更難**——這是「無限模式」設計，不是 bug。
*   **`Difficulty_Tutorial`** (`TutorialDifficultyID`) 是教學專用 ID，遊戲一開始強制使用。
*   **`m_FieldEffects` 永久套用**：每場戰鬥開始時都會套上，無法移除。
*   **`Description` 用 i18n key**：直接編 Asset 上的 description 欄位無效，必須加到語言檔。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_DifficultyData.cs`
*   **繼承自**：`RCG_Asset<RCG_DifficultyData>`
*   **AssetGroup**：`EditGameSetting`
*   **預設 ID**：`Difficulty_Normal`（也是建構式預設）

### A.2 欄位對照（節選）

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_IconSprite` | IconSprite | `RCG_SpriteData` | |
| `m_IsHidden` | IsHidden | `bool` | |
| `m_SortOrder` | SortOrder | `int` | 預設 1 |
| `m_BaseLevel` | BaseLevel | `int` | 預設 1 |
| `m_FieldEffects` | FieldEffects | `List<RCG_FieldEffectGenData>` | |
| `m_HealthMult` / `m_AtkMult` / `m_PriceMult` / `m_SoulPriceMult` / `m_DarkMistMult` | 各 Mult | `float` | 預設 1f |
| `m_SellPriceMult` | SellPriceMult | `float` | 預設 0.5f |
| `m_InitResources` | InitResources | `List<RCG_ResourceGenData>` | |
| `m_CampFireDecreaseRate` / `m_CampFireHealPercentageMult` | 篝火相關 | `float` | |
| `m_ItemUsageLimit` | ItemUsageLimit | `int` | |
| `m_AdditionalLength` / `m_AdditionalQuestProgress` | 進度加成 | `float / int` | |
| `m_EnemySkillLevel` | EnemySkillLevel | `int` | |
| `m_RequiredCompletedDifficulty` | RequiredCompletedDifficulty | `List<RCG_DifficultyGenData>` | OR 關係 |

### A.3 重要 Method

*   **`OnVictory()`** — `++m_EnemySkillLevel`（無限模式遞增）。
*   **`LocalizedName / LocalizedDescription`** — i18n 對應；description 走 `<ID>_Description` key。
*   **建構式預設 `ID = "Difficulty_Normal"`**。

### A.4 與其他系統的互動

*   **`RCG_UnitLevelData.GetMaxHP / GetAtkMult`** — 套用 HealthMult / AtkMult。
*   **`RCG_MonsterLevelActionData.GetAction`** — 加 EnemySkillLevel。
*   **`RCG_DataService.Ins.m_DifficultyData`** — runtime 套用點。
*   **`RCG_DifficultyGenData`** — Asset Entry；`TutorialDifficulty` 為教學專用。

### A.5 已知議題

*   `OnVictory` 永久遞增 `EnemySkillLevel` 會寫回 Asset；玩家在不同存檔玩同難度時會看到對方累積過的等級加成（**這是有意設計的「無限挑戰」機制**）。