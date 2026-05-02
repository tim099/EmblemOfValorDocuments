---
title: 設定目標 說明
description: 自動依規則重新選取目標（不彈 UI）；會覆蓋原本的目標清單
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 設定目標

> 程式類別名稱：`RCG_SetTargetSetting`

## 用途
**自動依規則重新選定目標**（不開 UI）。常見用途：
*   「將攻擊轉移到血量最低的敵人」
*   「重定向到後排目標」
*   AI 觸發效果時自動指定目標

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **UseOldSelectUnitRule** | — | 預設打勾（向前兼容）。打勾使用舊版 `RCG_SelectTargetData`；不打勾用新版 `RCG_TargetSelectRule`。 |
| **SelectTarget** | UseOldSelectUnitRule=true 時 | 舊版目標規則（`RCG_SelectTargetData`）。 |
| **SelectUnitRule** | UseOldSelectUnitRule=false 時 | 新版目標規則（`RCG_TargetSelectRule`）。 |

## 行為說明
*   依設定的規則自動解析目標清單，覆蓋 `iData.Targets`。
*   不開 UI、不需玩家輸入；適合 AI 與自動化效果。
*   描述顯示對應規則的描述（短名）。

## 注意事項
*   **與「選取目標」的差別**：本設定**不出 UI**；「選取目標」要玩家點擊。
*   **新規則 vs 舊規則**：新版 `RCG_TargetSelectRule` 表達能力更強；舊版 `RCG_SelectTargetData` 僅為兼容保留。新資料請用新版（取消打勾 `UseOldSelectUnitRule`）。
*   **同附錄中 TODO**：程式內有 `// TODO: Test QWQ2`，新版規則實際使用尚需驗證。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_SetTargetSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_SetTargetSetting` → 「設定目標」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_UseOldSelectUnitRule` | `UseOldSelectUnitRule` | `bool` | — | 預設 `true`（向前兼容） |
| `m_SelectUnitRule` | `SelectUnitRule` | `RCG_TargetSelectRule` | — | `[Conditional("m_UseOldSelectUnitRule", false)]` |
| `m_SelectTarget` | `SelectTarget` | `RCG_SelectTargetData` | — | `[Conditional("m_UseOldSelectUnitRule", true)]` |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：依 `m_UseOldSelectUnitRule` 走 `m_SelectUnitRule.SelectTargets(iData)` 或 `m_SelectTarget.GetTargets(iData)`，賦值到 `iData.Targets`。
*   **`GetDescription`**：代理到對應規則的 `Description / GetShortName`。

### A.4 與其他系統的互動
*   **`RCG_TargetSelectRule`**：新版目標規則容器。
*   **`RCG_SelectTargetData`**：舊版目標選擇器；廣泛被其他 setting 使用。

### A.5 已知議題
*   `// TODO: Test QWQ2` — 新版規則尚需測試驗證。
