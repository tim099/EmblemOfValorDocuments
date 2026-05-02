---
title: 移動 說明
description: 移動單位到指定位置（前排 / 後排 / 對面 / 指定位置）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 移動

> 程式類別名稱：`RCG_MoveSetting`

## 用途
**移動單位**到指定位置。常見用途：
*   「換位卡」— 把後排角色拉到前排
*   「衝鋒」— 角色移動到敵方位置
*   「撤退」— 移動到後排避戰

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **MoveType** | 是 | 移動方式：<br>• **ToOther** — 移動到另一排（前↔後切換，預設）<br>• **ToFront** — 移動到前排<br>• **ToBack** — 移動到後排<br>• **ToTarget** — 移動到目標位置（與 `TargetPositions` 配合） |
| **MoveTarget** | 是 | 要移動的單位選擇器。 |

## 行為說明
*   `ToFront` / `ToBack` / `ToOther` — 對每個目標各自移動到對應排：
    *   `ToOther` → 在前排則去後排，反之亦然
    *   `ToFront` → 一律到前排
    *   `ToBack` → 一律到後排
*   `ToTarget` — 配合外部設定的 `TargetPositions`，**最多移動 N 個單位到 N 個位置**（按索引配對）。
*   描述格式：「**{MoveTarget} 移動 {MoveType}**」（i18n key `Move_Des`）。

## 注意事項
*   **死亡目標跳過**：每次移動前會檢查 `IsNullOrDead`，避免操作死人。
*   **ToTarget 模式需要 `TargetPositions`**：通常前面要接「**設定目標**」(`RCG_SetTargetSetting`) 或類似設定填入 `iData.m_TriggerData.m_TargetPositions`，否則沒地方可去。
*   **目標位置已有單位**：技術上由 `RCG_BattleField.MoveUnit` 決定 — 多半會推擠或交換位置。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_MoveSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_MoveSetting` → 「移動」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_MoveType` | `MoveType` | `UnitMoveType` (檔內 enum) | `MoveType` (=「移動行為」) | 4 種模式 |
| `m_MoveTarget` | `MoveTarget` | `RCG_SelectTargetData` | — | |

### A.3 重要 Method 摘要
*   **`AddAction`**：依 `m_MoveType` 分流：
    *   `ToTarget` → `iData.m_TriggerData.m_TargetPositions` 與 `aTargets` 按索引配對，呼叫 `RCG_BattleField.Ins.MoveUnit(target, position, token)`。
    *   其他 → 對每個 target 計算 `aTargetPosition`（前排 / 後排 / 對面），同樣 `MoveUnit`。
*   **`GetDescriptionShort`** → 直接顯示移動 sprite (`EffectIcon.Move`)。

### A.4 與其他系統的互動
*   **`RCG_BattleField.Ins.MoveUnit(unit, pos, token)`**：實際的移動入口（含動畫）。
*   **`UnitPos`**：前 / 後 enum。
*   **`iData.m_TriggerData.m_TargetPositions`**：`ToTarget` 模式的目標位置來源。
