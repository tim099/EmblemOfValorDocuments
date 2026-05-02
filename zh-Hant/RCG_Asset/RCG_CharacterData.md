---
title: 玩家角色資料 (RCG_CharacterData) 說明
description: 玩家可選擇 / 加入隊伍的角色模板：HP、初始牌組、初始技能、解鎖條件、加入特典
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 玩家角色資料

> 程式類別名稱：`RCG_CharacterData`

## 用途

**玩家可選擇或在劇情中加入隊伍的角色模板**。每個英雄、夥伴、可招募 NPC 都是一個 `RCG_CharacterData`。內含：基本資訊、初始 HP、初始牌組、初始技能/專精、解鎖條件、加入隊伍時的「加入牌組 (JoinDeck)」等。

繼承自 `RCG_Asset<RCG_CharacterData>`，實作介面：`UCLI_ShortName` / `RCGI_Unloackable`（解鎖系統）。

## 編輯器中的樣貌

```
RCG_CharacterData: <ID>
    Data (UnitData)            ← 編輯時真正修改的設定（HP / Deck / Skills / Order / Unlock）
    RuntimeData                ← 遊戲執行時才變動的資料（裝備、當前技能等）
    Preview (右側)              ← 頭像 / 名稱 / 最大 HP / 技能 / 牌組
```

## 主要欄位（Data 內部）

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Name** | 是 | 角色顯示名（多語系） |
| **MaxHp** | 是 | 最大生命值 |
| **Avatar** | 是 | 頭像 / 立繪 sprite |
| **Intro** | 否 | 角色介紹文字（選角畫面用） |
| **Order** | 是 | 選角畫面排序索引 |
| **Deck** | 是 | 起始牌組（`RCG_DeckData`） |
| **JoinDeck** | 否 | 中途加入隊伍時帶來的牌組（與 Deck 不同的進度） |
| **UnitSkillDatas** | 否 | 起始已學會的單位技能 |
| **SkillTags** | 是 | 角色擁有的專精（戰士 / 法師 / 牧師…），用於卡牌專精檢查 |
| **Unlock** | 否 | 解鎖條件（`RCG_UnlockEntry`）。`UnlockType.Tutorial` 表示教學解鎖；商店解鎖則需另在 GameRecord 紀錄購買 |

## 行為說明

### 選角分類
程式提供四個 static 入口取得不同角色清單（皆按 `m_Order` 排序）：
*   `GetAllTutorialCharacters()` — 教學解鎖的角色
*   `GetAllUnlockedCharacters()` — 已解鎖
*   `GetAllLockedCharacters()` — 未解鎖
*   `GetAllJoinCharacters()` — 教學 + 已解鎖（可加入隊伍的）

### 解鎖判斷 (`Unlocked`)
*   `UnlockEntry.Unlocked == true`：已過解鎖條件 → 商店類還要看 `RCG_GameRecord.UnlockedCharacters` 是否含 ID。
*   `UnlockType.None`：永遠解鎖。
*   `UnlockType.Tutorial`：要 GameRecord 內登記過完成教學。

### 編輯器分頁
*   **OnGUI** 顯示左側完整 Data 編輯欄 + 右側預覽。
*   **TestDropSkills** 按鈕（內建測試工具）可預覽掉落 / 抽卡組合。

## 注意事項

*   **`Order` 影響選角畫面排序**：未填會堆在最前。
*   **`JoinDeck` 與 `Deck` 是兩份**：開局選的角色用 `Deck`；劇情途中招募的角色用 `JoinDeck`（通常較精簡，避免破壞牌組平衡）。
*   **解鎖條件的兩層判斷**：`UnlockEntry` + 商店購買記錄；改動解鎖邏輯時兩邊都要檢查。
*   **`m_RuntimeData`** 是執行時才變的東西（裝備、當前技能），編輯器不要動它。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_CharacterData.cs`
*   **繼承自**：`RCG_Asset<RCG_CharacterData>`
*   **實作介面**：`UCLI_ShortName` / `RCGI_Unloackable`
*   **AssetGroup**：`EditCharacter`

### A.2 欄位對照（外層）

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_Data` | Data | `UnitData`（巢狀） | 編輯期資料 |
| `m_RuntimeData` | RuntimeData | `UnitRuntimData` | 執行期資料（裝備等） |

`UnitData` 內含 `m_LocalizeName` / `m_Avatar` / `m_Intro` / `m_MaxHp` / `m_Deck` / `m_JoinDeck` / `m_UnitSkillDatas` / `m_SkillTags` / `m_Unlock` / `m_Order` 等。

### A.3 重要 Method 摘要

*   **`Unlocked` (property)** — 兩階段解鎖判斷（UnlockEntry → GameRecord）。
*   **`GetAllTutorialCharacters` / `GetAllUnlockedCharacters` / `GetAllLockedCharacters` / `GetAllJoinCharacters`** (static) — 選角畫面用。
*   **`Preview` / `OnGUI`** — 編輯器繪製。
*   **`Data.TestDropSkills(...)`** — 內建掉落測試工具。
*   **`Init(string ID)` / `Init(CharacterID)`** — 多型建構入口。

### A.4 與其他系統的互動

*   **`RCG_DeckData`** — `m_Deck` / `m_JoinDeck` 引用的牌組。
*   **`RCG_UnitSkillData`** — `m_UnitSkillDatas` 引用的初始技能。
*   **`RCG_SkillTagGenData`** — 專精系統。
*   **`RCG_GameRecord.UnlockedCharacters`** — 商店 / 教學解鎖記錄。
*   **`RCG_UnlockEntry`** — 解鎖條件。
*   **`UnitRuntimData`** — 執行期角色狀態（HP、裝備）。

### A.5 已知議題

*   `Unlocked` 的 `UnlockType.Tutorial` 路徑有 `// QWQ23!!` 註解，標示舊版邏輯異動。
*   `m_ActivePowers` 已被註解，表示曾規劃過「主動能力」欄位，目前由 `UnitSkillData` 系統取代。
