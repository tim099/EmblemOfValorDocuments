---
title: 任務資料 (RCG_QuestData) 說明
description: 一段任務（小地圖）的完整定義：節點配置、戰鬥池、獎勵池、目標、暗霧、隨機生成
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-en
---

> [!WARNING]
> Translation pending — this file needs an English translation.
The original zh-Hant content is included below for reference.


# 任務資料

> 程式類別名稱：`RCG_QuestData`

## 用途

**一段「任務」（小地圖）的完整定義**。從大地圖進入任務節點後，遊戲會生成這段任務對應的小地圖：節點配置、節點間路徑、敵人配置、獎勵池、暗霧規則、任務目標。任務支援多種生成方式：手動編輯、純隨機、混合隨機、殺戮尖塔風格、洞窟、遺跡⋯

繼承自 `RCG_Asset<RCG_QuestData>`，實作介面：`UCL.Core.UI.UCLI_FieldOnGUI`。

## 編輯器中的樣貌

```
RCG_QuestData: <ID>
    Name / QuestObjective / QuestIcon / BGM / Map prefab
    QuestGoals                 ← 任務目標（主要 / 次要）
    FieldEffects               ← 場地效果（可開關）
    StoryPool                  ← 此任務的事件池
    QuestBattles               ← 預設戰鬥池（按敵人類型分）
    Card / Item / EquipmentDropPools  ← 各類獎勵池（按敵人類型分）
    BattleScene                ← 預設戰鬥場景
    QuestType                  ← Default / RandomGen / MixRandom / SlayTheSpire / CaveGen / RuinGen / TestEvents / TestBattles
    [QuestType 對應的子設定]      ← 各種 RandomGenData
    DetailSetting              ← 條件 / Tag / 難度 / 暗霧 / 地圖大小 / 自動進入等
    MapEditData                ← 節點編輯資料（隱藏，用 QuestEditor 編）
    [按鈕] Open QuestEditor    ← Editor playing 時可開節點編輯器
    [按鈕] Export Quest Info   ← 匯出此任務的怪物 / 戰鬥資訊到 Markdown
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Name / QuestObjective** | 是 | 任務名 / 目標描述（多語系） |
| **QuestIcon** | 是 | 任務圖（大地圖節點顯示） |
| **BGM** | 是 | 背景音樂 |
| **Map** | 是 | 對應的小地圖 Prefab |
| **QuestGoals** | 否 | 主要 / 次要任務目標（達成觸發獎勵） |
| **FieldEffects** | 否 | 此任務的場地效果（可開關） |
| **StoryPool** | 否 | 此任務的故事事件池 |
| **QuestBattles** | 否 | 按敵人類型對應的戰鬥池（普通 / 精英 / Boss⋯） |
| **CardDropPools / ItemDropPools / EquipmentDropPools** | 否 | 按敵人類型對應的獎勵池；缺漏時 fallback 到 EnemyType 的預設池 |
| **BattleScene** | 否 | 預設戰鬥場景（無指定時用此） |
| **QuestType** | 是 | 地圖生成方式（決定下方哪個子設定可用） |
| **DetailSetting** | 是 | 細節（條件 / Tag / 難度 / 暗霧設定 / 地圖大小 / `AutoEnterIfOnly` 等） |

`DetailSetting` 內含：

| 子欄位 | 說明 |
|---|---|
| **CompleteConditions** | 顯示前置條件（AND；不符合則隱藏） |
| **IncompatibleConditions** | 互斥條件（不符合則隱藏） |
| **TopMenuState / QuestTags / QuestLv** | 上方選單狀態 / Tag 列表 / 難度等級 |
| **QuestLength / QuestDarkMistChange** | 任務長度 / 暗霧變化（會被 `DifficultyData.m_DarkMistMult` 乘） |
| **DarkMistRelatedSettings** | 暗霧細節：移動時是否上升、暗霧外觀、節點抗性自動分配 |
| **Difficulty / AddDifficultyAfterPass** | 任務基礎難度 / 通關後的難度增量 |
| **QuestProgress / QuestEventIcons** | 進度值 / 事件圖示 |
| **CustomizeMapSize / MapWidth / MapHeight** | 自訂地圖長寬 |
| **AutoEnterIfOnly** | 大地圖只有此任務時自動進入 |

## 行為說明

### 隨機生成 (`GetMapEditData`)
依 `QuestType` 分流到對應的 GenData：
*   `MixRandom` → `m_QuestMixRandomData.RandomGen` + `BalanceDistanceIteration`
*   `RandomGen` / `SlayTheSpire` / `SlsHorizontal` / `CaveGen` / `TestEvents` → 各自的 RandomGen
*   `Default` → 直接讀預存的 `m_MapEditData`
*   `RuinGen` / `TestBattles` → **未實作**（log error）

### 任務目標自動生成 (`GenerateQuestGoals` static)
若任務沒設目標，靜態方法會依當前大地圖階段的獎勵池生成：
*   **主目標**（基於 hash seed）：3 種輪換——裝備獎 / 單位技能獎 / 卡牌獎；要求是「擊敗 Boss」（Boss tag 任務）或「擊敗 N 個精英」。
*   **次目標**：3 種輪換——靈魂值門檻 / 暗霧等級門檻 / 一般敵擊敗數；獎勵是金錢 / 靈魂值 / 道具。

### 挑戰目標附加 (`AppendGameChallengeGoals` static)
把 `RCG_DataService.Ins.m_ChallengeGoalDatas` 的未完成挑戰目標標記為 `m_IsChallengeGoal` 並附加到任務目標。

### 通關 (`QuestComplete`)
`Difficulty += m_DetailSetting.m_AddDifficultyAfterPass`：通關此任務後永久提升難度。

### 進度 (`QuestProgress`)
*   `BigMapType.Loop`（舊版 Loop 大地圖）→ 直接回 `QuestProgressToComplete`。
*   新版（按 Tag 計數）→ 含任一 `BigMapData.QuestProgressTags` 的 Tag → 回 1，否則回 0。

### 條件檢查
*   **`IsPrerequirementMet(selected)`** — 已通關過 → false；`CompleteConditions` 全 AND 通過。
*   **`IsQuestAvailable(selected)`** — 已通關過 → false；`IncompatibleConditions` 全 AND 通過。

## 注意事項

*   **`QuestType` 換掉時記得清空 `MapEditData`**：否則舊隨機資料會殘留干擾。`RemoveRandomNodes` 可清。
*   **`AutoEnterIfOnly` 只在常駐任務 / 起始祝福場景才有效**：大地圖檢查只剩此任務時自動進入。
*   **目標自動生成是 deterministic**：用 quest IDs 的 hash 算 seed，**重複進入同一組任務會生成相同目標**。
*   **`RuinGen` / `TestBattles` 未實作**：選了會 LogError 而沒有實際生成。
*   **`m_FieldEffects` 元素是 `ToggleableFieldEffectGenData`**：可關閉，runtime 走 `FieldEffects` property 過濾出 enabled 的。
*   **`Export Quest Info` 按鈕**：匯出怪物配置 Markdown 到 `Assets/Export/Quest/<ID>.md`，方便外部審查戰鬥配置。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_MapScripts/RCG_QuestData.cs`
*   **繼承自**：`RCG_Asset<RCG_QuestData>`
*   **實作介面**：`UCL.Core.UI.UCLI_FieldOnGUI`
*   **AssetGroup**：`EditQuestSetting`
*   **CurQuest** (static) — 最近建構的 RCG_QuestData，方便外部存取

