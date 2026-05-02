---
title: 選取手牌 說明
description: 開啟選牌 UI（或自動選），把選中的卡寫入後續設定可引用的 SelectedHandCards
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 選取手牌

> 程式類別名稱：`RCG_SelectHandCardSetting`

## 用途
**讓玩家（或系統）選取手牌** — 將結果存入 `SelectedHandCards` 給後續設定使用（如「棄牌」「強化」「複製」「融合」「卡牌轉物品」等）。是「**前置設定**」性質。

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **SelectType** | 是 | 選取方式（**13 種**，最常用見下方）。 |
| **SelectRange** | SelectFromRange* 必填 | 選取範圍（手牌 / 牌堆 / 棄牌堆，可多選）。 |
| **SelectNum** | SelectByCount/RandomSelect/RandomOneCardWithCost/LeftSide(Of)/RightSide 必填 | 選取張數（或費用值）。 |
| **CardTags** | 否 | 限定有此標籤的卡。 |
| **SelectCardNotIncludedTags** | — | 反向選取（不含這些標籤）。 |
| **SelectCardWithoutEnhancement** | — | 限定未強化的卡。 |
| **SelectCountVariable** | 否 | 把選取張數寫入變數（後續設定可引用）。 |
| **DetailSetting** | 否 | 描述格式微調（`Default` / `Then` / `SelectPrefix`）。 |

### 主要選取方式（SelectType）
| 值 | 行為 |
|---|---|
| **SelectByCount** | 玩家選 N 張（最常用，預設） |
| **SelectAnyCount** | 玩家任意數量 |
| **SelectAll** | 自動全選（無 UI） |
| **SkipSelect** | 跳過選取（前面已選好） |
| **CreatedCards** | 自動選取本次效果生成的卡 |
| **RandomSelect** | 隨機選 N 張 |
| **SelectFromRange** | 從手牌 / 牌堆 / 棄牌堆混合選 |
| **SelectFromRangeAll** | 從多個範圍**全選** |
| **ThisCard** / **TriggeredCard** | 這張觸發中的卡 |
| **LeftSide** / **RightSide** | 該卡左 / 右側 N 張 |
| **LeftSideOfThis** | 此卡左側的卡 |
| **RandomOneCardWithCost** | 隨機選一張費用為 X 的卡（X 由 `SelectNum` 指定） |

## 行為說明
*   **執行流程**：開啟選擇 UI（或依模式自動選），將結果填入 `iData.SelectedHandCards`。
*   **後續引用**：下游設定（如棄牌、強化）讀取 `iData.SelectedHandCards`。
*   **變數寫入**：`SelectCountVariable` 非空時，把實際選取張數寫入該變數。
*   **描述**：根據 `DetailSetting.DescriptionType` 決定句型。

## 注意事項
*   **本設定通常不單獨使用**：它只負責「選取」並填值；要對選中的卡做事，**後面要接** 棄牌 / 強化 / 複製 / 融合 / ... 等設定。
*   **AI 觸發**：玩家選擇 UI 對 AI 無效；AI 用此設定請選 `RandomSelect` / `SelectAll` / `SkipSelect` 等不需玩家輸入的模式。
*   **SelectFromRange 範圍空清單**：等於「不進行選取」。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_SelectHandCardSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_SelectHandCardSetting` → 「選取手牌」
*   **同檔內巢狀類**：`SelectType` (13 個值) / `DescriptionType` (3 個值) / `DetailSetting` (含 `m_DescriptionType`)

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_SelectType` | `SelectType` | enum | — | 13 種選取方式 |
| `m_SelectRange` | `SelectRange` | `List<CardPos>` | — | `[Conditional(nameof(m_SelectType), false, SelectFromRange, SelectFromRangeAll)]` |
| `m_SelectNum` | `SelectNum` | `IntVariable` | — | `[Conditional(... 6 種需要數量的 SelectType)]` |
| `m_CardTags` | `CardTags` | `List<RCG_CardTagGenData>` | — | |
| `m_SelectCardNotIncludedTags` | `SelectCardNotIncludedTags` | `bool` | — | |
| `m_SelectCardWithoutEnhancement` | `SelectCardWithoutEnhancement` | `bool` | — | |
| `m_SelectCountVariable` | `SelectCountVariable` | `string` | — | |
| `m_DetailSetting` | `DetailSetting` | `DetailSetting` (檔內) | `DetailSetting` | |

### A.3 重要 Method 摘要
*   **`AddAction`**（檔案 100+ 行外）：依 `m_SelectType` 14 種分支 — 開 UI / 自動選 / 範圍選 / 隨機選等，最終填入 `iData.SelectedHandCards`。
*   **`TagDes` (private)**：標籤描述生成；含 `NotIncludedDes` 與 `SelectCardWithoutEnhancementDes` 兩種額外修飾。
*   **`GetDescriptionFormat`** / **`GetDescriptionParams`**：依 `m_DetailSetting.m_DescriptionType` 變化句型。

### A.4 與其他系統的互動
*   **`iData.SelectedHandCards`**：填值目標；下游設定讀取此清單。
*   **`CardPos` enum**：選取範圍類型（Hand / Deck / DiscardPile）。
*   **`CreateAction.AddSelectCardAction`**：開 UI 選牌的入口。
