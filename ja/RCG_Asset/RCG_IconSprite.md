---
title: TMP 圖示資料 (RCG_IconSprite) 說明
description: 給 TextMeshPro 用的 inline 圖示資源；UI 描述中的小圖示（HP / 護甲 / 卡牌等）來源
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-ja
---

> [!WARNING]
> 翻訳待機中 — このファイルは日本語翻訳が必要です。
参考用に zh-Hant 原文を以下に掲載しています。


# TMP 圖示資料

> 程式類別名稱：`RCG_IconSprite`

## 用途

**給 TextMeshPro 用的 inline 圖示資源**。卡牌描述、狀態說明、戰鬥日誌中常出現「+5 HP」「-3 護甲」之類的文字，HP / 護甲等小圖示就是用 TMP 的 `<sprite name=...>` 標籤插入；本 Asset 就是這些圖示的來源。系統自動把所有 `RCG_IconSprite` 打包成一張 `TMP_SpriteAsset` 給文字渲染。

繼承自 `RCG_Asset<RCG_IconSprite>`。

## 編輯器中的樣貌

```
RCG_IconSprite: <ID>
    Icon       ← 圖示 sprite
    Scale      ← 縮放（TMP 內顯示用）
    BearingX   ← 水平基線偏移
    BearingY   ← 垂直基線偏移（預設 0.85）
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Icon** | 是 | 圖示 sprite（`RCG_SpriteData`），預設 `Status_BloodDrain` |
| **Scale** | 是 | 在 TMP 內的縮放（預設 1） |
| **BearingX** | — | 水平基線偏移 |
| **BearingY** | — | 垂直基線偏移（預設 0.85，對應 TMP 文字基線） |

## 行為說明

### TMPKey
每個 IconSprite 自動產生 TMP 標籤：`<sprite name={ID}>`。在文字裡用這個就能 inline 顯示對應圖示。

### 自動打包成 TMP_SpriteAsset
`InitSpriteAsset(token)` 在啟動時掃所有 `RCG_IconSprite` 並打包成單一 `TMP_SpriteAsset`；模組重新載入時 (`UCL_ModuleService.OnLoadedModule`) 會自動 refresh。

### `Save()` 觸發 RefreshSpriteAsset
編輯器內存檔本 Asset 會自動觸發 `RefreshSpriteAsset()` 重建打包，立即看到變更。

### 內建 static 引用
*   `EffectIcon_Target` / `EffectIcon_CountDown` / `EffectIcon_Armor` / `EffectIcon_Health` / `EffectIcon_Card` / `EffectIcon_Die` / `EffectIcon_Energy` / `EffectIcon_Magnifier`
這些是程式內常用的圖示，直接用 static property 取得。

### Editor 工具
*   **`Output sprite sheet`**（編輯器專用）：匯出整套 sprite sheet 圖（合圖檔）。
*   **`Refresh SpriteAsset`**：手動重新打包。

## 注意事項

*   **打包是 async**：`InitSpriteAsset` 用 `s_IsRefreshingSpriteAsset` 旗標互斥，避免並行打包；refresh 中再呼叫會等到上次結束。
*   **`BearingY = 0.85` 預設**：與 TMP 字型基線對齊，調整可能會讓圖示飄上飄下。
*   **預設 ID `AbsorbShield`**（`RCG_IconSpriteGenData.DefaultID`）。
*   **`m_Disable` 已被註解**：曾規劃跳過 disabled 圖示；目前所有圖示都會被打包。
*   **`LocalizeName` 行為**：UI 模式回 TMPKey（圖示），非 UI 模式回 i18n 翻譯（純文字）；卡牌描述系統依 `RCG_BattleSetting.IsShowOnUI` 切換。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_IconSprite.cs`
*   **繼承自**：`RCG_Asset<RCG_IconSprite>`
*   **AssetGroup**：`EditTags`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_Icon` | Icon | `RCG_SpriteData` | 預設 `Status_BloodDrain` |
| `m_Scale` | Scale | `float` | 預設 1 |
| `m_BearingX` | BearingX | `float` | 預設 0 |
| `m_BearingY` | BearingY | `float` | 預設 0.85 |

### A.3 重要 Method（含 static）

*   **`InitSpriteAsset(token)` (static async)** — 啟動時打包；含 OnLoadedModule 訂閱。
*   **`GenerateSpriteAsset(token)` (static)** — 載入所有 icons 並建立 `TMP_SpriteAsset`。
*   **`RefreshSpriteAsset / RefreshSpriteAssetAsync`** — 重新打包（重複呼叫互斥）。
*   **`LoadIconSprites(token)` (private static)** — 讀所有 textures + names + icons 的 helper。
*   **`Save()` (override)** — 觸發 RefreshSpriteAsset。
*   **`TMPKey` (property)** — `<sprite name={ID}>`。
*   多個 static `EffectIcon_*` 快速引用。

### A.4 與其他系統的互動

*   **`TMP_SpriteAsset`** — TextMeshPro 圖示資源。
*   **`RCG_TMPTools`** — 打包與工具（`CreateSpriteAsset / RefreshSpriteAsset / CreateIconSpriteSheetEditor`）。
*   **`RCG_SpriteData`** — 來源 sprite。
*   **`UCL_ModuleService.OnLoadedModule`** — 模組重載時自動 refresh。
*   **`RCG_BattleSetting.IsShowOnUI`** — 文字描述模式切換。
*   **`RCG_IconSpriteGenData`** — Asset Entry；預設 `AbsorbShield`。

### A.5 已知議題

*   `m_Disable` / `CheckSpriteAsset / GenerateSpriteAsset (sync ver)` 等多處註解，標示舊版同步打包流程已改 async。
*   `s_IsRefreshingSpriteAsset` 是全局 static flag，極端情況下若 task 拋例外未復位可能卡住未來 refresh。