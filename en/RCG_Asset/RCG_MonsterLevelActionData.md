---
title: 等級化怪物動作 (RCG_MonsterLevelActionData) 說明
description: 把同一招式的不同等級版本綁在一起；難度提升時自動換成更強的版本
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-en
---

> [!WARNING]
> Translation pending — this file needs an English translation.
The original zh-Hant content is included below for reference.


# 等級化怪物動作

> 程式類別名稱：`RCG_MonsterLevelActionData`

## 用途

**把「同一招式的多個等級版本」綁在一起**。例如「攻擊_近距_x1」是 `Lv0`、「攻擊_近距_x2」是 `Lv1`、「攻擊_近距_AOE」是 `Lv2`⋯ 怪物等級提升時系統自動依索引換成更強的版本。也可選用「**單一動作模式**」(`m_UseLevelAsIndex = false`)：只放一個 Action，由動作內部用 HiddenVariable 自行讀等級調整。

繼承自 `RCG_Asset<RCG_MonsterLevelActionData>`。

## 編輯器中的樣貌

```
RCG_MonsterLevelActionData: <ID>
    UseLevelAsIndex (bool)
    MaxSkillLevel
    SkillLevelOffest
    ▼ Actions (UseLevelAsIndex = true)   ← 索引模式：列出每個等級對應的 Action
    ▼ SingleAction (UseLevelAsIndex = false) ← 單一模式：只放一個 Action
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **UseLevelAsIndex** | 是 | `true`：用 `Actions[level]` 取對應動作；`false`：永遠用 `SingleAction`，由內部處理等級 |
| **MaxSkillLevel** | 是 | 等級上限；超過會 clamp（預設 100） |
| **SkillLevelOffest** | — | 等級基準偏移（>0 才生效）；用來**整體提升**這組動作的有效等級 |
| **SingleAction** | UseLevelAsIndex=false | 唯一的 Action（自行用 `m_SkillLevel` 變數調整威力） |
| **Actions** | UseLevelAsIndex=true | 依等級排序的 Action 清單；`Actions[i]` 對應 level i |

## 行為說明

### `GetAction(level)`
1. 加上**全局難度技能等級加成**：`level += DifficultyData.m_EnemySkillLevel`。
2. 若 `SkillLevelOffest > 0`，再加上 offset。
3. clamp 到 `[0, MaxSkillLevel]`。
4. **索引模式** (`UseLevelAsIndex = true`)：
    *   `Actions` 為空 → 回 `Idle`。
    *   否則 `result = Actions[clamp(level, 0, Actions.Count - 1)]`。
5. **單一模式**：直接 `result = SingleAction`。
6. 把 `level` 寫到 `result.m_Action.m_SkillLevel`，再回傳。

### 預覽
*   索引模式：列出所有 Action（可逐個展開預覽）。
*   單一模式：只顯示一行 SingleAction。

## 注意事項

*   **`Actions[i]` 與 `level i` 對應**：列表索引就是等級，沒列到的等級會被 clamp 到最後一個。
*   **`MaxSkillLevel = 100` 比 `Actions.Count` 大時**：超過 `Actions.Count - 1` 的等級全部用最後一個 Action（強度卡在頂）。
*   **`SkillLevelOffest` 只在 >0 時生效**：負值會被忽略；想降低強度要走全局難度設定。
*   **單一模式的 `m_SkillLevel`**：runtime 寫入回傳的是 clone，原 Asset 不受影響。
*   **`m_SingleAction` 的 HiddenVariable** 是設計上的擴充點：在 Action 內部用變數綁定 `SkillLevel` 可實現「同一招式按等級線性增強」而不必開十個 Action。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_MonsterLevelActionData.cs`
*   **繼承自**：`RCG_Asset<RCG_MonsterLevelActionData>`
*   **AssetGroup**：`EditBattleSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_UseLevelAsIndex` | UseLevelAsIndex | `bool` | 預設 `true` |
| `m_MaxSkillLevel` | MaxSkillLevel | `int` | 預設 100 |
| `m_SkillLevelOffest` | SkillLevelOffest | `int` | 預設 0；只在 > 0 時加 |
| `m_SingleAction` | SingleAction | `RCG_MonsterActionGenData` | `Conditional(UseLevelAsIndex == false)` |
| `m_Actions` | Actions | `List<RCG_MonsterActionGenData>` | `Conditional(UseLevelAsIndex == true)` |

### A.3 重要 Method 摘要

*   **`GetAction(int level)`** — 主入口；含難度加成 / offset / clamp / level 寫回 Action。
*   **`Preview`** — 列出所有 Action（索引模式）或單一 Action（單一模式）。

### A.4 與其他系統的互動

*   **`RCG_MonsterActionData`** — 此處掉的 Action 模板。
*   **`RCG_MonsterActionGenData`** — Asset Entry 包裝；預設 `IdleID = "Idle"`。
*   **`RCG_MonsterLevelActionGenData`**（檔內）— 對外引用此資料的型別，自帶 `m_Level` (`IntVariable`)；`GetAction()` 會用此 level 去查 `RCG_MonsterLevelActionData`。
*   **`RCG_DataService.Ins.m_DifficultyData.m_EnemySkillLevel`** — 全局技能等級加成。

### A.5 已知議題

*   `RCG_MonsterLevelActionGenData` 內部建構出錯時會印 LogError 後 fallback 到 `Idle`；隱藏 bug 風險。