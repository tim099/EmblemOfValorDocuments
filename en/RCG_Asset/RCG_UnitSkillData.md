---
title: 單位技能資料 (RCG_UnitSkillData) 說明
description: 角色學會的「技能」——類似裝備但不佔裝備槽；提供被動效果、領袖技、學習特典
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-en
---

> [!WARNING]
> Translation pending — this file needs an English translation.
The original zh-Hant content is included below for reference.


# 單位技能資料

> 程式類別名稱：`RCG_UnitSkillData`

## 用途

**角色學會的技能模板**。類似裝備，但不佔裝備槽。每個技能可以：
*   提供被動效果（戰鬥中各種 trigger 觸發）
*   標記為「**領袖技**」（只在角色站第一位時生效）
*   學習時觸發一次性事件（例：永久 +1 HP、解鎖額外抽卡格）

繼承自 `RCG_Asset<RCG_UnitSkillData>`，實作介面：`RCGI_Status`（戰鬥狀態系統）/ `RCGI_Unloackable`（解鎖）。

## 編輯器中的樣貌

```
RCG_UnitSkillData: <ID>
    Name(多國語言)
    Icon                          ← 技能圖
    Effects                       ← 戰鬥中觸發的效果（OnPlay / OnTurnStart / ...）
    AcquireSkillEvents            ← 學會時觸發的一次性事件
    SkillTags                     ← 需要的專精（角色須符合才能學）
    Tags                          ← 一般標籤
    UnitSkillDescription / Template ← 描述方式（Auto / Manually）
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Name** | 是 | 技能顯示名（多語系） |
| **Icon** | 是 | 技能圖示 |
| **Effects** | 否 | 戰鬥中各種 trigger 上要跑的效果（OnPlay / OnTurnStart / OnAttack…） |
| **AcquireSkillEvents** | 否 | 學會技能時觸發的事件（例：擴充手牌數、永久強化） |
| **SkillTags** | 否 | 必要專精——角色擁有的專精須**全包含**這裡列的才能學（AND 關係） |
| **Tags** | 否 | 一般物品/技能標籤（用於分類、條件判斷） |
| **CanLearnRepeatedly** | — | 是否可重複學習（疊加效果） |
| **HideThisSkill** | — | 學會後**不顯示在技能欄**（適合「只觸發學習特典」這種隱藏增益） |
| **IsLeaderSkill** | — | 是否為**領袖技**：只在角色站第一位時觸發 Effects |
| **CanDrop** | — | 是否能從掉落池抽到（false = 只能透過特定途徑取得） |
| **InitCounters** | 否 | 起始計數器值（給 Effects 中的計數器類效果使用） |
| **Unlock** | 否 | 解鎖條件 |
| **UnitSkillDescriptionType** | 是 | `Auto`（自動由 Effects 合成）或 `Manually`（手動撰寫） |
| **UnitSkillDescription** | Manually 時 | 手動描述（多語系） |
| **DescriptionTemplate** | Manually 時 | 含 `{(OnPlay.0)}` 等佔位符的範本字串 |

## 行為說明

### 觸發判定
`OnUnitState(triggerOn, data)` 在每個 trigger 上：
1. 從 `Effects` 取出該 trigger 的所有效果。
2. 若 `IsLeaderSkill = true` 且擁有者**不是隊伍第一位**（與 `RCG_BattleField.LeadUnit` 比對 ID），跳過。
3. 否則逐一觸發。

### 學會時 (`OnAquireSkill`)
所有 `AcquireSkillEvents` 加入 `RCG_MapEventManager`，套到該角色身上（會立刻或在合適時機 fire）。

### 描述生成
*   **Auto**：依「領袖技標籤 → AcquireEvents 描述 → Effects 描述」順序自動串接。
*   **Manually**：以 `UnitSkillDescription` 為本體，透過 `GetDescriptionParams()` 替換 `(OnPlay.0)` 之類的佔位符；可按 `Generate Description Template` 按鈕從現有 Effects 自動產出範本。

### Tooltip Infos
`Infos` 屬性聚合所有 effect 的 `CardInfoData`；領袖技會在最前插入 `BattleTag_LeadAbility` 的解說。

## 注意事項

*   **`SkillTags` 是 AND 關係**：列了多個專精表示「全部都要有」才能學；要做「擇一」需開兩個技能。
*   **`HideThisSkill = true`** 適合純粹做「學會時加 max HP」這類**永久效果而無被動**的技能；玩家看不到 buff icon 但效果已生效。
*   **`IsLeaderSkill`** 與隊伍切換領袖機制綁定；領袖切換時不會自動重觸發 Effects（`OnPlay` 不會重 fire）。
*   **`CanDrop = false`** 的技能不會被 DropPool 抽到，常用於劇情解鎖技能。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_UnitSkillData.cs`
*   **繼承自**：`RCG_Asset<RCG_UnitSkillData>`
*   **實作介面**：`RCGI_Status` / `RCGI_Unloackable`
*   **AssetGroup**：`EditCharacter`
*   **預設 Icon 路徑**：`RCG_SpriteData.SpriteFolder + "/UnitSkills"`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | `Name` |
| `m_Icon` | Icon | `RCG_SpriteData` | `Icon` |
| `m_Effects` | Effects | `List<RCG_CommonEffect>` | `Effects` |
| `m_AcquireSkillEvents` | AcquireSkillEvents | `List<RCG_MapEvent>` | `AcquireSkillEvents` |
| `m_SkillTags` | SkillTags | `List<RCG_SkillTagGenData>` | `SkillTags` |
| `m_Tags` | Tags | `List<RCG_ItemTagGenData>` | `Tags` |
| `m_CanLearnRepeatedly` | CanLearnRepeatedly | `bool` | `CanLearnRepeatedly` |
| `m_HideThisSkill` | HideThisSkill | `bool` | `HideThisSkill` (Header `HideThisSkillDes`) |
| `m_IsLeaderSkill` | IsLeaderSkill | `bool` | `IsLeaderSkill` |
| `m_CanDrop` | CanDrop | `bool` | `CanDrop` |
| `m_InitCounters` | InitCounters | `List<int>` | `InitCounters` |
| `m_Unlock` | Unlock | `RCG_UnlockEntry` | `Unlock` |
| `m_UnitSkillDescriptionType` | DescriptionType | `CardDescriptionType` | — |
| `m_UnitSkillDescription` | Description | `RCG_LocalizeData` | `Conditional(Manually)` |
| `m_DescriptionTemplate` | Template | `string` | `Conditional(Manually)` |

