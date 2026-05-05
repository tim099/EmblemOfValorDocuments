---
title: 卡牌轉換 說明
description: 把指定的卡牌轉換為另一種卡牌（手牌、選中、本卡三種來源）
last_updated: 2026-05-05
target_audience: [Designer, Modder, AI_Agent]
---

# 卡牌轉換

> 程式類別名稱：`RCG_CardConvertSetting`

## 用途
把指定的卡牌**整批轉換**成另一種卡牌。常見用途：
*   「將手牌全部變為亡者卡」
*   「將這張卡變為強化版」
*   「將選中的卡轉為廢牌」

也用於系統內部 — 角色死亡時自動把不能再打出的卡轉為「亡者卡」。

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **ConvertType** | 是 | 要轉換的來源：<br>• **SelectedCards** — 玩家選擇的手牌<br>• **AllHandCards** — 全部手牌<br>• **ThisCard** — 觸發此效果的卡片本身 |
| **ConvertTarget** | 是 | 要轉換成的目標卡牌（`RCG_CardGenData`）。 |

## 行為說明
*   描述格式依 `ConvertType` 套不同 i18n key（`CardConvert_SelectedCards` / `CardConvert_AllHandCards` / `CardConvert_ThisCard`）。
*   執行時：
    1. 依 ConvertType 收集要轉換的卡牌清單。
    2. 對每張卡 `aDeck.Convert(舊卡, 新卡)` 替換。
    3. **手牌中的卡**會播放轉換動畫（每張間隔 0.2 秒），其他位置（牌堆）的直接替換無動畫。

## 注意事項
*   **ThisCard 只在卡牌觸發時有效**：若被狀態 / 戰鬥事件觸發，沒有「這張卡」的概念，會找不到目標。
*   **SelectedCards 必須事先選好**：通常前面要接一個「**選取手牌**」設定來填入 `iData.SelectedHandCards`。
*   **轉換目標不存在 ID 會 NRE**：請確認 `ConvertTarget` 指向的卡片 ID 已註冊。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：[`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardConvertSetting.cs`](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardConvertSetting.cs)
*   **繼承自**：`RCG_BattleSetting`
*   **`[System.Serializable]`** 標記
*   **i18n 類別名 key**：`RCG_CardConvertSetting` → 「卡牌轉換」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_ConvertType` | `ConvertType` | `ConvertType` (enum) | — | `SelectedCards` / `AllHandCards` / `ThisCard` |
| `m_ConvertTarget` | `ConvertTarget` | `RCG_CardGenData` | — | 目標卡牌模板 |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：
    1. 依 `m_ConvertType` 收集 `aCards`（手牌 / 選中 / 本卡）。
    2. 對每張 clone 出新 `RCG_CardBattleData`，呼叫 `aDeck.Convert(舊, 新)`。
    3. 手牌類的卡片播放 `CardConvertAnim` 動畫（間隔 0.2 秒），其餘直接替換。
*   **`RemoveDeadUnitCards(triggerEffectData)` (static)**：角色死亡 hook — 找出 `!HaveRequireSkills(PlayerSkillTags)` 的卡，全部轉成 `RCG_CardData.DeadCardID`。
*   **`LocalizeKey`** → `"CardConvert_" + m_ConvertType.ToString()`，描述依此查表。

### A.4 與其他系統的互動
*   **`RCG_Player.Ins.Deck.Convert`**：實際執行卡片替換。
*   **`RCG_CardBattleData.CreateCard(target)`**：clone 出新卡實例。
*   **`RCG_Card.CardConvertAnim`**：手牌轉換動畫。
*   **`RCG_CardData.DeadCardID`**：死亡時自動轉換的目標。
