---
title: 隨機觸發 說明
description: 隨機選取效果觸發、依機率觸發、或依變數機率觸發三種模式
last_updated: 2026-05-05
target_audience: [Designer, Modder, AI_Agent]
---

# 隨機觸發

> 程式類別名稱：`RCG_RandomTriggerSetting`

## 用途
**讓效果以隨機方式發生**。三種模式：
*   **隨機選 N 個**：從清單中隨機挑 N 個效果觸發
*   **整批依機率**：以一個固定機率決定整批效果是否觸發
*   **整批依變數機率**：與上同，但機率由變數決定（例如「依當前 HP 比例觸發」）

常見用途：賭博類卡、機率型 Buff、多選一隨機效果。

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **RandomTriggerMode** | 是 | 三種模式（見下）。 |
| **RandomPickCount** | RandomPick 模式 | 隨機挑幾個效果觸發。`Conditional` 控制顯示。 |
| **TriggerRate** | TriggerByRate 模式 | 觸發機率（0~100，slider 控制）。 |
| **VariableRate** | TriggerByVariableRate 模式 | 觸發機率變數（其值代表 0~100 的百分比）。 |
| **TriggerSettings** | 是 | 候選效果清單；只有 `IsEnable = true` 的會被納入。 |

### 模式詳細
*   **RandomPick** — 從 enable 的候選中**隨機挑 N 個**觸發；同一效果不重複觸發。
*   **TriggerByRate** — 擲一次 0~99 的隨機數，< `TriggerRate` 則**整批**觸發；否則全跳過。
*   **TriggerByVariableRate** — 同 TriggerByRate，但機率值由 `VariableRate.GetValue` 決定。

## 行為說明
*   描述會列出所有候選效果，配合機率資訊呈現（例如「50% 機率觸發：[A], [B]」或「隨機觸發 1 個：[A] [B] [C]」）。

## 注意事項
*   **RandomPickCount > 候選數**：`RandomPick` 內部使用 `RandomPick(list, count)`；超過候選數的部分會被工具函式處理（通常取上限）。
*   **空清單**：合法但什麼也不會發生。
*   **TriggerByRate 機率 0 / 100**：分別等於「永不觸發」「永遠觸發」 — 屬於極端情況，多半是設計錯誤。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：[`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_RandomTriggerSetting.cs`](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_RandomTriggerSetting.cs)
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_RandomTriggerSetting` → 「隨機觸發」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_RandomTriggerMode` | `RandomTriggerMode` | enum | — | 3 種模式 |
| `m_RandomPickCount` | `RandomPickCount` | `IntVariable` | — | `[Conditional(nameof(m_RandomTriggerMode), false, RandomPick)]` |
| `m_TriggerRate` | `TriggerRate` | `int` | — | `[UCL_IntSlider(0, 100)]` + `[Conditional(... TriggerByRate)]`，預設 50 |
| `m_VariableRate` | `VariableRate` | `IntVariable` | — | `[Conditional(... TriggerByVariableRate)]` |
| `m_TriggerSettings` | `TriggerSettings` | `List<RCG_BattleSetting>` | — | 過濾後 `TriggerSettings = m_TriggerSettings.Where(IsEnable)` |

### A.3 重要 Method 摘要
*   **`AddAction`**：依 `m_RandomTriggerMode`：
    *   `RandomPick` → `RCG_GameManager.Random.RandomPick(allEnabled, count)`，逐一 `AddAction(InsertInOrder)`。
    *   `TriggerByRate` → `Random.Range(0, 100) < m_TriggerRate` → 若觸發，全 enable 子設定逐一 AddAction。
    *   `TriggerByVariableRate` → 同上但機率取 `m_VariableRate.GetValue(iData)`。
*   **`Infos`** → 聚合所有 enable 候選的 `Infos` 並 `AppendIfNotRepeat` 去重。
*   **`GetDescriptionParams`** → 索引 `[0]` 為機率描述（VariableRate 模式為變數描述，其他模式為 `m_TriggerRate.ToString()`，**RandomPick 模式 `[0]` 為佔位但不會用到**）；後接候選子設定參數（前綴 `_{i}`）。
*   **`GetDescriptionFormat`** →
    *   `RandomPick`：候選用 `"[xxx]"` 框並換行串接，模板帶入 `m_RandomPickCount.GetDescription`。
    *   `TriggerByRate / VariableRate`：候選用 `","` 分隔串接，模板帶入 `aParams[0].Item1`。
*   **`GetFusionCandidateSettings`** → 對所有 enable 子的候選聚合（不含自身）。
*   **`GetFusionBaseSetting`**：clone + 重建 `m_TriggerSettings` 為 placeholder 化版本。

### A.4 與其他系統的互動
*   **`RCG_GameManager.Random.RandomPick / Range`**：種子控制的隨機來源（影響重現性）。
*   **`enum.GetLocalizeDes`** (擴充方法)：依 `RandomTriggerMode` 取得對應 i18n 模板字串。

### A.5 已知議題
*   檔頭原本是 `/* AutoHeader Test */` 區塊註解；2026-05-05 G01-5 註解作業時依 [CLAUDE.md](../../../CLAUDE.md) 「禁止 `/* */`」改為 `// AutoHeader Test`。若 AutoHeader 工具依賴此標記，請改為偵測 `//` 形式。
