---
title: RCG_UnitData 說明
description: <!-- TODO: 一句話功能摘要 -->
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# RCG_UnitData

> 程式類別名稱：`RCG_UnitData`

## 用途

<!-- TODO: 描述這個 Asset 在遊戲裡負責什麼、什麼情境會用、舉 1-2 個範例。 -->

繼承自 `RCG_Asset<RCG_UnitData>`。

## 編輯器中的樣貌

```
<!-- TODO: 描繪此 Asset 在編輯器內的版面 -->
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **HitClip** | — | <!-- TODO: 說明欄位用途 --> |
| **DieClip** | — | <!-- TODO: 說明欄位用途 --> |
| **BattleStartClips** | — | <!-- TODO: 說明欄位用途 --> |
| **BattleWinClips** | — | <!-- TODO: 說明欄位用途 --> |
| **AttackClips** | — | <!-- TODO: 說明欄位用途 --> |
| **UnitLevelGenData** | — | <!-- TODO: 說明欄位用途 --> |
| **Note** | — | <!-- TODO: 說明欄位用途 --> |
| **HasAI** | — | <!-- TODO: 說明欄位用途 --> |
| **Classes** | — | <!-- TODO: 說明欄位用途 --> |
| **UnitSkills** | — | <!-- TODO: 說明欄位用途 --> |
| **Name** | — | <!-- TODO: 說明欄位用途 --> |
| **MaxHP** | — | <!-- TODO: 說明欄位用途 --> |
| **CombatEffectiveness** | — | <!-- TODO: 說明欄位用途 --> |
| **MonsterTags** | — | <!-- TODO: 說明欄位用途 --> |
| **InitActions** | — | <!-- TODO: 說明欄位用途 --> |
| **DetailSetting** | — | <!-- TODO: 說明欄位用途 --> |
| **SummonSetting** | — | <!-- TODO: 說明欄位用途 --> |
| **UnitDisplayerType** | — | <!-- TODO: 說明欄位用途 --> |
| **SpriteDisplayData** | — | <!-- TODO: 說明欄位用途 --> |
| **DragonBoneDisplayData** | — | <!-- TODO: 說明欄位用途 --> |
| **SpineDisplayData** | — | <!-- TODO: 說明欄位用途 --> |
| **Model3DDisplayerDisplayData** | — | <!-- TODO: 說明欄位用途 --> |
| **PrebabDisplayData** | — | <!-- TODO: 說明欄位用途 --> |
| **Pos** | — | <!-- TODO: 說明欄位用途 --> |
| **PosAt** | — | <!-- TODO: 說明欄位用途 --> |
| **BaseLevel** | — | <!-- TODO: 說明欄位用途 --> |
| **UnitGenData** | — | <!-- TODO: 說明欄位用途 --> |
| **UnitSetting** | — | <!-- TODO: 說明欄位用途 --> |
| **MonsterStates** | — | <!-- TODO: 說明欄位用途 --> |

## 行為說明

<!-- TODO: 戰鬥 / 載入 / 解鎖時的觸發時機與順序。 -->

## 注意事項

<!-- TODO: 常見的設計反模式 / 容易踩到的坑。 -->

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊

*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_Battles/RCG_Monsters/RCG_UnitData.cs`
*   **繼承自**：`RCG_Asset<RCG_UnitData>`
*   **實作介面**：（無）

### A.2 欄位對照（自動產生，需人工複核）

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_HitClip` | HitClip | `RCG_SEGenData` | `HitClip` | |
| `m_DieClip` | DieClip | `RCG_SEGenData` | `DieClip` | |
| `m_BattleStartClips` | BattleStartClips | `List<RCG_SEGenData>` | `BattleStartClips` | |
| `m_BattleWinClips` | BattleWinClips | `List<RCG_SEGenData>` | `BattleWinClips` | |
| `m_AttackClips` | AttackClips | `List<RCG_SEGenData>` | `AttackClips` | |
| `m_UnitLevelGenData` | UnitLevelGenData | `RCG_UnitLevelGenData` | `UnitLevelGenData` | |
| `m_Note` | Note | `string` | `Note` | |
| `m_HasAI` | HasAI | `bool` | `HasAI` | |
| `m_Classes` | Classes | `List<RCG_SkillTagGenData>` | `Classes` | |
| `m_UnitSkills` | UnitSkills | `List<RCG_UnitSkillGenData>` | `UnitSkills` | |
| `m_Name` | Name | `RCG_LocalizeData` | `Name` | |
| `m_MaxHP` | MaxHP | `int` | `MaxHP` | |
| `m_CombatEffectiveness` | CombatEffectiveness | `int` | `CombatEffectiveness` | |
| `m_MonsterTags` | MonsterTags | `List<RCG_MonsterTagGenData>` | `MonsterTags` | |
| `m_InitActions` | InitActions | `List<RCG_BattleSetting>` | `InitActions` | |
| `m_DetailSetting` | DetailSetting | `DetailSetting` | `DetailSetting` | |
| `m_SummonSetting` | SummonSetting | `SummonSetting` | `SummonSetting` | |
| `m_UnitDisplayerType` | UnitDisplayerType | `UnitDisplayerType` | `UnitDisplayerType` | |
| `m_SpriteDisplayData` | SpriteDisplayData | `SpriteDisplayData` | `SpriteDisplayData` | UCL.Core.PA.Conditional(nameof(m_UnitDisplayerType), false, UnitDisplayerType.Sprite, UnitDisplayerType.Substitution) |
| `m_DragonBoneDisplayData` | DragonBoneDisplayData | `DragonBoneDisplayData` | `DragonBoneDisplayData` | UCL.Core.PA.Conditional(nameof(m_UnitDisplayerType), false, UnitDisplayerType.DragonBone) |
| `m_SpineDisplayData` | SpineDisplayData | `SpineDisplayData` | `SpineDisplayData` | UCL.Core.PA.Conditional(nameof(m_UnitDisplayerType), false, UnitDisplayerType.Spine) |
| `m_Model3DDisplayerDisplayData` | Model3DDisplayerDisplayData | `Model3DDisplayerDisplayData` | `Model3DDisplayerDisplayData` | UCL.Core.PA.Conditional(nameof(m_UnitDisplayerType), false, UnitDisplayerType.Model3D) |
| `m_PrebabDisplayData` | PrebabDisplayData | `SpritePrefabDisplayData` | `PrebabDisplayData` | UCL.Core.PA.Conditional(nameof(m_UnitDisplayerType), false, UnitDisplayerType.SpritePrefab) |
| `m_Pos` | Pos | `UnitPos` | `Pos` | |
| `m_PosAt` | PosAt | `int` | `PosAt` | UCL.Core.ATTR.UCL_HideOnGUI |
| `m_BaseLevel` | BaseLevel | `IntVariable` | `BaseLevel` | |
| `m_UnitGenData` | UnitGenData | `RCG_UnitGenData` | `UnitGenData` | UCL.Core.ATTR.AlwaysExpendOnGUI |
| `m_UnitSetting` | UnitSetting | `UnitSetting` | `UnitSetting` | |
| `m_MonsterStates` | MonsterStates | `Dictionary<string, MonsterState>` | `MonsterStates` | |

### A.3 重要 Method 摘要

<!-- TODO: 補上影響行為的關鍵 method（OnGUI / Preview / 序列化覆寫等）。 -->

### A.4 與其他系統的互動

<!-- TODO: 列出依賴 / 被依賴的類別與系統。 -->

### A.5 已知議題（選填）

<!-- TODO: TODO/FIXME 摘錄、待重構點。 -->