### A.3 重要 Method 摘要

*   **`OnUnitState(RCG_EffectTriggerOn, TriggerEffectData)`** — 主要觸發入口；含領袖技判斷。
*   **`TriggerOnUnitState(RCG_EffectTriggerOn)`** — 是否在此 trigger 有 effect（給 trigger 系統做 quick check）。
*   **`OnAquireSkill(RCG_CharacterData)`** — 學會時把 `AcquireSkillEvents` 加入 `RCG_MapEventManager`。
*   **`CheckRequireSkill(HashSet<RCG_SkillTagGenData>)`** — 角色當前 skills 是否包含全部 `m_SkillTags`。
*   **`Description` (property)** — Auto / Manually 的描述邏輯。
*   **`GetDescriptionTemplate / GetDescriptionParams`** — Manually 模式下的範本生成 / 參數抽取。
*   **`Status` (property)** — 回 `new RCG_StatusGenData(StatusType.UnitSkill, ID)`，作為 RCGI_Status 介面實作。

### A.4 與其他系統的互動

*   **`RCG_CommonEffect`** — 觸發效果單位。
*   **`RCG_MapEvent` / `RCG_MapEventManager`** — 學習特典事件系統。
*   **`RCG_BattleField.LeadUnit`** — 領袖技判斷。
*   **`RCG_BattleTag.Util.GetData("BattleTag_LeadAbility")`** — 領袖技標籤。