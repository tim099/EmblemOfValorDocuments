---
title: 遊戲初始資料 (RCG_GameInitData) 說明
description: 遊戲全局單例設定：起始道具/裝備/技能、卡牌系統設定、Pooling、編輯器設定、暗霧
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-ja
---

> [!WARNING]
> 翻訳待機中 — このファイルは日本語翻訳が必要です。
参考用に zh-Hant 原文を以下に掲載しています。


# 遊戲初始資料

> 程式類別名稱：`RCG_GameInitData`

## 用途

**遊戲的全局唯一設定資料**（單例 Asset，ID 固定為 `Default`）。包含：
*   起始物品 / 裝備 / 主動能力 / 資源
*   卡牌系統設定（`RCG_CardGameSetting`）
*   Pooling 設定（物件池）
*   Prefab 資源設定
*   編輯器設定
*   暗霧設定清單
*   裝備類型對應的圖示

繼承自 `RCG_Asset<RCG_GameInitData>`。

## 編輯器中的樣貌

```
RCG_GameInitData: Default
    GameVersion / DefaultLanguage
    CardGameSetting       ← 卡牌系統的全局設定
    PrefabResSetting      ← 通用 prefab 路徑
    PoolingGenDataSetting ← 物件池設定
    DarkMistSettings      ← 暗霧設定清單
    EditorSetting
    InitItems / InitActivePowers / InitEquipments / InitResources
    EquipmentTypeIcon     ← 裝備類型 → 圖示對應
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **GameVersion** | 是 | 遊戲版本（`RCG_GameVersionGenData`） |
| **DefaultLanguage** | 是 | 預設語系 |
| **CardGameSetting** | 是 | 卡牌系統設定（手牌數、抽牌規則等） |
| **PrefabResSetting** | 是 | 通用 prefab 路徑設定 |
| **PoolingGenDataSetting** | 是 | 物件池設定（哪些 prefab 要 pool / 初始量） |
| **DarkMistSettings** | 否 | 暗霧設定清單（多個 `RCG_DarkMistSettingsData` 引用） |
| **EditorSetting** | 否 | 編輯器相關設定 |
| **InitItems / InitActivePowers / InitEquipments / InitResources** | 否 | 開新遊戲時的起始物品 / 主動能力 / 裝備 / 資源 |
| **EquipmentTypeIcon** | 否 | `EquipmentType → RCG_SpriteData` 對應，UI 顯示用 |

## 行為說明

### `GameInit()`
開新遊戲時呼叫，把 `m_InitItems` / `m_InitEquipments` / `m_InitActivePowers` 全部加到玩家身上（個別 Asset 自帶 `AddItem` 邏輯）。

### `GetEquipmentIcon(EquipmentType)`
查 `m_EquipmentTypeIcon` 字典；找不到對應 key 時 LogError 並 fallback 到第一個 entry。

### Static 入口
*   `RCG_GameInitData.Ins` — 取 ID `Default` 的 Asset。
*   `CreateInstance()` — 從 Asset 重新生成一份起始資料（`useDefaultIfMissing = false`）。

## 注意事項

*   **ID 固定為 `Default`**：本類別預期只有一個 Asset。新增其他 ID 不會被系統自動讀取（除非透過 `Util.GetData` 手動呼叫）。
*   **`m_GameSetting` 已被註解**：曾規劃讓 GameInitData 引用 GameSettingData，現在改由 `Application.version` 自動關聯。
*   **`m_UnlockSetting` 已被註解**：解鎖設定改由 `RCG_UnlockData` 處理。
*   **`m_InitEquipments` 標 TODO**：「準備移到角色設定資料中 不同角色自帶不同裝備」——未來可能搬到 `RCG_CharacterData`。
*   **裝備類型圖示找不到時的 fallback**：用第一個 entry，可能不是想要的；漏設會看到錯誤的圖示。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_GameInitDatas/RCG_GameInitData.cs`
*   **繼承自**：`RCG_Asset<RCG_GameInitData>`
*   **AssetGroup**：`EditGameSetting`
*   **常數**：`DefaultID = "Default"`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_GameVersion` | GameVersion | `RCG_GameVersionGenData` | |
| `m_DefaultLanguage` | DefaultLanguage | `UCL_LanguageCodeEntry` | |
| `m_CardGameSetting` | CardGameSetting | `RCG_CardGameSetting` | |
| `m_PrefabResSetting` | PrefabResSetting | `RCG_PrefabResSetting` | |
| `m_PoolingGenDataSetting` | PoolingGenDataSetting | `RCG_PoolingGenDataSetting` | |
| `m_DarkMistSettings` | DarkMistSettings | `List<RCG_DarkMistSettingsGenData>` | |
| `m_EditorSetting` | EditorSetting | `RCG_EditorSetting` | |
| `m_InitItems` | InitItems | `List<RCG_ItemGenData>` | |
| `m_InitActivePowers` | InitActivePowers | `List<RCG_ActivePowerGenData>` | |
| `m_InitEquipments` | InitEquipments | `List<RCG_EquipmentGenData>` | |
| `m_InitResources` | InitResources | `List<RCG_ResourceGenData>` | |
| `m_EquipmentTypeIcon` | EquipmentTypeIcon | `Dictionary<EquipmentType, RCG_SpriteData>` | |

### A.3 重要 Method 摘要

*   **`Ins` / `CreateInstance` (static)** — 兩種取得入口（前者用 cache，後者強制重讀）。
*   **`GameInit()`** — 開新遊戲時加入起始物品 / 裝備 / 能力。
*   **`GetEquipmentIcon(type)`** — 圖示查詢；找不到 fallback 第一個。
*   **`CardGameSetting / PoolingGenDataSetting / PrefabResSetting` (static properties)** — 快捷存取。

### A.4 與其他系統的互動

*   **`RCG_DataService`** — runtime 玩家資料；`InitItems` 等加入此處。
*   **`RCG_DarkMistSettingsData`** — 暗霧設定清單元素。
*   **`RCG_CardGameSetting`** — 卡牌系統設定。
*   **`UCL_LanguageCodeEntry`** — 語系設定。

### A.5 已知議題

*   `m_GameSetting` / `m_UnlockSetting` / `m_CardClassBackgroundImages` 等多處已註解，標示舊版設計改動。
*   `m_InitEquipments` 待搬移到角色資料的 TODO。