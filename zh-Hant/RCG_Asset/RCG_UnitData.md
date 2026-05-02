---
title: 單位資料 (RCG_UnitData) 說明
description: 戰場上所有單位（怪物、玩家角色、召喚物）的本體資料：HP、外觀、AI 行為、技能等全部定義在這
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 單位資料

> 程式類別名稱：`RCG_UnitData`

## 用途

**戰場上每一個單位的本體模板**——怪物、玩家角色、召喚物都是這個類別的實例。包含：HP / 戰鬥力、外觀（圖、Spine、龍骨、3D Prefab）、AI 行為（不同狀態下使用哪些技能）、初始動作、必要的單位設定等。取代舊的 `RCG_MonsterData`。

繼承自 `RCG_Asset<RCG_UnitData>`。

## 編輯器中的樣貌

```
RCG_UnitData: <ID>
    UnitSetting (m_UnitSetting)        ← 主要設定：HP、標籤、初始動作、外觀、定位
    MonsterStates (m_MonsterStates)    ← AI 狀態機：每個狀態下使用哪些 Action
    Preview (右側)                      ← 即時呈現外觀、技能列表、HP
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **UnitSetting** | 是 | 單位的核心設定（HP / 標籤 / 初始 Action / 外觀 / 定位等） |
| **MonsterStates** | 是 | AI 狀態機：dictionary，key 是狀態 ID，value 是該狀態下可用的 `MonsterAction` 清單。**至少要有 `Default` 狀態**（系統會自動補上） |

`UnitSetting` 內部包括：

| 子欄位 | 說明 |
|---|---|
| **Name** | 單位顯示名（多語系） |
| **MaxHP** | 最大生命值 |
| **CombatEffectiveness** | 戰鬥力指標；序列化時若為 0 會自動以 `MaxHP` 補上 |
| **MonsterTags** | 怪物標籤（種族、屬性等），影響某些卡牌/裝備效果 |
| **InitActions** | 進場時觸發的初始動作（buff / debuff / 召喚物） |
| **DetailSetting** | 細節設定（攻擊次數、計數器⋯，視類型而定） |
| **SummonSetting** | 被召喚時的設定 |
| **UnitDisplayerType** | 顯示方式：Sprite / DragonBone / Spine / 3D Model / Prefab |
| **SpriteDisplayData / DragonBoneDisplayData / ...** | 對應顯示器的資料（Conditional 顯示） |
| **Pos / PosAt** | 站位偏移、座標 |
| **BaseLevel / UnitGenData** | 基礎等級與單位 ID 包裝 |
| **HasAI / Classes / UnitSkills** | AI 開關、職業（卡牌專精用）、攜帶的單位技能 |

## 行為說明

### AI 狀態機 (MonsterStates)
每個怪物有多個「狀態」（例如 `Default`、`Berserk`、`Phase2`），每個狀態下有一組可用的 `MonsterAction`。實際戰鬥中由 `MonsterStateTransition` 決定何時切換狀態，由 Action 的選擇規則決定當回合用哪一招。

### 顯示方式
*   **Sprite**：靜態圖（最簡單，適合 placeholder）。
*   **DragonBone / Spine**：2D 骨架動畫。
*   **Model3DDisplayer / Prebab**：3D 模型 / 自訂 Prefab。
切換 `UnitDisplayerType` 後對應欄位才會顯示。

### 初始動作 (InitActions)
進場時自動套用一次的動作。常見用途：給予自身 buff、召喚從者、設定計數器初值。

### 預覽（編輯器內）
右側即時顯示：頭像、最大 HP、怪物標籤、初始動作描述、各狀態下的技能圖示與名稱。**Low RAM 模式**會跳過頭像繪製以省記憶體。

## 注意事項

*   **`Default` 狀態必須存在**：系統會在 `OnGUI` 自動補上，但保險起見先設好。
*   **`CombatEffectiveness` 的 fallback**：`SerializeToJson` 時若該值為 0 會用 `MaxHP` 補；想保留 0 須特別處理。
*   **`NullMonsterID = "Null"`、`DefaultUnitID = "Devil"`、`BattleSceneMonsterID = "BattleScene"`**：這些是系統保留 ID，不要拿來命名一般怪物。
*   **編輯介面的 Skill / Action / SummonSetting** 各自從 `RCG_MonsterActionData` / `RCG_UnitSkillData` 等 Asset 引用，本檔只持有 ID 包裝。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_Battles/RCG_Monsters/RCG_UnitData.cs`
*   **繼承自**：`RCG_Asset<RCG_UnitData>`
*   **AssetGroup**：`EditBattleSetting`，sort = `RCG_UnitData`

### A.2 欄位對照（外層）

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_UnitSetting` | UnitSetting | `RCG_UnitSetting`（巢狀） | 主要核心資料 |
| `m_MonsterStates` | MonsterStates | `Dictionary<string, MonsterState>` | AI 狀態機；key 是狀態 ID |

`UnitSetting` 內含 `m_Name` / `m_MaxHP` / `m_CombatEffectiveness` / `m_MonsterTags` / `m_InitActions` / `m_DetailSetting` / `m_SummonSetting` / `m_UnitDisplayerType` + 對應 DisplayData / `m_Pos` / `m_PosAt` / `m_BaseLevel` / `m_UnitGenData` / `m_HasAI` / `m_Classes` / `m_UnitSkills`。

### A.3 重要 Method 摘要

*   **`Preview` / `OnGUI`** — 編輯器渲染；OnGUI 會自動補 `Default` MonsterState。
*   **`SerializeToJson`** — 補 `m_CombatEffectiveness == 0` 的 fallback。
*   **`Avatar` / `GetAvatar(RCG_BattleUnit)`** — 顯示器圖像取得。
*   **`CreateSelectAssetPage`** — 開 `RCG_UnitDataEditorPage`。
*   **`LocalizeName`** — `GetData().LocalizedName`。

### A.4 與其他系統的互動

*   **`RCG_BattleUnit`** — runtime 戰鬥單位實例。
*   **`MonsterState` / `MonsterStateTransition`** — 狀態機的元件（含 `DefaultStateID`）。
*   **`RCG_MonsterAction` / `RCG_MonsterActionData`** — 怪物可用招式定義。
*   **`RCG_UnitSkillData`** — 單位被動 / 領袖技。
*   **`RCG_UnitDataEditorPage`** — 編輯主畫面。

### A.5 已知議題

*   `DeserializeFromJson` 有被註解掉的 `m_UnitSetting = m_MonsterData` 容錯（舊版欄位遷移）。
*   程式註解 `// 暫時用HP當作戰鬥力指標` 指 `m_CombatEffectiveness` 的 fallback 規則待重新設計。
