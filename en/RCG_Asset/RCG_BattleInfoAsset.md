---
title: 戰鬥資訊資產 (RCG_BattleInfoAsset) 說明
description: 戰鬥日誌左側顯示的「即時統計資訊」設定（本回合打出手牌數、傷害總量等）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-en
---

> [!WARNING]
> Translation pending — this file needs an English translation.
The original zh-Hant content is included below for reference.


# 戰鬥資訊資產

> 程式類別名稱：`RCG_BattleInfoAsset`

## 用途

**戰鬥日誌左側顯示的即時統計資訊**。例如「本回合打出手牌：3」「累積傷害：120」「剩餘抽卡格：2」。每個統計項目是一個 `RCG_BattleInfoAsset` 實例；戰鬥 UI 會依 `m_Order` 排序並逐一顯示。

繼承自 `RCG_Asset<RCG_BattleInfoAsset>`。

## 編輯器中的樣貌

```
RCG_BattleInfoAsset: <ID>
    Enable
    InfoType   ▾ IntVariable
    Variable   ← InfoType=IntVariable 時顯示
    Order
    HideIfZero
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Enable** | 是 | 是否啟用此資訊（false 直接從顯示中拿掉） |
| **InfoType** | 是 | 資訊類型，目前只有 `IntVariable` |
| **Variable** | InfoType=IntVariable | 要顯示的整數變數（`IntVariable`），可綁定戰鬥計數器 |
| **Order** | 是 | 顯示順序，越小越前面 |
| **HideIfZero** | — | 為 0 時隱藏（避免一堆 0 洗版） |

## 行為說明

### `ShowInfo(data)`
*   `m_HideIfZero = true` 且 `Variable.GetValue() == 0` → 不顯示。
*   否則顯示。

### `GetInfo(data)`
依 `InfoType` 取得字串：
*   `IntVariable` → `m_Variable.GetDescription(iData)`（已格式化，含標籤前綴等）。

## 注意事項

*   **`InfoType` 目前只支援 `IntVariable`**：未來可能新增 String / Float 類型，但目前 enum 只有一個值。
*   **`m_Order` 衝突時**：兩個 Asset 同 Order 的顯示順序不保證；建議手動分開值。
*   **HideIfZero 對非 IntVariable 無效**：邏輯目前只 cover IntVariable case。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_BattleInfoAsset.cs`
*   **繼承自**：`RCG_Asset<RCG_BattleInfoAsset>`
*   **AssetGroup**：`EditBattleSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_Enable` | Enable | `bool` | 預設 `true` |
| `m_InfoType` | InfoType | `InfoType` enum | 目前只有 `IntVariable` |
| `m_Variable` | Variable | `IntVariable` | `Conditional(IntVariable)` |
| `m_Order` | Order | `int` | 預設 0；越小越前 |
| `m_HideIfZero` | HideIfZero | `bool` | 預設 `false` |

### A.3 重要 Method

*   **`ShowInfo(TriggerEffectData)`** — 是否要顯示；依 `m_HideIfZero` 與 Variable 值判斷。
*   **`GetInfo(TriggerEffectData)`** — 取格式化後的字串。

### A.4 與其他系統的互動

*   **`IntVariable`** — 變數綁定來源。
*   **戰鬥日誌左側 UI** — 顯示這些 Asset 的消費端。

### A.5 已知議題

*   `InfoType` enum 只有一個值，預示未來擴充計畫但尚未實作。