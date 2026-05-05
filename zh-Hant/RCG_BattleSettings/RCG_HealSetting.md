---
title: 治療設定說明
description: 對選定目標恢復生命值的戰鬥設定，支援固定值與百分比兩種治療方式
last_updated: 2026-05-05
target_audience: [Designer, Modder, AI_Agent]
---

# 治療

> 程式類別名稱：`RCG_HealSetting`

## 用途
對**指定目標**恢復生命值。最簡單的「回血卡」「治療技」「自我修復狀態」都用這個設定。

## 編輯器中的樣貌
```
▼ ✓ [治療(Heal)] 回復 1 點 HP 給 (目標)
    HealType    [數值 / 百分比]
    治療量      [數值] 1
    治療目標    ▶ (展開後設定目標範圍)
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **HealType** | 是 | 治療方式：<br>• **數值 (Amount)** — 直接回復指定點數，例：「回 5 點 HP」。<br>• **百分比 (Percentage)** — 依目標最大 HP 的百分比回復，例：「回 30% HP」。 |
| **治療量** | 是 | 數字（支援變數綁定）。HealType 為「數值」時是點數，為「百分比」時是 0~100 的百分比值。 |
| **治療目標** | 是 | 治療對象選擇器。可以是「自身」「我方全體」「選中目標」等多種型態，由「**目標選擇器**」決定。 |

> [!NOTE]
> **治療量** 支援「變數」（例如「等於剩餘卡片數」「等於某狀態層數」），不限定常數。下拉「數值/變數」即可切換。

## 行為說明

### 戰鬥中發生什麼
1. 系統解析「治療目標」拿到實際目標清單。
2. 若清單非空，依 **HealType** 對每個目標執行回血。
3. UI 上會跳出綠色數字（生命值上升）。

### 描述會怎麼顯示
*   **HealType = 數值**：「回復 {治療量} 點 HP 給 {治療目標}」（搭配紅心圖示）
*   **HealType = 百分比**：「回復 {治療量}% HP 給 {治療目標}」

### 卡片融合
兩張「治療」卡融合時，**治療量會直接相加**（其他欄位以前者為準）。例如：「回 3 點」+「回 5 點」融合 → 「回 8 點」。

## 注意事項

*   **不要把「治療」用在攻擊牌上湊「攻擊+回血」**：請改用「**組合效果**」包覆，把「攻擊」和「治療」分別作為子設定 — 兩者語意正交，不該混在一起。
*   **治療量為負值**：技術上不會 crash，但語意錯誤（治療打負數變傷害？）。**要造成傷害請改用「攻擊」設定**，不要鑽這個漏洞。
*   **治療目標選「敵方」**：合法但奇特（給敵方回血通常是 debuff 或玩笑卡）；確認這是設計意圖再用。

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊
*   **檔案路徑**：[`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_HealSetting.cs`](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_HealSetting.cs)
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_HealSetting` → 「治療」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_HealType` | `HealType`（未本地化） | `HealType` (enum) | — | 列舉值有 i18n：`HealType_Amount` (數值)、`HealType_Percentage` (百分比) |
| `m_HealAmount` | 治療量 | `IntVariable` | `HealAmount` | `[UCL.Core.PA.UCL_FieldOnGUI]` |
| `m_HealTarget` | 治療目標 | `RCG_SelectTargetData` | `HealTarget` | 取代舊的 `m_HealRange`（已淘汰） |

### A.3 重要 Method 摘要
*   **`AddAction(TriggerEffectData, AddActionMode)`**：
    1. `m_HealTarget.GetTargets(iData)` 解析目標。
    2. 非空時建立 `RCG_PlayerHealAction(m_HealType, User, targets, m_HealAmount.GetValue, iData)`。
    3. 以 `AddActionMode.InsertAction` 插入。
*   **`GetDescriptionFormat`**：
    *   `HealType.Amount` → i18n key `HealIcon_Des`，組合 `{Heal}` + `{Target}` + 心形圖示。
    *   其他 → `m_HealType.GetLocalizeDes(...)` 拿對應格式串。
    *   `iTriggerEffectData.GetDescription` 套句尾修飾。
*   **`GetDescriptionShort`**：`Percentage` → `{Heal}%`；其他 → `{Heal}`。
*   **`Fusion(other)`**：必須對方也是 `RCG_HealSetting`；clone 自身並用 `IntVariable.FuseAdd` 合併 `m_HealAmount`。

### A.4 與其他系統的互動
*   **`RCG_PlayerHealAction`**：實際的 Action class；負責動畫播放與生命值修改。
*   **`RCG_SelectTargetData`**：通用目標選擇器，整個 BattleSetting 系統共用；`GetTargets(iData)` 是入口。
*   **`IntVariable`**：可序列化的數值容器，支援常數 / 變數 / 隱藏變數 三種型態。

### A.5 已知議題
*   舊的 `m_HealRange` 欄位與註解中的 `DeserializeFromJson` 處理已被遺棄；目前完全走 `m_HealTarget`。
*   舊版 `GetDescription` 註解保留作為對比參考（`//QWQ QWQ` 標記）。
