---
title: 複製 說明
description: 複製選取的手牌，可指定加入手牌、牌堆或棄牌堆
last_updated: 2026-05-05
target_audience: [Designer, Modder, AI_Agent]
---

# 複製

> 程式類別名稱：`RCG_CopySetting`

## 用途
**複製選取的手牌**，產生一張**新的副本**並依設定加入手牌、牌堆或棄牌堆。常見用途：
*   「複製這張卡到手牌」
*   「複製選中的卡牌進牌堆頂」

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **SelectHandCardSetting** | 是 | 選取要複製的手牌（含 `SkipSelect` / `ThisCard` / 一般選取等模式）。 |
| **CreateCardType** | 是 | 副本加入位置：手牌 / 牌堆 / 棄牌堆 / 牌堆頂。 |

## 行為說明
*   先觸發 `SelectHandCardSetting`，然後對每張選中的卡呼叫 `CopyCard()` 產生副本，並透過 `CreateAction.CreateCard(副本, CreateCardType)` 插入指定位置。
*   描述根據 `SelectType` 不同有三種變化：
    *   `SkipSelect` → 直接顯示「複製」
    *   `ThisCard` → 顯示「複製這張卡」
    *   其他 → 「{選取描述} 並複製」
*   若 `CreateCardType ≠ AddToHandCard`，會額外附加位置說明。

## 注意事項
*   **副本不繼承詠唱狀態**：複製後是新卡，原卡的詠唱進度**不會帶過去**（除非設計上特別處理）。
*   **複製強化過的卡**：`CopyCard()` 會包含強化效果；副本仍是強化版（**不會降級為基礎版**）。
*   **CreateCardType = AddToDeckTop**：玩家不會看到動畫；可能誤以為沒效果。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：[`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CopySetting.cs`](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CopySetting.cs)
*   **繼承自**：`RCG_BattleSetting`
*   **無 i18n 類別名 key**：i18n key `RCG_CopySetting` 用於描述（「複製」），但 `AllTypes` 顯示為 stripped name 「Copy」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_SelectHandCardSetting` | `SelectHandCardSetting` | `RCG_SelectHandCardSetting` | — | `[SerializeField] protected` |
| `m_CreateCardType` | `CreateCardType` | `CreateCardType` (enum) | — | 共用 `RCG_CardCreateSetting.CreateCardType` |

### A.3 重要 Method 摘要
*   **`AddAction`**：選牌 → 對每張 `aCard.CopyCard()` → `CreateAction.CreateCard(副本, m_CreateCardType, InsertInOrder)`。
*   **`Infos`**：`RCG_CopySetting` + `CopySettingInfo` i18n。
*   **`GetDescriptionFormat`**：依 `m_SelectHandCardSetting.m_SelectType` 三分支處理。

### A.4 與其他系統的互動
*   **`RCG_CardBattleData.CopyCard()`**：實際的副本產生方法。
*   **`CreateAction.CreateCard`**：副本插入位置的 Action。
*   **`RCG_SelectHandCardSetting`**：選牌觸發。
