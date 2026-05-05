---
title: 生成卡牌(多選) 說明
description: 從候選清單中讓玩家選擇卡牌生成（或從牌庫亂數選），可指定加入位置與張數
last_updated: 2026-05-05
target_audience: [Designer, Modder, AI_Agent]
---

# 生成卡牌(多選)

> 程式類別名稱：`RCG_CreateSelectedCardSetting`

## 用途
從**候選清單**或**牌庫**讓玩家挑選卡牌生成 — 比「**生成卡牌**」多了「**從多選一**」的環節。常見用途：
*   「從 3 張隨機獎勵中選一張加入手牌」
*   「從整個牌庫挑一張加入手牌」

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **CreateType** | 是 | 候選來源：<br>• **Default** — 從 `CardGenDatas` 設定的清單選<br>• **FromDeck** — 從指定牌庫選 |
| **CreateCardType** | 是 | 加入位置（同「生成卡牌」）：手牌 / 牌堆 / 棄牌堆 / 牌堆頂。 |
| **CreateCount** | 是 | 玩家最多可選張數。 |
| **IsRandomSelect** | — | 打勾 = **隨機**選擇（不出 UI 直接抽）；不打勾 = 玩家手動選。 |
| **CardGenDatas** | CreateType=Default | 候選卡片清單（多張卡片模板）。 |
| **Deck** | CreateType=FromDeck | 候選牌庫資料（`RCG_DeckGenData`）。 |

## 行為說明
*   `Default` + 玩家選擇 → 從 `CardGenDatas` 開選擇 UI 讓玩家選 N 張。
*   `FromDeck` + 玩家選擇 → 從指定牌庫的所有卡開選擇 UI。
*   `IsRandomSelect = true` → 不開 UI，直接隨機抽 N 張。
*   選定後依 `CreateCardType` 加入指定位置。

## 注意事項
*   **與「生成卡牌」的差別**：「生成卡牌」直接生成寫死的卡；本設定**有讓玩家選**的環節（除非 `IsRandomSelect`）。
*   **CardGenDatas 重複的處理**：候選清單中重複出現的卡會在 `Infos` 自動去重；但選擇 UI 仍可能展示多次。
*   **AI 觸發此設定**：玩家選擇 UI 對 AI 無效；建議 AI 用 `IsRandomSelect = true` 避免卡住。
*   **FromDeck 拉取整個牌庫**：可能造成大量候選，UI 可能擁擠 — 請考慮是否要先過濾。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：[`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CreateSelectedCardSetting.cs`](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CreateSelectedCardSetting.cs)
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_CreateSelectedCardSetting` → 「生成卡牌(多選)」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_CreateType` | `CreateType` | enum (檔內) | — | `Default` / `FromDeck` |
| `m_CreateCardType` | `CreateCardType` | `CreateCardType` (共用) | — | 4 種位置 |
| `m_CreateCount` | `CreateCount` | `IntVariable` | — | 預設 1 |
| `m_IsRandomSelect` | `IsRandomSelect` | `bool` | — | |
| `m_CardGenDatas` | `CardGenDatas` | `List<RCG_CardGenData>` | — | `[Conditional("m_CreateType", false, Default)]` |
| `m_Deck` | `Deck` | `RCG_DeckGenData` | — | `[Conditional("m_CreateType", false, FromDeck)]` |

### A.3 重要 Method 摘要
*   **`Infos`**：`Default` 模式 hash 去重後加每張卡 info；`FromDeck` 模式加 deck info。
*   **`CardDatas` (private)**：依 CreateType 取候選 `RCG_CardData` 清單。
*   **`AddAction`**（檔案 90 行外）：選牌 + 加入位置（依 IsRandomSelect 分流）。

### A.4 與其他系統的互動
*   **`RCG_DeckGenData`**：牌庫資料來源。
*   **`CreateAction.AddSelectCardAction / CreateCard`**：UI 選牌與卡片插入入口。
