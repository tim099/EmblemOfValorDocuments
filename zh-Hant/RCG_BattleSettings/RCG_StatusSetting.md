---
title: 狀態 說明
description: 給予 / 減少 / 移除 / 轉換 / 吸收 / 隨機抽取狀態的全功能設定
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 狀態

> 程式類別名稱：`RCG_StatusSetting`

## 用途
**遊戲中的「狀態」核心設定**。給予層數、減少層數、移除整個狀態、按類型批次操作、A 轉 B 等多種模式。是除了攻擊以外**最常用的設定之一**。

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **目標** (`Target`) | 是 | 狀態套用的目標選擇器。 |
| **StatusModifiedType** | 是 | 變動類型，**10 種模式**（見下表）。 |
| **狀態** (`Status`) | Add/Decrease/Remove/Convert/Set 模式必填 | 要操作的狀態類型（`RCG_CustomStatusGenData`）。 |
| **StatusDropPool** | StatusDropPool 模式必填 | 從哪個狀態池隨機抽。 |
| **ConvertStatus** | Convert 模式必填 | 要轉換成的目標狀態。 |
| **ConvertLayer** | Convert 模式 | 多少層觸發一次轉換（預設 10）。 |
| **ConvertRatio** | Convert 模式 | 轉換比例（預設 1）。例如 ConvertLayer=10 + ConvertRatio=3 → 「每 10 層恍惚 → 3 層暈眩」。 |
| **StatusTags** | RemoveStatusOfTargetType/AbsorbStatusOfTargetType 模式 | 要操作的狀態類型清單（如「全部 Debuff」）。 |
| **值** (`Amount`) | Add/Decrease/Set 模式 | 變動層數（變數可變）。 |
| **DescriptionType** | — | 描述句型：`Default` / `Then`。 |
| **詳細設定** (`DetailSetting`) | — | 含 `SkipAnim`（跳過演出，直接設定層數）。 |

### StatusModifiedType 模式
| 值 | 行為 |
|---|---|
| **AddStatusLayer** | 新增層數（預設） |
| **DecreaseStatusLayer** | 減少層數 |
| **RemoveStatus** | 移除指定狀態（不論層數） |
| **RemoveAllStatus** | 移除全部狀態 |
| **RemoveAllDebuff** | 移除全部負面狀態 |
| **RemoveStatusOfTargetType** | 依 StatusTags 清單批次移除 |
| **AbsorbStatusOfTargetType** | 把目標的某類狀態吸到自身 |
| **ConvertStatus** | A 狀態轉換為 B 狀態（依 Layer / Ratio） |
| **SetStatusLayer** | 設定層數為指定值 |
| **StatusDropPool** | 從狀態池隨機抽一個狀態給予 |

## 行為說明
*   依模式對 `m_Target.GetTargets(iData)` 套用。
*   有 `Amount` 的模式支援變數綁定（如「依手牌數加敏捷」）。
*   `Convert` 模式：每 `ConvertLayer` 層消耗，產生 `ConvertRatio` 層的 ConvertStatus。
*   描述格式因模式而異 — UI 顯示 i18n 名稱 + 圖示。

### 卡片融合
僅 `AddStatusLayer / DecreaseStatusLayer / SetStatusLayer` 三個模式可融合（Amount 加總）。其他模式不可融合。

## 注意事項
*   **AbsorbStatusOfTargetType**：吸到「**自身**」（不是目標選擇器）— 注意是固定行為。
*   **狀態池隨機**：`StatusDropPool` 受戰鬥種子影響；同種子下重玩會抽到一樣的狀態。
*   **SkipAnim**：跳過演出可加快結算；但玩家可能來不及看到狀態變化。重要狀態建議不打勾。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_StatusSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **`[System.Serializable]`** 標記
*   **i18n 類別名 key**：`RCG_StatusSetting` → 「狀態」
*   **同檔內巢狀類**：`DetailSetting`（含 `m_SkipAnim`）/ `DescriptionType` enum (`Default`, `Then`)
*   **同檔頂層 enum**：`StatusModifiedType` (10 個值)

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_Target` | 目標 | `RCG_SelectTargetData` | `Target` | |
| `m_StatusModifiedType` | `StatusModifiedType` | `StatusModifiedType` | — | 預設 `AddStatusLayer` |
| `m_Status` | 狀態 | `RCG_CustomStatusGenData` | `Status` | `[Conditional(...5 種模式)]` |
| `m_StatusDropPool` | `StatusDropPool` | `RCG_StatusDropPoolGenData` | — | `[Conditional(... StatusDropPool)]` |
| `m_ConvertStatus` | `ConvertStatus` | `RCG_CustomStatusGenData` | — | `[Conditional(... ConvertStatus)]` |
| `m_ConvertLayer` | `ConvertLayer` | `IntVariable` | — | `[Conditional(... ConvertStatus)]`，預設 10 |
| `m_ConvertRatio` | `ConvertRatio` | `IntVariable` | — | `[Conditional(... ConvertStatus)]`，預設 1 |
| `m_StatusTags` | `StatusTags` | `List<RCG_StatusTagGenData>` | — | `[Conditional(... 兩個 OfTargetType 模式)]` |
| `m_Amount` | 值 | `IntVariable` | `Amount` | `[UCL_FieldOnGUI]` + `[Conditional(... 3 種需要 Amount 的模式)]` |
| `m_DescriptionType` | `DescriptionType` | `DescriptionType` (檔內 enum) | `DescriptionType` | |
| `m_DetailSetting` | 詳細設定 | `DetailSetting` (檔內巢狀類) | `DetailSetting` | 含 `m_SkipAnim` |

### A.3 重要 Method 摘要
*   **`Fusion`**：要求 `m_StatusModifiedType` 同為三種 Layer 操作之一；clone + `IntVariable.FuseAdd(m_Amount)`。
*   **`GetShortName`**：依 StatusModifiedType 走分支：直接 i18n / `GetDescription()` / `m_StatusDropPool.GetData().GetShortName()`。
*   **`AddAction`** / **`Infos`**（檔案 200 行外）：依模式套用實際邏輯與資訊聚合。

### A.4 與其他系統的互動
*   **`RCG_UnitStatus`**：實際的狀態容器（`GetStatusLayer` / `m_Status` 字典等）。
*   **`RCG_StatusTagGenData.StatusDebuff / StatusDot`**：類型標記，用於批次移除。
*   **`CreateAction.StatusAction`**：實際的 Action 建構工具（被 `RCG_ConsumeStatusSetting` 等共用）。

### A.5 已知議題
*   `CanEnhence / Enhence` 已註解（強化系統重構中）。
