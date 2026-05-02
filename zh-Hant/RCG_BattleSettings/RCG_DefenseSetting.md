---
title: 護甲 說明
description: 對目標增加 / 減少 / 清除 / 倍增 / 衰減 / 設定護甲層數
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 護甲

> 程式類別名稱：`RCG_DefenseSetting`

## 用途
**修改護甲層數**。最常見的「防禦卡」「Buff 卡」核心，支援多種變化模式。

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **DefenseAlterType** | 是 | 變化模式：<br>• **Add** — 增加護甲<br>• **Sub** — 減少護甲<br>• **Clear** — 清空護甲<br>• **Double** — 倍增護甲<br>• **Decay** — 自然衰減（**可被某些狀態阻止**，與 Clear 不同）<br>• **Set** — 設為指定值 |
| **護甲** (`Defense`) | Add/Sub/Set 必填 | 護甲量（變數可變）。`Conditional` 控制顯示 — `Clear/Double/Decay` 模式會自動隱藏。 |
| **DefenseTarget** | 是 | 護甲套用目標選擇器（取代舊的 `m_Range`）。 |

## 行為說明
*   依 DefenseAlterType 套用：
    *   `Add` / `Sub` / `Set` — 直接加減 / 設定數值
    *   `Clear` — 強制清零（**不受任何狀態保護**）
    *   `Decay` — 衰減（**可被某些狀態（如「持盾」）阻止**）
    *   `Double` — 倍增當前護甲
*   描述格式依模式套不同 i18n key（`DefIcon_DesAdd` / `DefIcon_DesSub` / `DefIcon_DesClear` / `DecayDes` / ...）+ 護甲圖示。

## 注意事項
*   **Decay vs Clear 的差別**：`Clear` 強制歸零，`Decay` 可被防衰減狀態阻止 — 設計回合結束的自然衰減一律用 `Decay`。
*   **Sub 不會變負值**：護甲下限為 0；扣超過會卡在 0 不會變負。
*   **永遠展開**：`[AlwaysExpendOnGUI]`。
*   **與「狀態」的差別**：護甲是**獨立的數值**，不算狀態層數；查詢層數類條件不會看到護甲。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_DefenseSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **`[System.Serializable] + [AlwaysExpendOnGUI]`** 標記
*   **i18n 類別名 key**：`RCG_DefenseSetting` → 「護甲」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_DefenseAlterType` | `DefenseAlterType` | enum (檔內) | — | 6 種模式 |
| `m_Defense` | 護甲 | `IntVariable` | `Defense` | `[UCL_FieldOnGUI]` + `[Conditional(m_DefenseAlterType, false, Add, Sub, Set)]` |
| `m_DefenseTarget` | `DefenseTarget` | `RCG_SelectTargetData` | — | 取代舊的 `m_Range` |

### A.3 重要 Method 摘要
*   **`AddAction`**（檔案 90 行外）：依 DefenseAlterType 對 `m_DefenseTarget.GetTargets(iData)` 套用變化。
*   **`GetDescriptionFormat`**：依 DefenseAlterType 套不同 i18n key（`DefIcon_DesXxx` / `DecayDes` 等）。
*   **`Infos`**：`IsShowOnUI` 啟用時加入護甲圖示資訊。

### A.4 與其他系統的互動
*   **`EffectIcon.Armor.GetTMPSpriteName()`**：描述中的護甲圖示。
*   **`RCG_BattleUnit` 護甲層**：實際的護甲容器；衰減判定可能由 `m_UnitStatus` 中的 buff 阻止。
