---
title: 暗霧設定 (RCG_DarkMistSettingsData) 說明
description: 暗霧機制的等級數據：不同暗霧等級下對玩家的弱化效果與對敵方的強化效果
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-ja
---

> [!WARNING]
> 翻訳待機中 — このファイルは日本語翻訳が必要です。
参考用に zh-Hant 原文を以下に掲載しています。


# 暗霧設定

> 程式類別名稱：`RCG_DarkMistSettingsData`

## 用途

**暗霧機制的等級數據**。暗霧是隨遊戲進行不斷加深的環境壓力，等級越高玩家越弱、敵方越強。本資料定義「在不同暗霧等級下，雙方獲得什麼效果」——通常是以兩個共用的 `CustomStatus`（`DarkMistBuff` / `DarkMistDebuff`）為載體，戰鬥開始時自動套用。

繼承自 `RCG_Asset<RCG_DarkMistSettingsData>`。

## 編輯器中的樣貌

```
RCG_DarkMistSettingsData: <ID>
    DarkMistBuffGenData      ← 給敵方的強化狀態 ID
    DarkMistDebuffGenData    ← 給玩家的弱化狀態 ID
    DarkMistDatas            ← 各等級資料清單（DarkMistLevelData）
        TriggerLevel
        DarkMistBuffStatusData    ← 此等級下的強化效果
        DarkMistDebuffStatusData  ← 此等級下的弱化效果
        Effects                   ← 戰鬥開始時觸發的 BattleSetting
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **DarkMistBuffGenData** | 是 | 共用的「敵方強化狀態」ID（預設 `DarkMistBuff`） |
| **DarkMistDebuffGenData** | 是 | 共用的「玩家弱化狀態」ID（預設 `DarkMistDebuff`） |
| **DarkMistDatas** | 是 | 各等級資料清單（按 `TriggerLevel` 排序，**最後元素優先**） |

每個 `DarkMistLevelData` 內含：

| 子欄位 | 說明 |
|---|---|
| **TriggerLevel** | 此等級觸發門檻（暗霧等級 ≥ 此值才使用本筆設定） |
| **DarkMistBuffStatusData** | 此等級下的敵方強化效果（覆寫到 `DarkMistBuff` Status 上） |
| **DarkMistDebuffStatusData** | 此等級下的玩家弱化效果（覆寫到 `DarkMistDebuff` Status 上） |
| **Effects** | 戰鬥開始時觸發的 `RCG_BattleSetting` 序列 |

## 行為說明

### 取得當前等級資料 (`GetCurrentDarkMistLevelData`)
從清單**末端往前**找第一個 `TriggerLevel ≤ iLevel` 的資料；因此清單必須**按 TriggerLevel 升冪排序**才能正確匹配（例如 `[0, 5, 10, 20]`）。

### 觸發 (`OnTriggerEffect`)
找到對應等級資料後，把該資料的 `m_Effects` 全部加到動作佇列。

### 覆寫狀態 (`UpdateDarkMistEffect`)
把當前等級的 `DarkMistBuffStatusData` 寫入共用 `DarkMistBuff` Status 的 `m_Effects`——這是**就地修改全局 Status Asset**，每次戰鬥開始前重設一次。

## 注意事項

*   **DarkMistDatas 必須升冪排序**（依 `TriggerLevel`）：取資料邏輯依賴此前提，逆序或亂序會找錯級。
*   **共用 Status 是動態覆寫**：`DarkMistBuff` / `DarkMistDebuff` 兩個 `RCG_CustomStatusData` 的 effects 會被 runtime 覆寫；不要手動修這兩個 Asset。
*   **被多個來源使用**：`RCG_GameInitData.m_DarkMistSettings` 是清單，可有多套（例：難度不同套不同暗霧）。
*   **Preview 已被註解掉**：編輯器內預覽用基底類預設繪製。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_DarkMistSettingsData.cs`
*   **繼承自**：`RCG_Asset<RCG_DarkMistSettingsData>`
*   **AssetGroup**：`EditQuestSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_DarkMistBuffGenData` | DarkMistBuffGenData | `RCG_CustomStatusGenData` | 預設 `"DarkMistBuff"` |
| `m_DarkMistDebuffGenData` | DarkMistDebuffGenData | `RCG_CustomStatusGenData` | 預設 `"DarkMistDebuff"` |
| `m_DarkMistDatas` | DarkMistDatas | `List<DarkMistLevelData>` | 各等級資料 |

### A.3 重要 Method

*   **`OnTriggerEffect(data, level)`** — 找對應等級資料 → 套用 effects。
*   **`GetCurrentDarkMistLevelData(level)`** — 從尾端往前查 `TriggerLevel ≤ level` 的元素。
*   **`UpdateDarkMistEffect(level)`** — 覆寫共用 Buff Status 的 effects。
*   **`SetDarkMistBuff(status)`** — 用 JSON 序列化複寫 Buff 整個 Asset。

### A.4 與其他系統的互動

*   **`RCG_CustomStatusData`** — `DarkMistBuff` / `DarkMistDebuff` 兩個全局狀態。
*   **`RCG_GameInitData.m_DarkMistSettings`** — 引用此資料的清單。
*   **`RCG_BattleSetting`** — `m_Effects` 的元素型別。

### A.5 已知議題

*   `Preview` 完整實作已註解；改用基底類預設繪製。
*   清單必須手動排序，缺乏自動排序保證；若資料順序錯亂會找錯等級。