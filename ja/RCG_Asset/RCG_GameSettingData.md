---
title: 遊戲設定 (RCG_GameSettingData) 說明
description: 不同版本（Demo / 正式版）的遊戲級設定：主選單、最大解鎖等級、教學重置、商店刷新成本
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-ja
---

> [!WARNING]
> 翻訳待機中 — このファイルは日本語翻訳が必要です。
参考用に zh-Hant 原文を以下に掲載しています。


# 遊戲設定

> 程式類別名稱：`RCG_GameSettingData`

## 用途

**版本級的遊戲設定**——讓不同 build（Demo / 正式版 / 展覽版）能套不同設定。系統會用 `Application.version` 當作 ID 從中找對應 Asset；如果沒有同 ID 的，fallback 到 `Default`。

繼承自 `RCG_Asset<RCG_GameSettingData>`。

## 編輯器中的樣貌

```
RCG_GameSettingData: <ID = Application.version>
    GameVersion          ← Default / Demo
    MainMenu             ← 主選單設定引用
    MaxUnlockLevel       ← 最大解鎖等級上限
    ResetTutorialOnNewGame ← 是否每次開新遊戲重置教學
    RefreshBlessingShopPrice ← 祝福商店刷新成本基數
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **GameVersion** | 是 | `Default`（普通版）或 `Demo`（鎖定部分功能 / 解鎖等級） |
| **MainMenu** | 是 | 主選單設定的引用（`RCG_MainMenuEntry`） |
| **MaxUnlockLevel** | 是 | 最大解鎖等級；超過後等級不再成長（預設 1000） |
| **ResetTutorialOnNewGame** | — | 每次開新遊戲時重置教學，**展覽機**用 |
| **RefreshBlessingShopPrice** | 是 | 刷新祝福商店成本基數；總成本 = (剩餘可購買物品數 - 4) × 此值（預設 10） |

## 行為說明

### Asset 取得 (`Ins`)
`RCG_GameSettingData.Ins` 用 `Application.version` 當 ID 從 Asset 庫取對應實例。**因此 Asset 命名要符合 build 版本號**（如 `0.1.0` / `0.2.0-demo`）；若沒精確匹配，`Util.GetData(version, useDefaultIfMissing = true)` 會 fallback 到 `Default`。

### Demo 版限制
`GameVersion = Demo` 時遊戲邏輯會自動鎖定一些功能（例如：強制 `MaxUnlockLevel` 為較低值、禁用某些章節）。具體鎖定邏輯散落在各 manager；本檔只是個 flag。

## 注意事項

*   **ID 必須對應 Application.version**：建版號改了，舊的 Asset 不會自動套用（需新增同名 Asset 或讓 Default 涵蓋）。
*   **`m_GameSetting` 已被註解**：原本 `RCG_GameInitData.m_GameSetting` 引用此 Asset，現在改用 `Application.version` 自動查找。
*   **`RefreshBlessingShopPrice` 公式**：含負數 case（剩餘 ≤ 4 時成本為 0 或負數），實際價格邏輯需 clamp 到 ≥ 0。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_GameSettingData.cs`
*   **繼承自**：`RCG_Asset<RCG_GameSettingData>`
*   **AssetGroup**：`EditGameSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_GameVersion` | GameVersion | `GameVersion` enum | `Default` / `Demo` |
| `m_MainMenu` | MainMenu | `RCG_MainMenuEntry` | |
| `m_MaxUnlockLevel` | MaxUnlockLevel | `int` | 預設 1000 |
| `m_ResetTutorialOnNewGame` | ResetTutorialOnNewGame | `bool` | 預設 false |
| `m_RefreshBlessingShopPrice` | RefreshBlessingShopPrice | `int` | 預設 10 |

### A.3 重要 Method

*   **`Ins` (static)** — `Util.GetData(Application.version, useDefaultIfMissing = true)`。

### A.4 與其他系統的互動

*   **`RCG_MainMenuData`** — `m_MainMenu` 引用的主選單設定。
*   **`RCG_GameInitData`** — 遊戲初始資料；舊版 `m_GameSetting` 欄位已註解，現由 `Application.version` 自動關聯。
*   **`RCG_GameSettingGenData`** — Asset Entry；預設 ID = `"Default"`。

### A.5 已知議題

*   `Ins` 中的 `Debug.Log("Application.version: ...")` 已標記 "log!!"——應該是 dev 用，release 前要移除。