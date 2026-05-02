---
title: 清除意圖 說明
description: 清除目標單位的下回合意圖（敵人 AI 的攻擊計畫）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 清除意圖

> 程式類別名稱：`RCG_ClearIntentSetting`

## 用途
**清除敵人的下回合意圖**（Intent）— 也就是頭頂顯示的「下回合要做的動作」。常見用途：
*   「使敵人的下次攻擊失效」風格的卡
*   「打斷」「干擾」類的設定

清除後敵人會在下次決策時重新計算意圖。

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **目標** (`Target`) | 是 | 要清除意圖的目標選擇器；可選敵方單體或全體。 |

## 行為說明
*   對每個目標（**有 AI 才有效**）呼叫 `target.UnitAI.ClearIntent()`。
*   描述格式為「**清除 {目標} 的意圖**」（i18n key `ClearIntentDes`）。

## 注意事項
*   **無 AI 的目標跳過**：友軍角色通常無 AI，使用此設定對友軍無效。
*   **不會立即重決策**：清除後**等下次 AI 決策時機才會生成新意圖**；當回合內敵人不會立刻重新攻擊。
*   **意圖中已執行的部分不會回退**：如果敵人已經開始打出某段攻擊，清除意圖只會影響尚未發生的部分。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ClearIntentSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_ClearIntentSetting` → 「清除意圖」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_Target` | 目標 | `RCG_SelectTargetData` | `Target` | |

### A.3 重要 Method 摘要
*   **`AddAction`**：對每個 target 檢查 `target.HasAI`，是的話呼叫 `target.UnitAI.ClearIntent()`。
*   **`GetDescriptionFormat`** → i18n key `ClearIntentDes`。

### A.4 與其他系統的互動
*   **`RCG_BattleUnit.HasAI / UnitAI.ClearIntent`**：AI 系統的意圖清除入口。
