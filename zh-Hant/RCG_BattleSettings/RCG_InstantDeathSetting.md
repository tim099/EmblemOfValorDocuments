---
title: 即死 說明
description: 直接擊殺目標；可選無條件即死或低於指定 HP 即死兩種模式
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 即死

> 程式類別名稱：`RCG_InstantDeathSetting`

## 用途
**直接擊殺目標**（無視 HP / 護甲）。常見用途：
*   「斬首」「處決」類超強卡：對血量低於 X 的目標即死
*   特殊事件「無條件擊殺」（劇情演出 / 詛咒）

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Type** | 是 | 即死模式：<br>• **Default** — 無條件即死全部目標<br>• **HP** — 目標 HP ≤ 指定值才即死 |
| **目標** (`Target`) | 是 | 即死目標選擇器。 |
| **HP** | Type=HP 必填 | 即死的血量門檻；目標血量 ≤ 此值才即死。`Conditional` 控制顯示。 |

## 行為說明
*   **Default**：對所有目標呼叫 `RCG_TermInstantDeath.TriggerInstantDeath(iData, targets)`（內含詞條觸發）。
*   **HP**：對每個目標獨立判定 `target.HP <= m_HP`，符合者呼叫 `target.InstantDeath()`。
*   描述格式：
    *   `Default` → 「**對 {目標} 即死**」
    *   `HP` → 「**血量低於 {HP} 時對 {目標} 即死**」

### 卡片資訊
顯示 `RCG_Term.GetTerm(Term.InstantDeath)` 詞條框（解釋即死機制）。

## 注意事項
*   **Default 模式很強**：對 Boss 也直接擊殺。除非 Boss 有「無視即死」狀態，否則會被秒。設計時請慎用。
*   **HP 模式對 AOE 公平**：每個目標獨立判定，不會出現「整批 AOE 但只擊殺第一個」的問題（與其他即死實作的差異）。
*   **負值 HP**：`m_HP` 設負值會永遠不觸發（因為目標血量不會低於 0），等於「永遠不即死」。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_InstantDeathSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_InstantDeathSetting` → 「即死」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_Type` | `Type` | `InstantDeathType` (檔內) | — | `Default` / `HP` |
| `m_Target` | 目標 | `RCG_SelectTargetData` | `Target` | |
| `m_HP` | `HP` | `IntVariable` | `HP` (=「生命」) | `[Conditional(nameof(m_Type), false, HP)]`，預設 1 |

### A.3 重要 Method 摘要
*   **`AddAction`**：依 `m_Type` 分流：
    *   `Default` → `RCG_TermInstantDeath.TriggerInstantDeath(iData, targets)`。
    *   `HP` → 對每個 target 檢查 `target.HP <= m_HP.GetValue(iData)` 即呼叫 `target.InstantDeath()`。
*   **`Info`** → `new CardInfoData(RCG_Term.GetTerm(Term.InstantDeath))`。
*   **`GetDescriptionFormat`**：`InstantDeath_Des` (Default) / `InstantDeath_HP_Des` (HP)。

### A.4 與其他系統的互動
*   **`RCG_TermInstantDeath.TriggerInstantDeath`**：詞條觸發入口，含特效演出。
*   **`RCG_BattleUnit.InstantDeath`**：直接設定 HP=0 並走死亡流程的入口。
*   **`Term.InstantDeath`**：對應的詞條 enum。

### A.5 已知議題
*   `Condition` 模式（`m_Conditions`）已被註解掉 — 想做「條件式即死」目前要用「條件判斷」+「即死(Default)」組合。
