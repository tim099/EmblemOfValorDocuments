---
title: 協力 說明
description: 隊伍中符合專精需求的角色達到指定數量時觸發效果；可作為條件觸發或數量變數兩種模式
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 協力

> 程式類別名稱：`RCG_CollaborationSetting`

## 用途
**團隊合擊機制**：依「**我方有幾個角色符合特定專精**」決定是否觸發效果，或把符合數量作為變數使用。常見用途：
*   「若隊伍中有 2 名魔法師，造成額外傷害」
*   「依符合人數決定治療量」

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **CollaborationType** | 是 | 模式：<br>• **Default** — 達到 `CollaborationCount` 才觸發<br>• **Variable** — 自動觸發；符合人數作為變數 X |
| **CollaborationCount** | Default 模式 | 至少要有幾個角色符合條件（預設 2）。 |
| **RequireSkills** | 否 | 專精需求清單；空清單代表不檢查（任何隊員都算）。 |
| **CollaborationVariable** | Variable 模式 | 變數名稱（預設 `X`）；存入符合人數，後續設定可引用。 |
| **CollaborationSetting** | 是 | 觸發後執行的「**組合效果**」。 |

## 行為說明
*   **Default 模式**：
    *   檢查我方角色中符合 `RequireSkills`（任一即可）的數量是否 ≥ `CollaborationCount`。
    *   達標 → 執行 `CollaborationSetting`；未達標 → 不觸發。
*   **Variable 模式**：
    *   不檢查門檻，**直接觸發**。
    *   把符合人數寫入 `CollaborationVariable`，`CollaborationSetting` 內部可引用該變數做「依人數縮放」的效果。
*   描述包含：條件描述（i18n key `CollaborationTypeInfo_*`）+ 子設定描述。

## 注意事項
*   **RequireSkills 為空 + Default 模式**：等於「全員都算符合」，協力人數總是隊伍人數 — 確認設計意圖。
*   **變數命名衝突**：多個設定都用 `X`，後者會覆蓋前者；用語意化的變數名（如 `CollabCount`）。
*   **觸發子效果中可疊加新動作**：`CollaborationSetting` 是個 `RCG_CombineSetting`，可組合多種效果，但**注意巢狀深度**對描述可讀性的影響。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CollaborationSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_CollaborationSetting` → 「協力」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_CollaborationType` | `CollaborationType` | enum | — | `Default` / `Variable` |
| `m_CollaborationCount` | `CollaborationCount` | `int` | — | 預設 2 |
| `m_RequireSkills` | `RequireSkills` | `List<RCG_SkillTagGenData>` | — | |
| `m_CollaborationVariable` | `CollaborationVariable` | `string` | — | `[Conditional("m_CollaborationType", false, Variable)]`；預設 `"X"` |
| `m_CollaborationSetting` | `CollaborationSetting` | `RCG_CombineSetting` | — | `[AlwaysExpendOnGUI]` |

### A.3 重要 Method 摘要
*   **`Infos`**：自身 description（含 `GetCollaborationDes()` + `CollaborationTypeInfo_*` i18n）+ `m_CollaborationSetting.Infos`。
*   **`GetCollaborationRequireSkillDes()` (private)**：空清單 → `CollaborationRequireSkills_Any`；否則 `CollaborationRequireSkills_Specify`。
*   **`AddAction`**（檔案 100 行外）：解析符合人數，依 CollaborationType 決定是否觸發 `m_CollaborationSetting`。

### A.4 與其他系統的互動
*   **`RCG_SkillTagGenData`**：專精標籤模板。
*   **`RCG_CombineSetting`**：子效果容器。
