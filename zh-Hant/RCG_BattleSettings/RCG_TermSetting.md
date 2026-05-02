---
title: 詞條 說明
description: 觸發指定詞條（Term）所定義的效果；詞條由獨立的 RCG_Term 系統管理
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 詞條

> 程式類別名稱：`RCG_TermSetting`

## 用途
**觸發詞條（Term）效果**。「詞條」是遊戲中可重用的命名效果，例如「**消耗**」「**詠唱**」「**保留**」「**即死**」「**急速詠唱**」等 — 每個詞條都有獨立的觸發邏輯與卡片資訊面板。

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Term** | 是 | 要觸發的詞條 enum 值（預設 `HighSpeedChant` 急速詠唱）。 |

## 行為說明
*   觸發時呼叫 `RCG_Term.GetTerm(m_Term).TriggerEffect(iData, endAction)`。
*   描述顯示詞條的本地化名稱（綠色標題色）。
*   **`HasTerm(iTerm)` 回傳 true** 當 `iTerm == m_Term` — 上層辭條反查能找到此設定。
*   `Info` 顯示對應詞條的完整 Tooltip（除非 `Term.Empty`）。

## 注意事項
*   **詞條本身決定行為**：本設定只是「觸發**已定義的詞條**」 — 詞條邏輯在 `RCG_Term.GetTerm(...)` 那邊。要新增 Term 請查 Term 系統。
*   **Term.Empty**：合法但無效；常用於佔位或未來擴充。
*   **詞條觸發點**：「保留」「消耗」這類詞條本身有觸發時機（例如「打出時」「丟棄時」）— 本設定主動觸發 `TriggerEffect` 是另一回事，請確認詞條本身支援主動呼叫。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_TermSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_TermSetting` → 「詞條」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_Term` | `Term` | `Term` (enum) | `Term` (=「詞條」) | 預設 `HighSpeedChant` |

### A.3 重要 Method 摘要
*   **`AddAction`**：`RCG_Term.GetTerm(m_Term).TriggerEffect(iData, iEndAction)`。
*   **`HasTerm(Term iTerm)` (override)** → `iTerm == m_Term`；上層 `GetBattleSettings<RCG_TermSetting>` 後可進一步查辭條歸屬。
*   **`Info`** → `m_Term == Empty` 回 null；否則 `new CardInfoData(RCG_Term.GetTerm(m_Term))`。
*   **`TermDes` (private property)** → `LocalizeName.GetTagColor(TagColors.Term)`。

### A.4 與其他系統的互動
*   **`RCG_Term.GetTerm(Term)`**：詞條工廠；回傳對應的 Term 邏輯實例。
*   **`Term` enum**：詞條 ID 集合（`HighSpeedChant`、`InstantDeath`、`Retain`、`Empty`、...）。
*   **`RCG_Extensions.TagColors.Term`**：詞條色票。

### A.5 已知議題
*   舊版 `m_Term == Term.Retain` 的反序列化兼容已被註解（強制覆寫為 `Term.Empty` 的舊邏輯）。
