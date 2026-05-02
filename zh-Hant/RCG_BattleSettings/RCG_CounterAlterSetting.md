---
title: 計數器效果 說明
description: 修改觸發來源（裝備 / 單位技能）的內部計數器數值
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 計數器效果

> 程式類別名稱：`RCG_CounterAlterSetting`

## 用途
**修改觸發來源的計數器**。「計數器」是裝備或單位技能上的內部數值（例如「使用次數」「累積層數」）。常見用途：
*   「裝備：每受到 3 次傷害觸發一次」 — 計數器在累積與重置間切換
*   「裝備：每次戰鬥可用 X 次」 — 用計數器追蹤剩餘次數

> [!IMPORTANT]
> 此設定**只能在裝備或單位技能的觸發效果中使用** — 它依賴 `iData.EffectTriggerSource` 是 `RCGI_EffectCounter` 才有效。一般卡牌觸發**無計數器可改**。

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **CounterEffectType** | 是 | 計數器所屬類型：`UnitSkill`（單位技能）或 `Equipment`（裝備）。 |
| **CounterId** | 是 | 計數器索引（同一裝備可有多個計數器，0、1、2...）。 |
| **CounterAlter** | 是 | 變化量（變數可變）。 |
| **CounterAlterType** | 是 | 變化模式：`Add` / `Sub` / `Set`。 |

## 行為說明
*   讀取 `iData.EffectTriggerSource`，確認其為 `RCGI_EffectCounter`，呼叫 `counter.AlterCounter(CounterId, CounterAlterType, CounterAlter)`。
*   描述格式：「**{CounterEffectType} 計數器 {Id+1} {CounterAlterType} {CounterAlter}**」（i18n key `CounterAlterDes`）。

## 注意事項
*   **EffectTriggerSource 不對會錯誤訊息**：在卡牌效果中觸發此設定，會在 console 印錯誤但不會 crash。**請確認上層是裝備或技能效果**。
*   **永遠展開**：此設定有 `[AlwaysExpendOnGUI]`，Inspector 中**永遠不會折疊**。
*   **CounterId 越界**：超出該裝備擁有的計數器數量會觸發未定義行為，請確認 ID 與裝備配置一致。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CounterAlterSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **`[System.Serializable] + [AlwaysExpendOnGUI]`** 標記
*   **i18n 類別名 key**：`RCG_CounterAlterSetting` → 「計數器效果」
*   **同檔案 enum**：`CounterEffectType { UnitSkill, Equipment }` / `CounterAlterType { Add, Sub, Set }`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_CounterEffectType` | `CounterEffectType` | enum | — | |
| `m_CounterAlterType` | `CounterAlterType` | enum | — | |
| `m_CounterId` | `CounterId` | `int` | — | 預設 0 |
| `m_CounterAlter` | `CounterAlter` | `IntVariable` | — | |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：取 `iData.EffectTriggerSource`，is-check 為 `RCGI_EffectCounter`，呼叫 `counter.AlterCounter(m_CounterId, m_CounterAlterType, m_CounterAlter.GetValue(iData))`。
*   **`GetDescriptionShort`** → i18n key `RCG_CounterAlterSetting`（=「計數器效果」）。
*   舊邏輯（`m_TriggerData.m_Equipment / m_UnitSkill` 分流）已註解。

### A.4 與其他系統的互動
*   **`RCGI_EffectCounter`**：計數器介面，由裝備 / 單位技能實作。
*   **`iData.EffectTriggerSource`**：觸發來源；裝備觸發效果時會被填上對應實例。
