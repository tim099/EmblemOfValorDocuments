---
title: 怪物動作資料 (RCG_MonsterActionData) 說明
description: 怪物可使用的單一招式 / 動作模板：選擇規則、效果、圖示、描述
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 怪物動作資料

> 程式類別名稱：`RCG_MonsterActionData`

## 用途

**怪物可使用的單一招式 / 動作模板**。例如「揮砍」「召喚從者」「自爆」。每個 Action 內含：
*   選目標規則（哪些單位可被選為目標）
*   效果（傷害、buff、召喚⋯）
*   圖示
*   描述（自動 / 手動）

`RCG_UnitData` 的 `MonsterStates` 引用一組 `RCG_MonsterActionData` 作為該狀態下可用招式池。

繼承自 `RCG_Asset<RCG_MonsterActionData>`。

## 編輯器中的樣貌

```
RCG_MonsterActionData: <ID>
    Action (m_Action)
        SelectUnitRule         ← 選目標規則
        Effect 結構            ← 實際效果
        Icon                   ← 預覽圖示
        OverrideDescription    ← 是否手動覆寫描述
        UnitAction             ← 真正執行的動作
        Infos                  ← Tooltip 列出的附加資訊
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Action** | 是 | 內部 `RCG_MonsterAction`，包含此招式所有資料 |

`RCG_MonsterAction` 內部：

| 子欄位 | 說明 |
|---|---|
| **SelectUnitRule** | 選目標規則（`RandomEnemy` / `Friend` / 自身 / 站位 N 之類） |
| **OverrideDescription** | 是否使用 `m_UnitAction` 的描述覆蓋自動描述 |
| **UnitAction** | 招式真正執行的動作（傷害公式、效果序列） |
| **Icon** | 預覽 / Tooltip 顯示用的圖示 |
| **SkillLevel** | 招式等級（runtime 由 `RCG_MonsterLevelActionData.GetAction` 動態填入） |

## 行為說明

### 選目標 → 觸發 → 描述
1. 戰鬥中怪物選定要使用此 Action 後，依 `SelectUnitRule` 找出目標。
2. 套用 `m_UnitAction` 的效果到目標。
3. UI Tooltip 上顯示 `Icon` + `GetDescription()` + `Infos` 補充標籤資訊。

### 自動 vs 覆寫描述
`OverrideDescription = true` 時，預覽會在主描述下面**額外**附加 `m_UnitAction.GetDescription()`（兩段都顯示）。

### 預設 ID
透過 `RCG_MonsterActionGenData.IdleID` (`"Idle"`) 取得「無動作」的預設 Action；任何沒設動作的單位回合會跑這個。

## 注意事項

*   **SkillLevel 由外部填入**：`RCG_MonsterLevelActionData.GetAction(level)` 會把等級寫到 `m_SkillLevel`，本資料只是模板，runtime 取的是 clone 後的副本。
*   **OverrideDescription 雙顯示**：可能造成 Tooltip 過長，必要時關閉。
*   **Idle 動作不要刪**：許多 fallback 路徑會回傳 `RCG_MonsterActionGenData.Idle.GetData()`，缺了這個 ID 會 NRE。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_MonsterActionData.cs`
*   **繼承自**：`RCG_Asset<RCG_MonsterActionData>`
*   **AssetGroup**：`EditBattleSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_Action` | Action | `RCG_MonsterAction` | 主要資料容器 |

### A.3 重要 Method 摘要

*   **`Preview`** — 編輯器渲染：圖示 + 選目標規則 + 描述 + Effects 描述（若 OverrideDescription）+ Infos。
*   建構式預設 `ID = "New Action"`。

### A.4 與其他系統的互動

*   **`RCG_MonsterAction`** — 主要資料模型；含 `m_SelectUnitRule` / `m_UnitAction` / `m_Icon` / `m_SkillLevel`。
*   **`RCG_MonsterActionGenData`**（檔內）— Asset Entry 包裝；`Idle` 為系統預設。
*   **`RCG_MonsterLevelActionData`** — 對 Action 包裝以支援等級分級。
*   **`RCG_UnitData.m_MonsterStates`** — 引用 Actions 的容器。

### A.5 已知議題

*   有被註解掉的 `DeserializeFromJson` 容錯邏輯（`m_ID == "AttackEffect"` → `DefaultID`），標示舊版 ID 遷移歷史。
