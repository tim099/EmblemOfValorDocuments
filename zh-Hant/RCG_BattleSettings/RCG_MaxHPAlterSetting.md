---
title: 最大生命值變動 說明
description: 修改目標的最大 HP；可選永久或本場戰鬥兩種範圍
last_updated: 2026-05-05
target_audience: [Designer, Modder, AI_Agent]
---

# 最大生命值變動

> 程式類別名稱：`RCG_MaxHPAlterSetting`

## 用途
**修改目標的最大 HP**。常見用途：
*   永久增加角色血量（戰鬥獎勵 / 等級提升）
*   本場戰鬥內提升血量（暫時 Buff，戰後恢復）

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **AlterType** | 是 | 變化範圍：<br>• **Permanently** — 永久增加最大 HP（戰後保留）<br>• **ThisBattle** — 只在本場戰鬥（戰後恢復） |
| **值** (`Amount`) | 是 | 變化量（變數可變，可正可負）。 |
| **目標** (`Target`) | 是 | 目標單位選擇器。 |

## 行為說明
*   對未死亡的目標呼叫 `target.AlterMaxHP(AlterType, Amount)`。
*   描述格式：「**{Permanently/ThisBattle} 增加 {Amount} 最大 HP 給 {Target}**」（i18n key `MaxHPIcon_Des`，含心型圖示）。
*   執行後等待 0.6 秒讓 UI 動畫播完。

### 卡片融合
兩張本設定融合會把 Amount 加總（**不檢查 AlterType 是否相同**，可能會踩坑）。

## 注意事項
*   **AlterType 不對齊的融合**：永久 + 本場融合後 AlterType 取**前者**，可能造成「永久升級被誤標為暫時」。
*   **負值 Amount = 削減 HP 上限**：合法，但同時會把目標當前 HP 上限拉低（**若當前 HP 超過新上限會被截斷？** — 詳情查 `AlterMaxHP` 實作）。
*   **死亡目標跳過**：已死亡的目標不會被修改，避免死人加血。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：[`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_MaxHPAlterSetting.cs`](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_MaxHPAlterSetting.cs)
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_MaxHPAlterSetting` → 「最大生命值變動」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_AlterType` | `AlterType` | `AlterType` (檔內 enum) | — | `Permanently` / `ThisBattle` |
| `m_Amount` | 值 | `IntVariable` | `MaxHP` (=「生命值上限」) | `[UCL_FieldOnGUI]` 已被註解 |
| `m_Target` | 目標 | `RCG_SelectTargetData` | `Target` | |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：對 `aTargets` 中未死亡的呼叫 `aTarget.AlterMaxHP(m_AlterType, aAmount)`，後 `await Delay(0.6f)`。
*   **`Fusion`**：clone + `IntVariable.FuseAdd(m_Amount)`，**不檢查 `m_AlterType` 一致性**。
*   **`GetDescriptionFormat`** → i18n key `MaxHPIcon_Des`，4 個參數（Amount / Target / 心型 sprite / AlterType localize name）。

### A.4 與其他系統的互動
*   **`RCG_BattleUnit.AlterMaxHP(AlterType, int)`**：實際的修改入口；負責處理當前 HP / 上限的同步。
*   **`EffectIcon.Health`**：心型圖示。

### A.5 已知議題
*   舊版 `CanEnhence / Enhence` 已註解；強化系統重構中。
