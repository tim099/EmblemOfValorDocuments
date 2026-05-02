---
title: 消耗道具 說明
description: 從玩家背包消耗指定物品；可作為「消耗道具才能打出」的卡片條件
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 消耗道具

> 程式類別名稱：`RCG_ConsumeItemSetting`

## 用途
**消耗玩家背包中的物品**。可作為「**打出條件**」— 沒有對應物品時這張卡無法打出。常見用途：
*   「消耗 1 個火藥包，造成 AOE 火焰傷害」
*   「消耗 2 個藥草，全隊回血」

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **道具** (`Items`) | 是 | 要消耗的物品清單；每項是 `RCG_ItemGenData`（物品模板）。 |

## 行為說明
*   **可玩判定**：背包必須含有清單上的全部物品才能打出（按 ID 配對，**多個同 ID 也要對應消耗**）。
*   **觸發**：依清單逐一消耗對應背包物品。
*   描述：列出所有要消耗的物品名稱。

## 注意事項
*   **TODO 提醒**：目前**資源變化時不會即時刷新卡牌可玩判定** — 玩家可能看到「不可打出」狀態延遲更新。詳情查 source 中 `Todo:資源變化時 要刷新卡牌才能正確判斷目前是否能打出`。
*   **空清單**：合法，但等於無消耗 — 這張卡會永遠可打出，且不會消耗任何東西。
*   **重複消耗同 ID 物品**：清單中同 ID 出現兩次代表要消耗 2 個；背包不足 2 個則不可打出。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ConsumeItemSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_ConsumeItemSetting` → 「消耗道具」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_Items` | 道具 | `List<RCG_ItemGenData>` | `Items` (=「道具」) | 註解寫成「獲得的資源(金幣等..)」是舊註解，現為消耗 |

### A.3 重要 Method 摘要
*   **`CheckPlayable`**：clone `RCG_DataService.Ins.m_ItemsData.GetItems()`，逐一以 `aItem.ID` 配對找到並 RemoveAt；找不到立即 false。
*   **`Infos`**：base.Infos + 每個 `m_Items[i].GetData()` 的資訊。
*   **`AddAction`**（檔案 80 行外）：實際從背包扣除物品。

### A.4 與其他系統的互動
*   **`RCG_DataService.Ins.m_ItemsData`**：玩家背包資料來源。
*   **`RCG_ItemGenData / RCG_Item`**：物品模板與實例。

### A.5 已知議題
*   程式中明文 `Todo:資源變化時 要刷新卡牌才能正確判斷目前是否能打出` — 卡片可玩狀態未即時刷新。
*   欄位註解寫「獲得的資源」可能是 copy-paste 殘留，實際為「消耗的物品」。
