---
title: 卡牌融合 說明
description: 讓玩家選取多張手牌進行融合，產生新卡牌；原牌會被消滅
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 卡牌融合

> 程式類別名稱：`RCG_CardFusionSetting`

## 用途
**讓玩家選取手牌進行融合**：選中的卡會被消滅，並產生一張新的融合卡加入手牌。常見用途：
*   「鍊金術士」風格的合成卡
*   「將兩張攻擊卡合成為強化攻擊」

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **SelectHandCardSetting** | 是 | 內嵌的選取設定，決定選擇 UI 的條件、張數、限制等。 |

## 行為說明
*   先觸發內嵌的 `SelectHandCardSetting` 讓玩家選牌。
*   選定後：
    1. 對選中卡呼叫 `CardFusion()` 計算融合後的 `RCG_CardData`。
    2. 用融合結果建立新 `RCG_CardBattleData`。
    3. 把選中的卡全部 **`Banish` 消滅**（不是棄牌！）。
    4. 把新卡加入手牌。
*   描述格式為「**將選取的卡融合**」（i18n key `CardFusion_Des`）。

## 注意事項
*   **融合邏輯由 `CardFusion()` 決定**：實際融合規則是**卡牌系統層級**的（而非本設定）— 想知道兩張卡融合會產生什麼，請查看 `RCG_CardData` / `IList<RCG_CardData>.CardFusion()`。
*   **原牌一律消滅**：是 `RemoveType.Banish`，不會進棄牌堆，**不會觸發棄牌相關效果**。
*   **沒有融合候選時**：選擇 UI 仍會跳出但無可選項；玩家只能取消。請在 `SelectHandCardSetting` 中設定合理的最低張數。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardFusionSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_CardFusionSetting` → 「卡牌融合」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_SelectHandCardSetting` | `SelectHandCardSetting` | `RCG_SelectHandCardSetting` | — | |

### A.3 重要 Method 摘要
*   **`AddAction`**：先呼叫 `m_SelectHandCardSetting.AddAction(InsertInOrder)`，再 ActionTrigger 內：
    1. `iData.SelectedHandCards` → 收集 `RCG_CardData` 並逐張加入 `aDiscardCardList`。
    2. `aSelectedCards.CardFusion()` → `RCG_CardData` 融合結果。
    3. `RCG_CardBattleData.CreateCard(aFusionCardData, false)` → 新 battle data。
    4. `CreateAction.AddDiscardCardActions(..., RemoveType.Banish)` 消滅原牌。
    5. `CreateAction.CreateCard(aNewCard, CreateCardType.AddToHandCard)` 加入手牌。
*   **`GetDescriptionFormat`**：i18n key `CardFusion_Des`，內含 `m_SelectHandCardSetting.GetDescriptionFormat`。

### A.4 與其他系統的互動
*   **`IList<RCG_CardData>.CardFusion()`**：實際融合邏輯（擴充方法）。
*   **`RCG_SelectHandCardSetting`**：選牌 UI 觸發。
*   **`CreateAction.AddDiscardCardActions / CreateCard`**：消滅 + 生成的 Action 建構工具。
