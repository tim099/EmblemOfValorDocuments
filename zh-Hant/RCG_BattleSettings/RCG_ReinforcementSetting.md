---
title: 增援 說明
description: 召喚增援單位（目前僅敵方）；普通戰鬥隨機抽，精英 / Boss 戰用預設模板
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 增援

> 程式類別名稱：`RCG_ReinforcementSetting`

## 用途
**召喚敵方增援單位**到戰場。目前**僅支援怪物增援**（不能用於玩家方）。常見用途：
*   Boss 召喚雜兵
*   特定回合自動補充敵人

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **DefaultUnit** | 是 | 預設增援單位（**精英 / 魔王戰專用**）。普通戰鬥不使用此欄位，會從 `MonsterSet` 隨機抽。 |

## 行為說明
*   讀取 `RCG_BattleField.Ins.MonsterSet`，取出當前敵人類型的可用單位清單。
*   **普通戰鬥**：從清單中隨機挑一個。
*   **精英 / Boss 戰**：直接用 `DefaultUnit` — 避免生成過多精英造成失衡。
*   優先在 `iData.TargetPositions` 指定的空位生成；無指定時自動找空位。
*   描述格式：「**召喚怪物增援**」（i18n key `ReinforcementDes_Monster`）。

## 注意事項
*   **目前僅怪物增援**：程式內 `bool aIsMonster = true` 寫死；要召喚我方友軍請用「**召喚**」設定。
*   **無空位時的行為**：未填補成功則靜默跳過 — 玩家可能誤以為卡片無效。
*   **精英 / Boss 預設限制**：是為了避免「增援卡 + 精英」連鎖召喚一堆精英 — 此設計選擇有 `// TODO: 應該移到 Condition` 註解，未來可能重構。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ReinforcementSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_ReinforcementSetting` → 「增援」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_DefaultUnit` | `DefaultUnit` | `RCG_UnitGenData` | — | 精英 / Boss 戰使用 |

### A.3 重要 Method 摘要
*   **`LocalizeKey` (private)** → 寫死回傳 `"ReinforcementDes_Monster"`（注釋：目前僅敵人增援）。
*   **`AddAction`** (async)：
    1. 從 `MonsterSet.GetUnitSpawnDatas(EnemyType)` 隨機抽 → `aTargetUnit`。
    2. 若敵人類型 ≠ Normal → 改用 `m_DefaultUnit.GetData()`。
    3. 嘗試從 `iData.TargetPositions` 取空位；否則 `RCG_BattleField` 自動分配（檔案 80 行外的後續邏輯）。

### A.4 與其他系統的互動
*   **`RCG_BattleField.Ins.MonsterSet.GetUnitSpawnDatas`**：候選怪物清單來源。
*   **`RCG_BattleManager.Ins.EnemyType`** / **`RCG_EnemyTypeTagGenData.s_EnemyType_Normal`**：戰鬥類型判斷。
*   **`UCL_Random.Instance.Next`**：候選選擇的隨機來源。

### A.5 已知議題
*   `// TODO: 目前只有普通戰鬥獲得增援 (應該移到Condition)` — 應該抽成條件判斷，而非寫死於增援邏輯內。
*   `bool aIsMonster = true` 寫死，無法做友軍增援。
