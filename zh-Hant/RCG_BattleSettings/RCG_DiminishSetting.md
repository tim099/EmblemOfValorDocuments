---
title: 無效卡牌效果 說明
description: 削弱選取的手牌：移除指定數量的葉效果，從強化效果優先消耗
last_updated: 2026-05-05
target_audience: [Designer, Modder, AI_Agent]
---

# 無效卡牌效果

> 程式類別名稱：`RCG_DiminishSetting`

## 用途
**削弱選取的手牌** — 移除一定數量的「葉效果」（最末端的單一動作，例如「攻擊」「治療」「抽牌」）。優先從強化效果開始消耗，耗盡後才動到基礎效果。常見用途：
*   敵人技能：「使你下一張牌效果減半」
*   詛咒卡：「目標卡的某項效果被消除」

被削弱的葉效果會被 `RCG_DiminishedPlaceholder` 取代（顯示「已被削弱」紅字）。

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **SelectHandCardSetting** | 是 | 選取要削弱的手牌。 |
| **DiminishEffectCount** | 是 | 要移除的葉效果數量（預設 1）。 |

## 行為說明
*   選牌後，對每張卡呼叫 `aCard.Diminish(token, DiminishEffectCount)`。
*   消耗順序：**強化效果**先消耗，耗盡再動到**基礎效果**。
*   每被消耗的葉效果會替換為 `RCG_DiminishedPlaceholder`，描述顯示為「已被削弱」（紅字）。

## 注意事項
*   **不可被進一步削弱**：被替換為 `RCG_DiminishedPlaceholder` 的葉效果**不能再次被削弱**（避免無限疊加）。
*   **葉效果不一定明顯**：「組合效果」「條件判斷」內部含多個葉效果；`DiminishEffectCount = 1` 不一定能砍到「最關鍵的那一個」。
*   **DiminishEffectCount 太大**：超過卡片葉效果總數時，整張卡幾乎全變 placeholder — 設計時請拿捏。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：[`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_DiminishSetting.cs`](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_DiminishSetting.cs)
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_DiminishSetting` → 「無效卡牌效果」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_SelectHandCardSetting` | `SelectHandCardSetting` | `RCG_SelectHandCardSetting` | — | `[SerializeField] protected` |
| `m_DiminishEffectCount` | `DiminishEffectCount` | `int` | — | `[SerializeField] protected`，預設 1 |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：選牌 → 對每張 `aCard.Diminish(iToken, m_DiminishEffectCount)`。
*   **`GetBattleTags`** → 永遠回傳空（被削弱的卡不繼承標籤）。
*   **`GetDescription`**：「{選取描述}\n{被削弱說明} ({DiminishEffectCount}個)」（i18n key `DiminishSettingDes`）。
*   **`GetDescriptionParams`**：包含 `Title`、`DC`（DiminishEffectCount 字串）、選取設定的子 params。

### A.4 與其他系統的互動
*   **`RCG_CardBattleData.Diminish(token, count)`**：實際削弱的入口；負責從強化端開始消耗葉效果並替換為 placeholder。
*   **`RCG_DiminishedPlaceholder`**：佔位類別。
*   **`RCG_Extensions.TagColors.EnhenceTitle`**：標題顏色（borrow 自強化系統）。
