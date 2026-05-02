---
title: 手牌費用變化 說明
description: 對指定範圍的手牌進行費用增加 / 降低 / 歸零
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 手牌費用變化

> 程式類別名稱：`RCG_CardCostAlterSetting`

## 用途
**改變手牌費用**。常見用途：
*   「全部手牌費用 -1」
*   「一張隨機手牌變 0 費」
*   「費用最高的手牌歸零」

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **CardAlterRange** | 是 | 影響範圍：<br>• **AllHandCards** — 全部手牌<br>• **SelectedCards** — 玩家事前選擇的手牌<br>• **OneRandomHandCard** — 隨機一張<br>• **ThisCard** — 這張卡本身（卡牌觸發才有效）<br>• **MostCostCard** — 費用最高的卡<br>• **NewSelectedCards** — 在本效果中選擇（會自動展開選取設定） |
| **CostAlterType** | 是 | 變化模式：<br>• **AddHandCardCost** — 增加費用<br>• **SubHandCardCost** — 降低費用<br>• **HandCardCostToZero** — 變為 0 費（不需填數值） |
| **SelectHandCardSetting** | NewSelectedCards 時必填 | 內嵌的選取設定；其他模式下會自動隱藏。 |
| **費用** (`Cost`) | AddHandCardCost / SubHandCardCost 時必填 | 變化的數值（變為 0 費時忽略）。 |

## 行為說明
*   **NewSelectedCards** 模式會先觸發內嵌的選取設定，玩家選完才執行。
*   依範圍篩出卡片後依 CostAlterType 套：
    *   增加 → `AlterCost(+Cost)`
    *   降低 → `AlterCost(-Cost)`
    *   歸零 → `AlterCost(-當前 Cost)`（直接抵消）

### 卡片融合
兩張同 `CostAlterType` 的設定融合會把 `Cost` 加總。**不同 CostAlterType 之間不能融合**（避免「+1 與歸零」融合產生語意衝突）。

## 注意事項
*   **OneRandomHandCard** 會即時隨機，而非事先抽好；多次觸發會抽到不同張。
*   **ThisCard 在非卡牌觸發場合無效**：例如被狀態效果觸發時 `iData.Card == null`。
*   **HandCardCostToZero 不是「費用 -∞」**：是把當前費用減去當前值，**疊加 +1 費後又歸零會變 0 費**，非反向加成。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardCostAlterSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_CardCostAlterSetting` → 「手牌費用變化」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_CardAlterRange` | `CardAlterRange` | `CardAlterRange` (enum) | — | 6 種來源 |
| `m_CostAlterType` | `CostAlterType` | `CostAlterType` (enum) | — | 3 種模式 |
| `m_SelectHandCardSetting` | `SelectHandCardSetting` | `RCG_SelectHandCardSetting` | — | `[Conditional(m_CardAlterRange, false, NewSelectedCards)]` 條件顯示 |
| `m_Cost` | 費用 | `IntVariable` | `Cost` | |

### A.3 重要 Method 摘要
*   **`AddAction`**：`NewSelectedCards` 模式先呼叫 `m_SelectHandCardSetting.AddAction`，後續 ActionTrigger 中依範圍取卡並 `AlterCost`。
*   **`Fusion(other)`**：要求兩者 `m_CostAlterType` 相同；clone 自身並 `IntVariable.FuseAdd` 合併 `m_Cost`。
*   **`GetDescriptionFormat`**：依 CostAlterType 套不同 i18n key（`AddHandCardCost_Des` / `SubHandCardCost_Des` / `HandCardCostToZero_Des`）。

### A.4 與其他系統的互動
*   **`RCG_Card.AlterCost(int)`**：實際修改費用的入口。
*   **`UCL_Random.Instance.Next`**：OneRandomHandCard 的隨機來源。