### A.2 欄位對照（節選）

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_Name` / `m_QuestObjective` | Name / Objective | `RCG_LocalizeData` | |
| `m_QuestIcon` / `m_BGM` / `m_Map` | 圖 / BGM / 地圖 | `RCG_SpriteData` / `RCG_BGMGenData` / `RCG_PrefabResData` | |
| `m_QuestGoals` | QuestGoals | `List<RCG_QuestGoalData>` | |
| `m_FieldEffects` | FieldEffects | `List<ToggleableFieldEffectGenData>` | |
| `m_Tags` | Tags | `List<RCG_EventTagGenData>` | |
| `m_StoryPool` | StoryPool | `RCG_StoryDropPoolGenData` | |
| `m_MapEditData` | MapEditData | `MapEditData` | `[UCL_HideOnGUI]` |
| `m_QuestBattles` | QuestBattles | `Dictionary<RCG_EnemyTypeTagGenData, RCG_BattleSetDropPoolGenData>` | |
| `m_CardDropPools / m_ItemDropPools / m_EquipmentDropPools` | 各獎勵池 | `Dictionary<EnemyType, *DropPoolGenData>` | |
| `m_BattleScene` | BattleScene | `RCG_BattleSceneGenData` | |
| `m_QuestType` | QuestType | `QuestType` enum | 8 種模式 |
| `m_QuestRandomGenData / m_QuestMixRandomData / m_QuestCaveGenData / m_SlayTheSpireGenData / ...` | 各 GenData | 各自類別 | `Conditional(對應 QuestType)` |
| `m_DetailSetting` | DetailSetting | `DetailSetting` | |

### A.3 重要 Method 摘要

*   **`GetMapEditData(isLoadMap)`** — 主入口；按 QuestType 隨機生成或讀預存。
*   **`QuestComplete()`** — 通關後 Difficulty +=。
*   **`GenerateQuestGoals` (static)** — 自動產主 + 次目標。
*   **`AppendGameChallengeGoals` (static)** — 附加挑戰目標。
*   **`GetCardDropPool / GetItemDropPool / GetEquipmentDropPool / GetBattleSet`** — 按敵人類型取對應池；缺則 fallback。
*   **`IsPrerequirementMet / IsQuestAvailable`** — 顯示條件檢查。
*   **`QuestProgress` (property)** — 新舊兩種計算方式。
*   **`SaveGame / LoadGame`** — 隨機生成的 MapEditData 序列化。
*   **`ExportQuestInfoToJson`** — 匯出怪物資訊 Markdown 到 `Assets/Export/Quest/`。
*   **`OnGUI(field, dataDic, params)`** — 自訂繪製（PreviewTexture + Export 按鈕）。

### A.4 與其他系統的互動

*   **`MapEditData`** — 節點 / 路徑容器。
*   **`RCG_QuestDropPool`** — Quest 的隨機池。
*   **`RCG_BigMapData.QuestProgressTags`** — 進度判斷依據。
*   **`RCG_QuestRandomGenData / RCG_QuestMixRandomData / RCG_QuestCaveGenData / RCG_SlayTheSpireGenData / ...`** — 各 QuestType 的生成器。
*   **`RCG_QuestGoalData / QuestGoalRequirementData / GoalReward*`** — 目標 / 獎勵系統。
*   **`RCG_DataService.Ins.m_ChallengeGoalDatas`** — 挑戰目標來源。
*   **`UI.RCG_QuestEditorUI`** — 節點編輯器。

### A.5 已知議題

*   `RuinGen` / `TestBattles` 未實作，選了只 LogError。
*   `GenerateQuestGoals` 用 hash seed，缺乏隨機種子重置機制——同一組 IDs 永遠生同樣目標。
*   `OnGUI` 內舊版圖形繪製代碼（GL.Begin / GL.LINES）大段註解，現改用 PreviewTexture。
*   多處 `// QWQ` / `// QWQ23` 註解標示待重構與舊版遷移點。