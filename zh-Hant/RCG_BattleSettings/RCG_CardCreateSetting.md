---
title: 生成卡牌 說明
description: 生成指定卡牌並加入手牌、牌堆或棄牌堆；可指定生成張數
last_updated: 2026-05-05
target_audience: [Designer, Modder, AI_Agent]
---

# 生成卡牌

> 程式類別名稱：`RCG_CardCreateSetting`

## 用途
**生成新卡牌**並加入指定位置。常見用途：
*   「加入 2 張燒灼卡到棄牌堆」
*   「將強化版卡放入手牌頂端」
*   「生成 1 張嘲諷卡到牌堆」

需要讓玩家**從多選一**請改用「**生成卡牌(多選)**」(`RCG_CreateSelectedCardSetting`)。

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **CardData** | 是 | 要生成的卡牌模板（`RCG_CardGenData`）。 |
| **IsChanted** | 詠唱卡才顯示 | 是否為已詠唱狀態（限詠唱卡才有此欄位）。 |
| **CreateCardType** | 是 | 加入位置：<br>• **AddToHandCard** — 加入手牌<br>• **AddToDeck** — 加入牌堆隨機位置（預設）<br>• **AddToDiscardPile** — 加入棄牌堆<br>• **AddToDeckTop** — 加入牌堆頂端 |
| **CreateCount** | 是 | 生成張數；支援變數綁定。 |

## 行為說明
*   描述格式為「**生成 {張數} 張 {卡牌名}**」（i18n key `CardCreateEffectDes_{CreateCardType}`）。
*   執行時為每一張呼叫 `RCG_CardBattleData.CreateCard(模板, IsChanted)`，並插入指定位置。
*   生成的卡會記入 `iData.m_TriggerData.m_CreatedCards`，後續設定可引用。

### 卡片融合
兩張「生成卡牌」融合時，**生成張數會加總**（例如「生成 1 張 X」+「生成 2 張 X」→「生成 3 張 X」）。

## 注意事項
*   **CardData 指向不存在 ID**：執行時會在 `CardData.GetData()` 拿到 null，卡片名稱顯示空字串。請確認 ID 已註冊。
*   **詠唱卡（IsChant）的 IsChanted 欄位**：只有當 CardData 是詠唱卡時才會出現此欄位（以反射 `IsChantCard()` 判斷）。
*   **AddToDeckTop 的視覺反饋**：玩家不會看到動畫，僅下次抽牌時冒出來；若要強烈呈現請改 AddToHandCard。
*   **與「生成卡牌(多選)」的差別**：本設定**沒有讓玩家選**的環節，直接生成；多選版才會出選擇 UI。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：[`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardCreateSetting.cs`](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardCreateSetting.cs)
*   **繼承自**：`RCG_BattleSetting`，實作 `UCLI_FieldOnGUI`
*   **`[System.Serializable]`** 標記
*   **i18n 類別名 key**：`RCG_CardCreateSetting` → 「生成卡牌」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_CardData` | `CardData` | `RCG_CardGenData` | — | 模板 |
| `m_RuntimeCardData` | （隱藏） | `RCG_CardBattleDataPointer` | — | `[UCL_HideOnGUI]`；用於道具化引用 |
| `m_IsChanted` | `IsChanted` | `bool` | — | `[Conditional(nameof(IsChantCard))]` 詠唱卡才顯示 |
| `m_CreateCardType` | `CreateCardType` | `CreateCardType` (enum) | — | 4 種加入位置 |
| `m_CreateCount` | `CreateCount` | `IntVariable` | — | `[UCL_FieldOnGUI]` |

### A.3 重要 Method 摘要
*   **`CardData` (property)**：優先使用 `m_RuntimeCardData`，否則 fallback `m_CardData.GetData()`。
*   **`OnGUI`**：自繪欄位，呼叫 `UCL_GUILayout.DrawField(this, ..., RCG_StaticFunctions.LocalizeFieldName)`。
*   **`AddAction`**：依 `m_CreateCount` 迴圈呼叫 `RCG_CardBattleData.CreateCard(aCardData, m_IsChanted)`，記錄到 `iData.m_TriggerData.m_CreatedCards`，加入 `CreateAction.CreateCard(aCard, m_CreateCardType)`。
*   **`Fusion(other)`**：clone + `IntVariable.FuseAdd` 合併 `m_CreateCount`；不檢查 `m_CardData` 是否相同（**注意可能融合不同卡？**）。
*   **`IsChantCard()` (private)** → for reflection；用於 `Conditional` 判斷 `IsChanted` 欄位是否顯示。
*   **`SerializeToJson / DeserializeFromJson`**：保留 `m_RuntimeCardData` 序列化邏輯（已註解，目前走 base）。

### A.4 與其他系統的互動
*   **`RCG_CardBattleData.CreateCard(...)`**：產生卡片實例。
*   **`CreateAction.CreateCard`**：實際的 Action class。
*   **`RCG_CardBattleDataPointer`**：runtime 卡片參照器，用於物品化（道具中嵌套生成卡牌）。
*   **`RCG_StaticFunctions.LocalizeFieldName`**：欄位名 i18n callback。

### A.5 已知議題
*   `m_RuntimeCardData` 的序列化邏輯被註解掉（走 base 預設），確認此欄位是否仍需序列化保存。
*   `Fusion` 不檢查兩者 `m_CardData` 是否相同，理論上可能讓「生成 X」+「生成 Y」融合成「生成 X 兩次」（取前者的 CardData）— 須驗證設計意圖。
