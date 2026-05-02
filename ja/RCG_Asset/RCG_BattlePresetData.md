---
title: 戰鬥預設組合 (RCG_BattlePresetData) 說明
description: 用來快速啟動測試戰鬥的「全套配置」：怪物 + 玩家角色 + 牌組 + 裝備 + 難度，一鍵 StartBattle
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-ja
---

> [!WARNING]
> 翻訳待機中 — このファイルは日本語翻訳が必要です。
参考用に zh-Hant 原文を以下に掲載しています。


# 戰鬥預設組合

> 程式類別名稱：`RCG_BattlePresetData`

## 用途

**測試戰鬥用的快速配置**。把一整套「怪物 + 玩家角色 + 牌組 + 裝備 + 技能 + 難度」打包成一個 Asset，按下 Editor 內的 `StartBattle` 按鈕即可立刻進入戰鬥（跳過大地圖、角色選擇等流程）。**不是正式遊戲流程使用**，純粹是 QA / 設計師調整數值的調試工具。

繼承自 `RCG_Asset<RCG_BattlePresetData>`。

## 編輯器中的樣貌

```
RCG_BattlePresetData: <ID>
    BattleSetGenData   ← 引用的戰鬥組合
    Players            ← 出戰角色清單
    ExtraCards         ← 起始額外卡（除了 Deck 之外）
    Items              ← 起始道具
    Deck               ← 起始牌組（替換預設）
    Equipments         ← 各角色穿戴的裝備（dictionary）
    UnitSkills         ← 各角色已學的技能（dictionary）
    AllEquipments      ← 沒裝備到身上的裝備（背包）
    EnemyType          ← 敵人類型（普通 / 精英 / Boss）
    DifficultyData     ← 全局難度資料
    Difficulty         ← 動態難度值
    [按鈕] StartBattle ← Editor playing 時可一鍵開戰
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **BattleSetGenData** | 是 | 引用的 `RCG_BattleSet`（怪物配置） |
| **Players** | 是 | 出戰角色清單 |
| **ExtraCards** | 否 | 額外塞進牌組的卡（用於測試特定卡組合） |
| **Items** | 否 | 起始道具（藥水、捲軸） |
| **Deck** | 否 | 起始牌組；非 `Default` 時會覆蓋玩家牌組 |
| **Equipments** | 否 | 角色 → 裝備清單的 dictionary（哪個角色穿哪些裝備） |
| **UnitSkills** | 否 | 角色 → 技能清單的 dictionary |
| **AllEquipments** | 否 | 不裝備到身上、放背包的裝備 |
| **EnemyType** | 否 | 敵人類型 |
| **DifficultyData** | 否 | 全局難度設定（HP / Atk 倍率、敵人技能等級） |
| **Difficulty** | 否 | 動態難度值（疊加在大地圖難度之上） |

## 行為說明

### `StartBattleAsync`
按 `StartBattle` 按鈕（僅 Editor playing 時顯示）會：
1. 建立 BigMapManager 並進入大地圖。
2. 進入 Quest（包成 EnterQuestSetting）。
3. 套用 `m_DifficultyData` 與 `m_Difficulty` 到 `RCG_DataService`。
4. 替換玩家牌組（如果 `m_Deck.ID != Default`）。
5. 加入 ExtraCards / Items / Equipments / UnitSkills 等。
6. 進入戰鬥場景。

### 預覽
顯示底層 BattleSet 的 Preview，並提供 StartBattle 按鈕。

## 注意事項

*   **僅供測試**：正式遊戲不從這裡進入戰鬥；不要依賴此資料設計正式關卡。
*   **`m_Deck.ID == DefaultID` 時不替換牌組**，會用玩家當前牌組 — 確認此 Asset 的 `m_Deck` 有指定才會生效。
*   **InitActivePowers 邏輯已註解**：原本會套用角色初始主動能力，目前由 UnitSkill 系統取代，註解掉了。
*   **Editor not playing 時無法看到 StartBattle 按鈕**——必須在 Play Mode 下開 Developer 頁面才能用。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattlePresetData.cs`
*   **繼承自**：`RCG_Asset<RCG_BattlePresetData>`
*   **AssetGroup**：`EditBattleSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_BattleSetGenData` | BattleSet | `RCG_BattleSetGenData` | |
| `m_Players` | Players | `List<RCG_CharacterGenData>` | |
| `m_ExtraCards` | ExtraCards | `List<RCG_CardGenData>` | |
| `m_Items` | Items | `List<RCG_ItemGenData>` | |
| `m_Deck` | Deck | `RCG_DeckGenData` | |
| `m_Equipments` | Equipments | `Dictionary<RCG_CharacterGenData, List<RCG_EquipmentGenData>>` | |
| `m_UnitSkills` | UnitSkills | `Dictionary<RCG_CharacterGenData, List<RCG_UnitSkillGenData>>` | |
| `m_AllEquipments` | AllEquipments | `List<RCG_EquipmentGenData>` | 背包 |
| `m_EnemyType` | EnemyType | `RCG_EnemyTypeTagGenData` | |
| `m_DifficultyData` | DifficultyData | `RCG_DifficultyData` | |
| `m_Difficulty` | Difficulty | `int` | |

### A.3 重要 Method 摘要

*   **`StartBattleAsync(BigMap, Quest, CancellationToken)`** — 主入口；建大地圖 → 進 Quest → 套難度 → 替換 Deck → 加 Cards/Items/Equipments/UnitSkills。
*   **`Preview`** — 編輯器內顯示 BattleSet preview + StartBattle 按鈕。

### A.4 與其他系統的互動

*   **`RCG_BigMapManager`** / **`RCG_MapManager`** — 進入流程的核心。
*   **`RCG_DataService.Ins.m_DifficultyData / Difficulty`** — 套用難度。
*   **`RCG_DataService.Ins.m_DeckData`** — 替換牌組目標。
*   **`RCG_MapEventManager.Reset`** — 進入新戰鬥前清除事件佇列。

### A.5 已知議題

*   `// 初始角色能力` 一段已被註解，舊版透過 ActivePower 系統灌入初始能力的邏輯廢棄。