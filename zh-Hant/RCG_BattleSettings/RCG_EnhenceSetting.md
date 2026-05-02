---
title: 強化 說明
description: 對選取的手牌附加強化效果（OnPlay 時觸發），永久改造卡片行為
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 強化

> 程式類別名稱：`RCG_EnhenceSetting`

## 用途
**對選取的手牌附加額外效果**：被強化的卡打出時會在原本的效果之外，**多觸發一次** `EnhenceSetting`。常見用途：
*   「強化下一張攻擊卡：附加吸血效果」
*   「強化所有手牌：每張多抽 1 張卡」

被強化的卡會永久帶有這個效果，直到被「**無效卡牌效果**」削弱或卡片消失。

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **SelectHandCardSetting** | 是 | 選取要強化的手牌。 |
| **EnhenceSetting** | 是 | 附加上去的「**組合效果**」 — 觸發時機固定為 `OnPlay`。 |

## 行為說明
*   選牌後，對每張卡呼叫 `aCard.Enhence(commonEffect)`，把 `EnhenceSetting` 包進 `RCG_CommonEffect`（`OnPlay` 觸發點）並附加。
*   描述格式：「**{選取} : {強化標題} {強化內容描述}**」（i18n key `EnhenceSettingDes`，含 `EnhenceTitle` 顏色）。
*   附帶資訊框（`Infos`）：第一條為「強化說明」總覽，後接強化效果本身的 Infos。

### 標籤
強化設定不會把自己內含的標籤（`m_EnhenceSetting.GetBattleTags`）暴露在外層 — 是設計選擇，避免標籤被誤計入原卡。

## 注意事項
*   **強化是永久的**：一旦強化效果附加上去，會跟著卡到其消失或被削弱。
*   **觸發時機固定 `OnPlay`**：目前**不能改變**強化效果的觸發點。要其他時機請考慮用「條件判斷」+「狀態」實作。
*   **與「無效卡牌效果」的對應**：被削弱時優先消耗強化葉效果，本設定的內容首當其衝。
*   **不暴露標籤**：刻意設計 — 不要期待強化內含的「易碎」「敏捷」等標籤會出現在卡片資訊上。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_EnhenceSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_EnhenceSetting` → 「強化」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_SelectHandCardSetting` | `SelectHandCardSetting` | `RCG_SelectHandCardSetting` | — | `[SerializeField] protected` |
| `m_EnhenceSetting` | `EnhenceSetting` | `RCG_CombineSetting` | — | `[SerializeField] protected` |

### A.3 重要 Method 摘要
*   **`AddAction`**：選牌 → 對每張卡建 `RCG_CommonEffect { m_CombineSetting = m_EnhenceSetting, m_EffectTriggerOn.m_TriggerOn = OnPlay }`，呼叫 `aCard.Enhence(aEffect)`。
*   **`GetBattleTags`** → 永遠回傳空清單（**刻意覆寫**，不暴露 `m_EnhenceSetting` 的標籤）。
*   **`GetDescriptionFormat`**：「{選取 desc}\n{Title} : {Enhence desc + 內含 BattleTags}」。
*   **`GetEnhenceSettingDescription` (private)**：取 `m_EnhenceSetting.GetDescription` + 其 BattleTags 描述。
*   **`Infos`**：插入 `"RCG_EnhenceSetting" + "\n" + EnhenceSettingInfo(...)` 為第 0 項。

### A.4 與其他系統的互動
*   **`RCG_CommonEffect`**：強化效果包覆容器（含 `TriggerOn`）。
*   **`RCG_CardBattleData.Enhence(commonEffect)`**：實際的強化附加入口。
*   **`RCG_Extensions.TagColors.EnhenceTitle`**：標題顏色。
*   **i18n keys**：`RCG_EnhenceSetting` / `EnhenceSettingDes` / `EnhenceSettingInfo`。

### A.5 已知議題
*   舊版 `m_CostAlter` / `m_EffectTriggerTiming` 欄位已註解（強化系統重構中），目前無法調整費用變化或自訂觸發點。
