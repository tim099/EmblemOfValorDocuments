---
title: 場地效果 (RCG_FieldEffectData) 說明
description: 作用於整個戰場的環境效果模板（火焰場地、黑暗壟罩、各回合 -HP 等）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-en
---

> [!WARNING]
> Translation pending — this file needs an English translation.
The original zh-Hant content is included below for reference.


# 場地效果

> 程式類別名稱：`RCG_FieldEffectData`

## 用途

**作用於整個戰場的環境效果模板**。不像狀態 (CustomStatus) 是貼在單位身上，場地效果**全場通用**——例如「火焰場地：所有單位每回合 -3 HP」「黑暗：所有攻擊命中率 -20%」「神聖：所有治療 +50%」。

繼承自 `RCG_Asset<RCG_FieldEffectData>`，實作介面：`RCGI_Infos` / `UI.RCGI_StatusInfo`。

## 編輯器中的樣貌

```
RCG_FieldEffectData: <ID>
    Name(多國語言)
    Icon                 ← 場地效果圖示
    Effects              ← 觸發效果（依 trigger 套用到全場）
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Name** | 是 | 場地效果名（多語系） |
| **Icon** | 是 | 圖示，預設 `FieldEffectIcons_volcano.png` |
| **Effects** | 是 | 各 trigger 上要觸發的效果（`RCG_CommonEffect` list） |

## 行為說明

### 觸發
`OnUnitState(triggerOn, data)` → 從 `m_Effects` 取對應 trigger 的 effects 並依序觸發。**`OnUnitState` 命名是歷史遺留**——實際上場地效果不一定綁定 unit state，可在任何 trigger 觸發（OnBattleStart / OnTurnStart / OnTurnEnd 等）。

### 描述
自動由 `Effects` 串接（含 BattleTags 解說）。

## 注意事項

*   **沒有層數機制**：場地效果是「有 / 沒有」的二元狀態，不像 CustomStatus 有層數。
*   **多個場地效果**可疊加（透過 BattleManager 管理當前生效列表）；UI 上會列出所有 active 場地。
*   **抽取來源**：`RCG_BattleSceneData.m_FieldEffectDrops` 引用 `RCG_FieldEffectDropPool`，戰鬥開始時抽出本場場地效果。
*   **未實作 RCGI_Status 介面**：類別宣告有 `//, RCGI_Status` 註解標示曾規劃要實作但目前未做。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_FieldEffectData.cs`
*   **繼承自**：`RCG_Asset<RCG_FieldEffectData>`
*   **實作介面**：`RCGI_Infos` / `UI.RCGI_StatusInfo`
*   **AssetGroup**：`EditBattleSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Icon` | Icon | `RCG_SpriteData` | 預設 `FieldEffectIcons_volcano.png` |
| `m_Effects` | Effects | `List<RCG_CommonEffect>` | |

### A.3 重要 Method 摘要

*   **`OnUnitState(triggerOn, data)`** — 觸發效果（命名歷史遺留）。
*   **`TriggerOnUnitState(triggerOn)`** — quick check。
*   **`Description` (property)** — `m_Effects.GetDescription(showBattleTags = true)`。
*   **`Infos` (property)** — `m_Effects.GetInfos()`。

### A.4 與其他系統的互動

*   **`RCG_BattleSceneData.m_FieldEffectDrops`** — 戰鬥場景引用的場地效果池。
*   **`RCG_FieldEffectDropPool`** — 場地效果隨機池。
*   **`RCG_BattleManager.TriggerFieldEffect`** — runtime 觸發場地效果的入口。
*   **`RCG_FieldEffectGenData`** / **`RCG_FieldEffectDropPoolGenData`** — Asset Entry 包裝。

### A.5 已知議題

*   類別宣告 `, RCGI_Status` 是註解掉的——曾規劃但未實作此介面。
*   `m_AcquireSkillEvents` 等程式內被註解，疑似從 UnitSkill 複製過來時殘留的 dead code。