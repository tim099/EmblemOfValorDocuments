---
title: 召喚 說明
description: 召喚單位到戰場（敵人或我方），可指定召喚類型（一般 / 屍位）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 召喚

> 程式類別名稱：`RCG_SummonSetting`

## 用途
**召喚單位到戰場**。常見用途：
*   召喚我方友軍（治療師、輔助、肉盾）
*   敵人召喚雜兵
*   特殊「在死亡單位位置召喚」效果

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **UnitData** | 是 | 召喚單位的資料；嵌套 `RCG_UnitSummonData`，含召喚類型、單位、是否怪物等。 |

> [!NOTE]
> 嵌套類別 `RCG_UnitSummonData` 才是真正配置處：
> *   `m_Unit` — 要召喚的單位類型
> *   `m_IsMonster` — 預設立場（**敵方觸發時會自動翻轉**）
> *   `m_SummonType` — `Default`（找空位）/ `UnitDeadPosiiton`（在死亡單位的位置召喚）
> *   `m_Pos` — 召喚位置（前 / 後 / All）

## 行為說明
*   **立場翻轉**：如果是怪物方觸發此設定，`IsMonster` 會被反轉（怪物召喚我方 = 玩家視角的敵人）。
*   **Default 模式**：優先用 `iData.TargetPositions[0]` 指定的空位；無指定則自動找空位。
*   **UnitDeadPosiiton 模式**：在已陣亡單位的位置召喚（複生 / 屍位機制）。
*   播放 `VFX_Summon` 特效，等召喚動畫結束後觸發單位的 `OnBattleStart`（進場效果）。
*   描述格式：`SummonDes_{Monster|Player|UnitDeadPosiiton}` + 單位名。

## 注意事項
*   **無空位時靜默失敗**：找不到位置會 `Debug.LogError` 但不報錯給玩家 — 卡片可能看起來「沒效果」。
*   **召喚數上限**：受戰場最大單位數限制，已滿則無法召喚。
*   **進場效果觸發**：`OnBattleStart` 會被觸發，配備該觸發點的能力會立即生效。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_SummonSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_SummonSetting` → 「召喚」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_UnitData` | `UnitData` (=「戰鬥角色」) | `RCG_UnitSummonData` | `UnitData` | `[AlwaysExpendOnGUI]` |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：
    1. 依 `iData.Faction == UnitFaction.Enemy` 翻轉 `aIsMonster`。
    2. 依 `m_SummonType` 決定召喚位置（`Default` 找空位 / `UnitDeadPosiiton` 用陣亡單位位置）。
    3. `RCG_BattleField.SpawnUnit` 生成單位 + `VFX_Summon` 特效 + `SummonAnim` 等待 + `OnBattleStart`。
*   **`LocalizeKey` (private)**：依 SummonType 與 IsMonster 取對應 i18n key。
*   **`Info` / `GetDescriptionFormat`**：呈現召喚單位資訊與描述。

### A.4 與其他系統的互動
*   **`RCG_UnitSummonData`**：召喚資料容器（單位、類型、位置、立場）。
*   **`RCG_BattleField.SpawnUnit / TryGetEmptyPositions`**：戰場生成入口。
*   **`CommonVFX.VFX_Summon`**：召喚特效。
*   **`RCG_Unit.SummonAnim / OnBattleStart`**：召喚動畫與進場效果觸發。

### A.5 已知議題
*   舊版 `TriggerAsync` 與部分召喚邏輯保留為註解。
