---
title: 抽牌 說明
description: 從牌堆 / 棄牌堆抽取指定張數，可限定卡牌標籤
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 抽牌

> 程式類別名稱：`RCG_CardDrawSetting`

## 用途
**抽牌**到手牌。支援從**牌堆頂**（一般抽牌）、**牌堆內選取**（玩家挑）、**棄牌堆內選取**（撿回來）三種來源。

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **DrawCardNum** | 是 | 抽取張數（支援變數）。實際抽取會夾到 `[0, 玩家手牌空位]`。 |
| **DrawType** | 是 | 抽取方式：<br>• **FromDeckTop** — 從牌堆頂抽（一般抽牌，預設）<br>• **PickFromDeck** — 從牌堆中**選擇**抽<br>• **PickFromDiscardPile** — 從棄牌堆**選擇**抽（撿牌） |
| **CardTags** | 否 | 限定抽取有此標籤的卡（如「治療卡」）。 |
| **DrawCardNotIncludedTags** | 否 | 反向：抽**不含**這些標籤的卡。 |

## 行為說明
*   `FromDeckTop` 直接抽 N 張到手牌，**手牌已滿則不抽多餘的**。
*   `Pick*` 模式會跳出選牌 UI 讓玩家選擇 N 張。
*   描述格式為「**抽 {張數} 張 {標籤} 卡**」（i18n key `DrawCardIcon_{DrawType}`）。

### 卡片融合
兩張「抽牌」融合會把張數加總（**不檢查 DrawType / CardTags 是否相同**，可能踩坑）。

## 注意事項
*   **手牌空位限制**：`DrawCardNum` 會被夾到 `RCG_Player.Ins.CardSpace`，要抽的張數超過空位的部分會被吃掉而非觸發棄牌。
*   **標籤過濾與牌堆耗盡**：限定標籤抽牌時，若牌堆內無符合條件的卡，**不會自動洗棄牌堆**，可能抽到比 N 少。
*   **PickFrom* 模式對 AI 無效**：選牌 UI 是給玩家用的，敵人若觸發此設定會無動作（建議 AI 一律用 `FromDeckTop`）。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardDrawSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **`[System.Serializable]`** 標記
*   **i18n 類別名 key**：`RCG_CardDrawSetting` → 「抽牌」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_DrawCardNum` | `DrawCardNum` | `IntVariable` | — | `[UCL_FieldOnGUI]` |
| `m_DrawType` | `DrawType` | `DrawType` (檔內 enum) | — | 3 種來源 |
| `m_CardTags` | `CardTags` | `List<RCG_CardTagGenData>` | — | |
| `m_DrawCardNotIncludedTags` | `DrawCardNotIncludedTags` | `bool` | — | |

### A.3 重要 Method 摘要
*   **`AddAction`**：`Mathf.Clamp(m_DrawCardNum, 0, RCG_Player.Ins.CardSpace)` → 依 `m_DrawType` 分流：
    *   `FromDeckTop` → `CreateAction.AddDrawCardAction(iData, ...)`。
    *   `PickFromDeck` → `CreateAction.AddSelectCardAction(CardPos.Deck, ...)` + `AddPickFromDeckAction`。
    *   `PickFromDiscardPile` → 同上但 `CardPos.DiscardPile`。
*   **`Fusion(other)`**：clone + `IntVariable.FuseAdd(m_DrawCardNum)`，**不檢查其他欄位**。
*   **`LocalizeKey`** → `"DrawCardIcon_" + m_DrawType.ToString()`。

### A.4 與其他系統的互動
*   **`SelectCardSetting`**：`RCG_CardDiscardSetting.cs` 中定義的工具類，用於標籤過濾。
*   **`CreateAction.AddSelectCardAction / AddPickFromDeckAction / AddDrawCardAction`**：抽牌動作建構工具。
*   **`RCG_Player.Ins.CardSpace`**：手牌空位上限。
