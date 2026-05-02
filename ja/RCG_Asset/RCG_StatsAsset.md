---
title: 統計資料資產 (RCG_StatsAsset) 說明
description: 對應「遊戲內統計值」與 Steam 統計的橋樑：擊殺數、傷害總量等可累加數值
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-ja
---

> [!WARNING]
> 翻訳待機中 — このファイルは日本語翻訳が必要です。
参考用に zh-Hant 原文を以下に掲載しています。


# 統計資料資產

> 程式類別名稱：`RCG_StatsAsset`

## 用途

**遊戲內統計值的定義**。例如「累計擊殺數」「累計打出卡牌數」「累計通關次數」這種**可累加的歷史數值**，用 `RCG_StatsAsset` 定義；可選擇是否串接 Steam Stats 自動上報。

繼承自 `RCG_Asset<RCG_StatsAsset>`。

## 編輯器中的樣貌

```
RCG_StatsAsset: <ID>
    HasSteamStat         ← 是否串接 Steam 統計
    SteamUserStat        (HasSteamStat=true) ← 對應的 Steam 統計 ID
    GameStatsType        ← 對應的遊戲內統計枚舉值（None / KillCount / ... 等）
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **HasSteamStat** | — | 是否串接 Steam 統計 |
| **SteamUserStat** | HasSteamStat=true | 對應的 Steam 統計 entry |
| **GameStatsType** | 是 | 遊戲內統計類型（`GameStatsType` enum） |

## 行為說明

### Editor 內 `Create BuiltIn Stats` 工具
編輯器頁面上方有按鈕：對 `GameStatsType` 列舉內所有非 `None` 值，自動建立缺漏的 Asset（ID 用 enum 字串名）並把 `m_GameStatsType` 設好。**便於同步程式端 enum 與 Asset 端**。

## 注意事項

*   **`GameStatsType` enum 是程式控制**：新增統計類型要先改 enum，再用編輯器工具補 Asset。
*   **預設 ID `None`**（`RCG_StatsEntry.DefaultID`）：表示「無統計」；不要拿來作為實際統計的 ID。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_StatsAsset.cs`
*   **繼承自**：`RCG_Asset<RCG_StatsAsset>`
*   **AssetGroup**：`EditGameSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_HasSteamStat` | HasSteamStat | `bool` | |
| `m_SteamUserStat` | SteamUserStat | `UCL_SteamUserStatAssetEntry` | `Conditional(HasSteamStat)` |
| `m_GameStatsType` | GameStatsType | `GameStatsType` enum | |

### A.3 重要 Method / 工具

*   **`CreateSelectAssetPage`** — `RCG_StatsAssetEditorPage.Create()`，編輯器分頁。
*   **`Create BuiltIn Stats`**（Editor only） — 自動補齊缺漏的 stats Asset。

### A.4 與其他系統的互動

*   **`UCL_SteamUserStatAssetEntry`** — Steam SDK 串接。
*   **`GameStatsType` (enum)** — 程式端統計類型枚舉。
*   **`RCG_StatsEntry`** — Asset Entry；預設 `None`。