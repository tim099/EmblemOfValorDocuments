---
title: 觸發意圖 說明
description: 強制觸發目標的下回合行動（攻擊提前 / 模仿敵人技能等）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 觸發意圖

> 程式類別名稱：`RCG_TriggerIntentSetting`

## 用途
**強制觸發目標單位的下回合意圖**（即頭頂顯示的攻擊計畫）。常見用途：
*   「強制敵人提前出招」（讓玩家可以預判反擊）
*   「模仿敵方技能」（複製目標的行動）
*   「觸發後刷新意圖」（觸發完換新的攻擊計畫）

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **TriggerIntentType** | 是 | 觸發類型：<br>• **Default** — 觸發意圖<br>• **TriggerAndRefresh** — 觸發後刷新（換新意圖）<br>• **ImitateIntention** — 模仿目標意圖（讓使用者執行目標的行動） |
| **目標** (`Target`) | 是 | 目標單位選擇器；無 AI 的目標會被過濾。 |

## 行為說明
*   解析 `Target` 並過濾出有 AI 的目標，依 `BattleID` 排序確保穩定。
*   **Default / TriggerAndRefresh**：對每個目標插入 `RCG_IntentAction(target, type)` Action，**插入位置為 Last**（堆到佇列末端，等其他動作完成後執行）。
*   **ImitateIntention**：取**第一個目標**的 AI 當前 `Action.m_UnitAction`，讓使用者（`iData.User`）作為發動者執行 — 模仿其攻擊。

## 注意事項
*   **無 AI 目標自動跳過**：友軍角色通常無 AI，本設定對友軍無效。
*   **ImitateIntention 只取一個目標**：多目標選擇器會取第一個，其他被忽略。
*   **與「清除意圖」的對比**：清除是讓敵人**重決策**；本設定是**強制執行當前意圖**。
*   **Last 插入位置**：意圖觸發會**等其他動作完成後**才執行 — 與 `InsertInOrder` 不同，注意串接順序。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_TriggerIntentSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_TriggerIntentSetting` → 「觸發意圖」
*   **同檔頂層 enum**：`ETriggerIntentType` (`Default`, `TriggerAndRefresh`, `ImitateIntention`)

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_TriggerIntentType` | `TriggerIntentType` | `ETriggerIntentType` | — | 3 種類型 |
| `m_Target` | 目標 | `RCG_SelectTargetData` | `Target` | |

### A.3 重要 Method 摘要
*   **`AddAction`**：`m_Target.GetTargets(iData).Where(HasAI).OrderBy(BattleID)`，依 `m_TriggerIntentType` 分流：
    *   `ImitateIntention` → 取 `aTargets.FirstOrDefault().UnitAI.CurrentAction.m_UnitAction.AddAction(iData, InsertInOrder)`。
    *   其他 → 對每個 target `iData.AddAction(new RCG_IntentAction(target, m_TriggerIntentType), AddActionMode.Last)`。
*   **`GetDescriptionFormat`** → `m_TriggerIntentType.GetLocalizeDes(target)`。

### A.4 與其他系統的互動
*   **`RCG_BattleUnit.HasAI / UnitAI.CurrentAction`**：AI 系統的當前行動查詢。
*   **`RCG_IntentAction`**：實際觸發意圖的 Action class。
*   **`AddActionMode.Last`**：佇列尾端插入，與 `InsertInOrder` 行為不同。
