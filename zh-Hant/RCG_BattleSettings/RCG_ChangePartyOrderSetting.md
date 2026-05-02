---
title: 改變隊伍順序 說明
description: 切換我方隊伍的角色出戰順序（前排輪替或將指定角色拉前）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 改變隊伍順序

> 程式類別名稱：`RCG_ChangePartyOrderSetting`

## 用途
**改變我方角色的排序**，讓不同角色登場戰鬥。常見用途：
*   「換手卡」— 把第一名輪到最後
*   「拉前」— 把指定角色（通常是受到威脅的）拉到第一位

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **ChangePartyOrderType** | 是 | 排序變更方式：<br>• **FirstToLast** — 第一個角色排到最後（後輪輪替）<br>• **TargetToFirst** — 把選中角色拉到第一個 |
| **目標** (`Target`) | 是 | 目標選擇器（`TargetToFirst` 模式必須是單體）。 |

## 行為說明
*   `FirstToLast` 直接呼叫 `RCG_BattleManager.Ins.SwitchCharacterUI.FirstToLast()`。
*   `TargetToFirst` 對單體目標：播放「光點」VFX 後切換 UI。
*   描述會根據 ChangePartyOrderType 套不同 i18n（`ChangePartyOrderDes_FirstToLast` / `ChangePartyOrderDes_TargetToFirst`）。

## 注意事項
*   **TargetToFirst 多目標**：選擇器若回傳 0 或多個目標，行為**不一定如預期**（程式只在 `aTargets.Count == 1` 時播 VFX，其他情況可能跳過）。請選擇器設成「單體」。
*   **戰鬥節奏**：切換順序通常代表「下一個行動者改變」，請確認後續觸發點是否依賴正確的當前角色。
*   **永遠展開**：此設定有 `[AlwaysExpendOnGUI]`，Inspector 中**永遠不會折疊**。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ChangePartyOrderSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **`[System.Serializable] + [AlwaysExpendOnGUI]`** 標記
*   **i18n 類別名 key**：`RCG_ChangePartyOrderSetting` → 「改變隊伍順序」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_ChangePartyOrderType` | `ChangePartyOrderType` | `ChangePartyOrderType` (檔內 enum) | — | `FirstToLast` / `TargetToFirst` |
| `m_Target` | 目標 | `RCG_SelectTargetData` | `Target` | 取代舊的 `m_Range`（已淘汰） |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：依 ChangePartyOrderType 分流。`TargetToFirst` 在 `aTargets.Count == 1` 時 `RCG_VFXManager.CreateVFX(VFX_LightTarget)` 並切換。
*   **`GetDescriptionShort`** → 直接回傳 i18n key `RCG_ChangePartyOrderSetting`（=「改變隊伍順序」）作為精簡標籤。

### A.4 與其他系統的互動
*   **`RCG_BattleManager.Ins.SwitchCharacterUI`**：實際的隊伍切換 UI / 邏輯。
*   **`CommonVFX.VFX_LightTarget`**：拉前的視覺效果。
