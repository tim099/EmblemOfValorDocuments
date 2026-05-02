---
title: 條件判斷說明
description: 依條件分流的戰鬥設定；條件成立執行一條路徑，否則執行另一條
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 條件判斷

> 程式類別名稱：`RCG_ConditionalSetting`

## 用途
依「**判定狀態**」結果**分流**：條件成立 → 跑「符合條件則執行」；不成立 → 跑「不符合條件則執行」。常見用途：
*   「若擊殺則加 1 層敏捷，否則造成額外 3 點傷害」
*   「若 HP < 50% 則自我治療，否則加護甲」
*   「若手牌 ≥ 5 則抽 1 張，否則棄 1 張」

## 編輯器中的樣貌
```
▼ ✓ [條件判斷(Conditional)] 若 (條件描述) 則 ⋯ 否則 ⋯
    判定狀態           ▶ (條件清單)
    符合條件則執行     ▶ (子設定清單)
    不符合條件則執行   ▶ (子設定清單)
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **判定狀態** | 否（但通常要） | 條件清單。**全部** 條件都成立才視為「符合」（AND 邏輯）。空清單 = 永遠符合。 |
| **符合條件則執行** | 是（至少一項） | 條件成立時要執行的子設定清單。 |
| **不符合條件則執行** | 否 | 條件不成立時要執行的子設定清單。可以為空（不符合就什麼都不做）。 |

## 行為說明

### 條件如何判定
*   全部條件都通過才算「符合」（AND 結合）。
*   清單為空時 = **永遠符合** → 永遠走「符合條件則執行」這條路徑。
*   每個條件本身有自己的設定（例如「擊殺後」「HP 比例」「持有特定狀態」等），請查看各自的條件類型說明。

### 戰鬥中發生什麼
1. 觸發此設定時計算條件結果。
2. 符合 → 依序執行「符合條件則執行」清單中的所有**已啟用**子設定。
3. 不符合 → 改執行「不符合條件則執行」清單中的所有**已啟用**子設定。
4. 子設定清單內未啟用（沒打勾）的會被自動跳過。

### 描述會怎麼顯示
*   只有「符合條件則執行」+ 有條件：「**若 {條件}，則 {符合內容}**」
*   有「不符合條件則執行」：另起一行接「**否則 {不符合內容}**」
*   多個條件之間以「**且**」串接

### 預覽傷害的限制
卡片右上角的預覽傷害會**先試算條件**，挑對應的路徑取最大值顯示。但 — 

> [!WARNING]
> 對於「**結果型條件**」（例如「**若擊殺**」「**若觸發暴擊**」），預覽時動作還沒發生，無法準確得知結果。預覽顯示可能與實際傷害不符。**若卡片要對玩家承諾傷害數字，請改用穩定條件**（如 HP 比例、層數比較、持有狀態等）。

## 注意事項

*   **判定狀態為空 + 不符合條件則執行有東西** = **設計反模式**：條件永遠成立，「不符合」那條路徑永遠不會跑，但描述會顯示出來，玩家會疑惑。**至少加一條判定狀態，或刪掉「不符合條件則執行」的內容。**
*   **避免巢狀過深**：條件判斷裡再放條件判斷可以，但描述會變成「若 A 則：若 B 則：⋯」 — **建議用多條件 AND 取代巢狀**。
*   **條件之間是 AND，不是 OR**：要 OR 邏輯目前需用兩個「條件判斷」分別處理（或請程式擴充 OR 條件容器）。
*   **未啟用的子設定**：在「符合 / 不符合」兩條路徑內，沒打勾的設定會被略過，但**仍會出現在描述合成中**（部分情況），請確認資料整潔。

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ConditionalSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_ConditionalSetting` → 「條件判斷」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_CardConditions` | 判定狀態 | `List<RCG_Condition>` | `CardConditions` | AND 結合；空清單 = 永遠 true |
| `m_IfConditionFit` | 符合條件則執行 | `List<RCG_BattleSetting>` | `IfConditionFit` | |
| `m_ElseCondition` | 不符合條件則執行 | `List<RCG_BattleSetting>` | `ElseCondition` | |
| `IfConditionFit` (property) | — | `List<RCG_BattleSetting>` | — | `m_IfConditionFit.GetEnableBattleSettings()` |
| `ElseCondition` (property) | — | `List<RCG_BattleSetting>` | — | `m_ElseCondition.GetEnableBattleSettings()` |

### A.3 重要 Method 摘要
*   **`AddAction`**：在 `AddActionTrigger` 中以 `m_CardConditions.CheckConditions_AND(iData)` 判斷，分流呼叫 `IfConditionFit` 或 `ElseCondition` 中各子設定的 `AddAction(iData, InsertInOrder)`。
*   **`Infos`** → 聚合 `m_CardConditions[*].Infos` + `IfConditionFit[*].Infos` + `ElseCondition[*].Infos`，全部 `AppendIfNotRepeat` 去重。
*   **`GetBattleSettings<T> / (Type)`** → 自身 + 遞迴兩條路徑（**注意 OR 都遞迴，不挑分支**）。
*   **`GetDescriptionFormat`**：
    1. 暫存 `iData.m_FullSentence`，設 `false`。
    2. 有 If 子設定 + 有條件 → 套 `IfCardConditions` 模板「若 {ConditionDes}」。
    3. 套 `ConditionFitThenDes` 模板「則 {IfDescription}」。
    4. 有 Else → 換行接 `ElseTriggerEffectsDes`「否則 {ElseDescription}」。
    5. 復原 `m_FullSentence`，套 `iData.GetDescription` 句尾修飾。
*   **`ConditionDes` (private property)** → 將 `m_CardConditions[*].Description` 用 `WordSeperator + ConditionAnd + WordSeperator` 串起來。
*   **`GetPreviewDamage`** → 直接以當前 `iData` 跑 `CheckConditions_AND`，挑對應分支取 `Mathf.Max`（**對結果型條件不準**）。
*   **`PreloadData`** → 兩條路徑各自 `await` 子設定的 `PreloadData`。
*   **`GetFusionCandidateSettings`** → 兩條路徑下所有 `IsEnable = true` 的子設定的候選聚合（**不含自身**）。
*   **`GetFusionBaseSetting`** → clone 自身結構，重建兩條路徑為 placeholder 化版本，過濾掉 disable。

### A.4 與其他系統的互動
*   **`RCG_Condition`** → 條件判斷的基底；具 `Description` / `CheckCondition(iData)` 等。
*   **`CheckConditions_AND` (extension)** → `List<RCG_Condition>` 的擴充方法，全 true 才回傳 true。
*   **i18n keys**：`IfCardConditions` / `ConditionFitThenDes` / `ElseTriggerEffectsDes` / `ConditionAnd`。
*   **`LocalizedStringUtils.WordSeperator()`** → 中文無空白、英文加空白的語境感知分隔符。

### A.5 已知議題
*   舊版的 `ElseTriggerEffects` 邏輯（純串接 + 首字大寫）以 `// ` 註解保留作對比。
*   `m_FullSentence` 暫存還原是**手動操作**，若中途 throw exception 會洩漏狀態 — 改 `try/finally` 較穩。
*   `GetPreviewDamage` 對「結果型條件」（`OnKill` 等）的不準特性是設計取捨，請於設計時迴避。
