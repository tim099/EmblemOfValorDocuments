---
title: 計數器清除 說明
description: 一次清空觸發來源（裝備 / 單位技能）的所有計數器
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 計數器清除

> 程式類別名稱：`RCG_CounterClearSetting`

## 用途
**一次清空觸發來源的所有計數器**。比「計數器效果」更強硬：直接歸零**全部**而非單個。常見用途：
*   「裝備能力觸發後重置所有累積層」
*   「特定條件下強制清掉裝備充能」

> [!IMPORTANT]
> 同樣**只能在裝備或單位技能的觸發效果中使用**（依賴 `EffectTriggerSource` 是 `RCGI_EffectCounter`）。

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **CounterEffectType** | 是 | 計數器所屬類型：`UnitSkill` 或 `Equipment`。**目前僅作為描述顯示用**，實際清除是針對 EffectTriggerSource 全部計數器，不挑類型。 |

## 行為說明
*   呼叫 `counter.ClearAllCounters()` — 清除該觸發來源的**全部**計數器。
*   描述為「**清除 {CounterEffectType} 計數器**」（i18n key `CounterClearDes`）。

## 注意事項
*   **沒有 CounterId 欄位**：清除是「全部」性質，不能挑特定計數器。要清單一計數器請改用「**計數器效果**」+ `Set 0`。
*   **EffectTriggerSource 不對的錯誤**：與「計數器效果」相同，console 印錯但不 crash。
*   **永遠展開**：`[AlwaysExpendOnGUI]`。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CounterClearSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **`[System.Serializable] + [AlwaysExpendOnGUI]`** 標記
*   **i18n 類別名 key**：`RCG_CounterClearSetting` → 「計數器清除」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_CounterEffectType` | `CounterEffectType` | enum | — | 與 `RCG_CounterAlterSetting` 共用 |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：取 `iData.EffectTriggerSource`，is-check `RCGI_EffectCounter`，`counter.ClearAllCounters()`。
*   **`GetDescriptionFormat / GetDescriptionShort`**：均為 i18n key `CounterClearDes`，含 `m_CounterEffectType.GetLocalizeName()`。

### A.4 與其他系統的互動
*   **`RCGI_EffectCounter.ClearAllCounters`**：計數器全清入口。
