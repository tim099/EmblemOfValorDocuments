---
title: 宣告變數 說明
description: 從多種戰鬥條件取出數值並存入命名變數，後續設定可引用
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 宣告變數

> 程式類別名稱：`RCG_VariableSetting`

## 用途
**從各種戰鬥條件取出一個數值，存入指定變數名稱**，讓後續設定可引用此變數。是「**動態效果**」的核心 — 實現如「**對每張使用過的攻擊卡造成 1 點傷害**」這類依戰鬥狀態縮放的卡牌。

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **VariableName** | 是 | 變數名稱（預設 `X`），後續設定以此名稱引用。 |
| **VariableCondition** | 是 | 取值條件（**18 種**，見下表）。 |
| **Range** | TargetStatusCount/TargetAttackPower/TargetArmorCount/TargetDebuffLayersSum 模式 | 目標範圍（attack range 風格）。 |
| **UsedCardType** | CardUsedCount/TurnCardUsedCount 模式 | 計算的卡牌類型（空清單 = 不限類型）。 |
| **Status** | TargetStatusCount 模式 | 要查詢層數的狀態類型。 |
| **MonsterTags** | MonsterCount 模式 | 限制怪物類型；空清單 = 全部。 |
| **Value** | IntVariable 模式 | 直接給定的固定數值（變數可變）。 |

### VariableCondition 18 種模式
| 值 | 取出什麼 |
|---|---|
| **RemainCost** | 玩家當前能量（預設） |
| **HandCardCount** | 手牌張數 |
| **DiscardedCardCount** | 棄牌堆張數 |
| **DeckCardCount** | 抽牌堆張數 |
| **BanishedCardCount** | 消滅牌堆張數 |
| **CardUsedCount** | 本場戰鬥使用過的卡（可限類型） |
| **TurnCardUsedCount** | 本回合使用過的卡（可限類型） |
| **ThisCardUseCount** | 這張卡的累計使用次數 |
| **ThisCardCost** | 這張卡的當前費用 |
| **CostOfSelectedCards** | 選取卡的費用總和 |
| **TargetStatusCount** | 目標的指定狀態層數 |
| **TargetDebuffLayersSum** | 目標所有 Debuff 層數總和 |
| **TargetArmorCount** | 目標的護甲層數 |
| **TargetAttackPower** | 目標當前攻擊力（怪物 AI 行動中最大值） |
| **MonsterCount** | 場上活著的敵人數量（可限類型） |
| **PlayerUnitCount** | 場上活著的我方單位數量 |
| **DarkMistLevel** | 黯霧等級（資源 `Supply`） |
| **IntVariable** | 直接給定的固定數值或變數計算結果 |

## 行為說明
*   執行時：依 VariableCondition 計算數值 → 存入 `iData.VariableDic[VariableName]`。
*   描述格式：「**{變數條件描述} {變數名稱}({實際值}) = {取出的值}**」 — 在戰鬥中會即時顯示當前值。
*   Editor 模式（非戰鬥）下，變數值顯示為占位（不真的執行）。

## 注意事項
*   **變數名稱衝突**：多個 `VariableSetting` 用同名變數，後者會覆蓋前者。請語意化命名（如 `HandCount` / `EnemyAtk`）。
*   **變數作用域**：存於 `iData.VariableDic` — 同一個 `TriggerEffectData` 樹內共享。**子 ChildData 也能看到**（繼承）。
*   **VariableName 為空**：`AddAction` 直接 return — 等於這個設定**完全沒效果**。
*   **計算時機**：在 `AddAction` 觸發點計算 — 後續設定取值時拿的是該瞬間的快照。
*   **預覽傷害**：`GetPreviewDamage` 會把變數值預先寫入 `VariableDic` 讓下游預覽計算使用。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_VariableSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_VariableSetting` → 「宣告變數」
*   **同檔頂層 enum**：`VariableCondition` (18 個值)

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_VariableName` | `VariableName` | `string` | `VariableName` (=「變數名稱」) | 預設 `"X"` |
| `m_VariableCondition` | `VariableCondition` | `VariableCondition` (檔內 enum) | — | 預設 `RemainCost` |
| `m_Range` | `Range` | `AttackRange` | — | `[Conditional(... 4 種需要 Range 的條件)]` |
| `m_UsedCardType` | `UsedCardType` | `List<RCG_CardTagGenData>` | — | `[Conditional(... CardUsedCount, TurnCardUsedCount)]` |
| `m_Status` | `Status` | `RCG_CustomStatusGenData` | `Status` | `[Conditional(... TargetStatusCount)]` |
| `m_MonsterTags` | `MonsterTags` | `List<RCG_MonsterTagGenData>` | — | `[Conditional(... MonsterCount)]` |
| `m_Value` | `Value` | `IntVariable` | — | `[Conditional(... IntVariable)]` |

### A.3 重要 Method 摘要
*   **`GetVariableValue(iData)` (protected)**：18 種 case 分流計算實際數值；非戰鬥模式或 `RCG_Player.Ins == null` 直接回 0。
*   **`AddAction`**：`m_VariableName` 為空直接 return；否則 `iData.VariableDic.Set(m_VariableName, GetVariableValue(iData))`。
*   **`GetPreviewDamage`** → 把當前值寫入 VariableDic 並回傳 `-1`（**非攻擊牌但仍要寫入變數讓下游預覽**）。
*   **`VarName(iData)` (public)** → 戰鬥中顯示 `{name}({value})`，否則只顯示 `{name}`。
*   **`GetDescriptionParams / GetDescriptionFormat`**：依 18 種條件生成不同句型，套對應 i18n key（`VariableCondition_*Des` / `VariableEffectDes` / `VariableEffectDes_IntVariable`）。
*   **`Info`**：`TargetStatusCount` 模式回傳 `m_Status` 資訊；其他回 null。

### A.4 與其他系統的互動
*   **`iData.VariableDic`**：變數寫入目標；後續 `IntVariable` 解析時引用。
*   **`RCG_Player.Ins.Cost / GetHandCardCount / DiscardedCardCount / BanishedCardCount / Deck.Deck.Count`**：抽出資料來源。
*   **`RCG_BattleAnalytics.Ins.GetCardUsedCount / GetTurnCardUsedCount`**：卡片使用統計。
*   **`RCG_BattleScene.Ins.GetUnits(UnitPos.All, faction)`**：場上單位查詢。
*   **`RCG_DataService.Ins.GetResource(Supply)`**：黯霧等級資料來源。

### A.5 已知議題
*   舊版 `DeserializeFromJson` 處理（如 `IntVariable → RemainCost` 替換）已註解。
*   `// QWQ2` / `// TODO` 註解標記了部分待修區塊（`HandCardCount` 描述參數）。
