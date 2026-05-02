---
title: 資源類型 (RCG_ResourceTypeData) 說明
description: 遊戲內各種資源（金幣 / 補給 / 靈魂 / 祝福）的定義：名稱、描述、圖示
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-en
---

> [!WARNING]
> Translation pending — this file needs an English translation.
The original zh-Hant content is included below for reference.


# 資源類型

> 程式類別名稱：`RCG_ResourceTypeData`

## 用途

**遊戲內可累積資源類型的定義**。Gold（金幣）、Supply（補給 / 暗霧減少道具）、Spirit（靈魂）、Blessing（祝福）等都是不同 ID 的 `RCG_ResourceTypeData`。本資料定義它們的顯示名、描述、圖示。

繼承自 `RCG_Asset<RCG_ResourceTypeData>`。

## 編輯器中的樣貌

```
RCG_ResourceTypeData: <ID>
    Name(多國語言)
    Description(多國語言)
    IconSprite      ← 對應的 RCG_IconSprite 引用
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Name** | 是 | 資源名（多語系） |
| **Description** | 否 | 資源描述（用於 tooltip） |
| **IconSprite** | 是 | 圖示引用（`RCG_IconSpriteGenData`） |

## 行為說明

### LocalizeName 切換
*   `RCG_BattleSetting.IsShowOnUI = false` → 回 `m_Name.Name`（純文字名）。
*   `IsShowOnUI = true` → 回 `m_IconSprite.TMPKey`（給 TextMeshPro 用的 sprite tag）。

### 預設四種資源（程式內常數）
*   `Gold` (`ResourceType_Gold`)
*   `Supply` (`ResourceType_Supply`) — 也對應到「闇霧」相關用途
*   `Spirit` (`ResourceType_Spirit`)
*   `Blessing` (`ResourceType_Blessing`)

## 注意事項

*   **ID 命名規則**：`ResourceType_<EnumName>`（例如 `ResourceType_Gold`）。
*   **`enum ResourceType` (檔內)** 只列出三種（Gold / Supply / Spirit）；Blessing 直接用字串而沒進 enum。
*   **與 `RCG_DifficultyData.m_PriceMult / m_SoulPriceMult`** 等難度倍率的對應：價格倍率影響 Gold / Spirit 兩種資源的商店價。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_ResourceTypeData.cs`
*   **繼承自**：`RCG_Asset<RCG_ResourceTypeData>`
*   **AssetGroup**：`EditGameSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Description` | Description | `RCG_LocalizeData` | |
| `m_IconSprite` | IconSprite | `RCG_IconSpriteGenData` | |

### A.3 重要 Property

*   **`LocalizeName`** — UI 模式回 TMPKey；其他回 Name。
*   **`Description`** — `m_Description.Name`。
*   **`Icon`** — `m_IconSprite.GetData().IconSprite`。

### A.4 與其他系統的互動

*   **`RCG_ResourceTypeGenData`** — Asset Entry；含 4 個 static 預設（Gold / Supply / Spirit / Blessing）。
*   **`RCG_IconSpriteGenData`** — 圖示資源引用。
*   **`RCG_DataService.Ins.GetResource(...)`** — runtime 取資源量的入口。
*   **`RCG_BattleSetting.IsShowOnUI`** — 控制 LocalizeName 顯示模式。