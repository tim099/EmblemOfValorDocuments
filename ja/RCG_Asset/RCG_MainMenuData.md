---
title: 主選單資料 (RCG_MainMenuData) 說明
description: 主選單畫面的設定容器（背景、按鈕、版面）；目前內容主要由 RCG_MainMenuSetting 處理
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-ja
---

> [!WARNING]
> 翻訳待機中 — このファイルは日本語翻訳が必要です。
参考用に zh-Hant 原文を以下に掲載しています。


# 主選單資料

> 程式類別名稱：`RCG_MainMenuData`

## 用途

**主選單畫面的設定**——包裝一個 `RCG_MainMenuSetting`，讓不同 build / 版本能套不同主選單（例如展覽版用簡化選單、Demo 版鎖部分按鈕）。

繼承自 `RCG_Asset<RCG_MainMenuData>`。

## 編輯器中的樣貌

```
RCG_MainMenuData: <ID = Default>
    MainMenuSetting   ← 實際的主選單設定（背景、按鈕、版面、過場⋯）
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **MainMenuSetting** | 是 | 主選單設定本體（`RCG_MainMenuSetting`），含背景、按鈕、版面細節等 |

## 行為說明

本類別本身只是個薄薄的包裝；實際邏輯都在 `RCG_MainMenuSetting`（不是 RCG_Asset 子類，是純資料容器）。

### Static 入口
*   `RCG_MainMenuData.CreateInstance()` — 從 Asset 取 `Default` 實例（強制重讀）。
*   `RCG_MainMenuData.Ins` 已被註解；引用點改走 `RCG_GameSettingData.m_MainMenu`。

### 引用關係
`RCG_GameSettingData.m_MainMenu`（型別 `RCG_MainMenuEntry`）引用此資料；不同遊戲版本可在 GameSettingData 上指定不同 MainMenuData。

## 注意事項

*   **ID 預設為 `Default`**：本類別預期單一 Asset；要做版本變體就建多個 ID。
*   **`m_MainMenuSetting` 標 `[AlwaysExpendOnGUI]`**：Inspector 內預設展開，方便編輯。
*   **`Ins` static 已被註解**：取資料統一從 GameSettingData 走，不要直接 `RCG_MainMenuData.Util.GetData(...)`。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_MainMenuData.cs`
*   **繼承自**：`RCG_Asset<RCG_MainMenuData>`
*   **AssetGroup**：`EditGameSetting`
*   **常數**：`DefaultID = "Default"`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_MainMenuSetting` | MainMenuSetting | `RCG_MainMenuSetting` | `[AlwaysExpendOnGUI]` |

### A.3 重要 Method

*   **`CreateInstance()` (static)** — `Util.GetData(DefaultID, false)`。
*   建構式預設 `ID = DefaultID`。

### A.4 與其他系統的互動

*   **`RCG_MainMenuSetting`** — 實際內容。
*   **`RCG_MainMenuEntry`** — Asset Entry 包裝；`RCG_GameSettingData.m_MainMenu` 用此型別引用。

### A.5 已知議題

*   `Ins` static 已被註解，標示「請走 GameSettingData 取」的設計轉變。