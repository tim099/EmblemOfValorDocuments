---
title: 資源變化 說明
description: 戰鬥中獲得或消耗資源（金幣、魂力等）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 資源變化

> 程式類別名稱：`RCG_ResourceAlterSetting`

## 用途
**戰鬥中改變玩家持有的資源**（金幣、魂力等）。常見用途：
*   「戰鬥中獲得 50 金幣」
*   「消耗 1 魂力觸發強力卡牌」

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **ResourceAlterMode** | 是 | 變化模式：<br>• **Add** — 獲得（預設）<br>• **Consume** — 消耗（**會卡可玩判定**） |
| **AcquiredResources** | 是 | 資源清單；每項是 `RCG_ResourceGenData`（資源類型 + 數量）。 |

## 行為說明
*   **可玩判定**：`Consume` 模式下，每項資源的當前值必須 ≥ 消耗量。任一不足整張卡不可打出。
*   **觸發**：依模式對 `RCG_DataService.Ins` 的對應資源加 / 扣值。
*   描述：列出每項資源；`Consume` 套 `ConsumeItem` i18n、`Add` 套 `AcquireItem` i18n。

## 注意事項
*   **TODO 提醒**：資源變化時不會即時刷新卡牌的可玩狀態 — 同檔案有 `Todo:資源變化時 要刷新卡牌才能正確判斷目前是否能打出` 標記。
*   **永遠展開**：`[AlwaysExpendOnGUI]`，Inspector 中**永遠不會折疊**。
*   **空清單**：合法，但等於無變化。
*   **與「能量變化」的差別**：能量是戰鬥內每回合刷新的資源；本設定是**跨戰鬥的全域資源**（金幣、魂力）— 兩者不要混淆。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ResourceAlterSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **`[System.Serializable] + [AlwaysExpendOnGUI]`** 標記
*   **i18n 類別名 key**：`RCG_ResourceAlterSetting` → 「資源變化」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_ResourceAlterMode` | `ResourceAlterMode` | enum (檔內) | — | `Add` / `Consume` |
| `m_AcquiredResources` | `AcquiredResources` | `List<RCG_ResourceGenData>` | — | |

### A.3 重要 Method 摘要
*   **`CheckPlayable`**：`Consume` 模式逐一以 `RCG_DataService.Ins.GetResource(type)` 比對 `m_Amount.GetValue`，任一不足回 false；`Add` 模式恆為 true。
*   **`AddAction`**（檔案 100 行外）：依模式對每項資源呼叫加 / 減 API。
*   **`Infos`**：base + 每項資源的類型資訊（`AddIfNotRepeat`）。
*   **`GetDescriptionFormat`** / **`GetDescriptionParams`**（檔內）：用 `, ` 串接資源名，`Consume`/`Add` 模式套不同 i18n key。

### A.4 與其他系統的互動
*   **`RCG_DataService.Ins.GetResource(type)`**：當前資源值查詢。
*   **`RCG_ResourceGenData / RCG_ResourceTypeGenData`**：資源模板。
