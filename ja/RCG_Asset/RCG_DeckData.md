---
title: 牌組資料 (RCG_DeckData) 說明
description: 一份完整牌組的定義（卡牌清單與配置）；角色起始牌組、加入牌組、玩家牌組底層
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-ja
---

> [!WARNING]
> 翻訳待機中 — このファイルは日本語翻訳が必要です。
参考用に zh-Hant 原文を以下に掲載しています。


# 牌組資料

> 程式類別名稱：`RCG_DeckData`

## 用途

**一份完整牌組的定義**。每張卡牌 + 數量 = 牌組。應用情境：
*   角色 `RCG_CharacterData.m_Deck`（起始牌組）
*   角色 `m_JoinDeck`（中途加入時帶來的牌組）
*   `RCG_BattlePresetData.m_Deck`（測試戰鬥用牌組）
*   特殊事件給予的整套牌

繼承自 `RCG_Asset<RCG_DeckData>`，實作介面：`RCGI_Unloackable`（可解鎖）。

## 編輯器中的樣貌

```
RCG_DeckData: <ID>
    Name           ← 牌組顯示名（多語系）
    Deck           ← 卡牌清單（SpawnDeckData，含每張卡 + 數量）
    Unlock         ← 解鎖條件
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Name** | 否 | 顯示名（多語系）；空白時 fallback 到 ID |
| **Deck** | 是 | 卡牌清單與數量（`SpawnDeckData`） |
| **Unlock** | 否 | 解鎖條件 |

## 行為說明

### `SelectCard(setting)`
按 `SelectCardSetting` 從牌組裡**順序**篩出第一張符合的卡（**非隨機**）；命中則 clone 成 `RCG_CardBattleData` 回傳。

### `SelectCards(count, setting)`
TODO：尚未實作（程式內 `// ToDo` 直接 return null）。

### `GetAllCards()`
回傳整副牌組的 `RCG_CardGenData` 清單。

### Tooltip Infos
`Infos = m_Deck.Infos`：聚合所有卡上的狀態效果說明（例：「出血」、「燃燒」狀態解說）。

## 注意事項

*   **`SelectCards` 還沒實作**：要批量隨機抽卡，目前不能用此入口；需自己實作或走別的 utility。
*   **預設 ID `Default` / `BackUp`**：`RCG_DeckGenData.DefaultID = "Default"`、`BackUpID = "BackUp"`——常作為「初始牌組」與「備用牌組」的命名約定。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_DeckData.cs`
*   **繼承自**：`RCG_Asset<RCG_DeckData>`
*   **實作介面**：`RCGI_Unloackable`
*   **AssetGroup**：`EditItems`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Deck` | Deck | `SpawnDeckData` | 卡牌清單 + 配置 |
| `m_Unlock` | Unlock | `RCG_UnlockEntry` | |

### A.3 重要 Method

*   **`SelectCard(setting)`** — 按條件順序選一張，clone 成 `RCG_CardBattleData`。
*   **`SelectCards(count, setting)`** — **未實作**（return null + TODO）。
*   **`GetAllCards()`** — 全副牌列表。
*   **`AllCardsName / Infos / LocalizedName`** — 顯示用屬性。

### A.4 與其他系統的互動

*   **`SpawnDeckData`** — 實際牌組容器。
*   **`RCG_CardBattleData`** — 戰鬥用卡片實例。
*   **`SelectCardSetting`** — 卡牌選擇規則。
*   **`RCG_DeckGenData`** — Asset Entry；`Default` / `BackUp` 兩個常數 ID。

### A.5 已知議題

*   `SelectCards` 尚未實作（`// ToDo`）。