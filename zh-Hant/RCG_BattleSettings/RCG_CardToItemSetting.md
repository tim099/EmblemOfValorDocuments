---
title: 卡牌轉為物品 說明
description: 把選取的手牌道具化（產生「打出此卡」效果的物品），可選臨時或永久
last_updated: 2026-05-05
target_audience: [Designer, Modder, AI_Agent]
---

# 卡牌轉為物品

> 程式類別名稱：`RCG_CardToItemSetting`

## 用途
**把卡牌轉化為背包物品**。物品的使用效果就是「**生成這張卡到手牌並打出**」。常見用途：
*   「將攻擊卡儲存為藥水」
*   「卡片化道具」風格的特殊機制

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **SelectHandCardSetting** | 是 | 選取要道具化的手牌；展開設定條件、張數、過濾。 |
| **IsTmpItem** | — | 預設打勾。打勾 = 戰鬥結束自動移除；不打勾 = 永久保留。 |

## 行為說明
*   先讓玩家透過 `SelectHandCardSetting` 選牌。
*   對每張選中的卡：
    1. clone 出新 `RCG_ItemData`（名稱、圖示沿用卡片）。
    2. 物品的「使用效果」自動寫入 `RCG_CardCreateSetting`（生成此卡並加入手牌）。
    3. 詠唱卡會把 `IsChanted` 帶入避免重複詠唱。
    4. 註冊為 `BattleRuntime` 資料並加入背包。
*   原牌會被消滅。
*   描述格式：`IsTmpItem=true` → `CardToTmpItem_Des`；否則 → `CardToItem_Des`。

## 注意事項
*   **臨時物品 vs 永久物品的存檔**：臨時物品在戰鬥結束時清除；永久物品則保存到玩家背包，跨戰鬥保留 — 大量永久道具化可能讓背包爆炸，請設計上限。
*   **道具化後不能再融合**：原卡已被消滅；想做「卡牌融合 + 道具化」請先融合再道具化。
*   **與「卡牌融合」的差別**：融合是合多張變新卡；本設定是「**卡 → 物品**」單向轉換。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：[`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardToItemSetting.cs`](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardToItemSetting.cs)
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_CardToItemSetting` → 「卡牌轉為物品」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_SelectHandCardSetting` | `SelectHandCardSetting` | `RCG_SelectHandCardSetting` | — | |
| `m_IsTmpItem` | `IsTmpItem` | `bool` | `IsTmpItem` (=「是否為臨時物品?」) | 預設 `true` |

### A.3 重要 Method 摘要
*   **`AddAction`**：選牌後對每張卡：
    1. 建 `RCG_ItemData`（`Data.m_Name = m_LocalizeName`, `Data.m_Icon = m_CardIcon`, `ID = aCard.ID`）。
    2. 嵌套 `RCG_CommonEffect` (`OnPlay`) → `RCG_CardCreateSetting` (`AddToHandCard`, `m_RuntimeCardData = new RCG_CardBattleDataPointer(aCard)`)。
    3. 詠唱卡 → `aSetting.m_IsChanted = true`。
    4. `RCG_DataService.Ins.AddRuntimeData(aItemData, DataType.BattleRuntime)`。
    5. `new RCG_Item(aItemData, RCG_Item.ItemType.Runtime)` + `m_TmpItem = m_IsTmpItem` + `AddItem()`。
*   **`Info`**：`m_IsTmpItem ? CardInfoData.TmpItemInfo : null`。

### A.4 與其他系統的互動
*   **`RCG_CardCreateSetting` + `RCG_CardBattleDataPointer`**：物品的使用效果以這個組合表達「打出記下的卡」。
*   **`DataType.BattleRuntime`**：戰鬥結束自動清除的資料容器。
*   **`RCG_AquireItemPanel`**：UI 反饋（檔案 80 行外的後續邏輯）。
