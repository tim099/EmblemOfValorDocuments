---
title: 設定戰鬥資訊 說明
description: 修改戰鬥階段的全域變數值（戰鬥內共享資料）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 設定戰鬥資訊

> 程式類別名稱：`RCG_SetBattleDataSetting`

## 用途
**修改戰鬥階段的共享資料**（`RCG_BattleManager.BattleData`）— 在這場戰鬥中跨設定共享的數值容器，類似「戰鬥內全域變數」。常見用途：
*   「累積造成的總傷害」
*   「記錄玩家本場使用了多少次某動作」

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **RuntimeDataOperator** | 是 | 內嵌的 `RCG_RuntimeDataSetter`（指定要設定的欄位 + 值 + 計算公式）。預設綁定 `RCG_RuntimeStructGenData.BattleDataID`（戰鬥資料結構）。 |

## 行為說明
*   觸發時呼叫 `RCG_BattleManager.BattleData.SetValue(m_RuntimeDataOperator)`，把計算結果寫入指定欄位。
*   描述完全代理到 `m_RuntimeDataOperator.GetDescription`。
*   `GetShortName` 預設為 `"SetBattleData:{operator.GetShortName()}"`。

## 注意事項
*   **資料結構需事先定義**：`RuntimeDataOperator` 要設定哪個欄位，由 `RCG_RuntimeStructGenData.BattleDataID` 結構定義 — 想新增欄位請查該資料結構。
*   **不能融合**：本設定不參與卡片融合（`GetFusionBaseSetting` 回傳 null）。
*   **戰鬥結束後資料消失**：BattleData 是戰鬥內的；跨戰鬥要記錄請改用「**資源變化**」或全域 RuntimeData。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_SetBattleDataSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_SetBattleDataSetting` → 「設定戰鬥資訊」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_RuntimeDataOperator` | `RuntimeDataOperator` | `RCG_RuntimeDataSetter` | — | 預設綁 `RCG_RuntimeStructGenData.BattleDataID` |

### A.3 重要 Method 摘要
*   **`AddAction`**：`RCG_BattleManager.BattleData.SetValue(m_RuntimeDataOperator)`。
*   **`GetDescription / GetDescriptionFormat / GetShortName`**：代理到 `m_RuntimeDataOperator`。
*   **`GetFusionCandidateSettings`** → 空清單；**`GetFusionBaseSetting`** → null（不參與融合）。

### A.4 與其他系統的互動
*   **`RCG_BattleManager.BattleData`**：戰鬥內共享資料容器。
*   **`RCG_RuntimeDataSetter`**：欄位設定器，含公式計算。
*   **`RCG_RuntimeStructGenData.BattleDataID`**：對應的資料結構模板。
