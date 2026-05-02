---
title: 變身 說明
description: 把目標單位變身為另一種單位；支援預設變身、轉生、重生三種模式
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 變身

> 程式類別名稱：`RCG_UnitTransformSetting`

## 用途
**把單位變身為另一種單位**。常見用途：
*   Boss 切換型態（保留血量改變外觀）
*   「轉生」「重生」類復活機制
*   特殊變身卡牌

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **目標** (`Target`) | 是 | 變身目標選擇器。 |
| **UnitData** | 是 | 要變身為哪種單位（`RCG_UnitGenData`）。 |
| **TransformSetting** | 是 | 變身細節設定（嵌套）。 |

### TransformSetting 內欄位
| 欄位 | 預設 | 說明 |
|---|---|---|
| **TransformType** | `Default` | 變身類型：<br>• **Default** — 一般變身。**保留 HP 與狀態**，只改變行為與模型（**Boss 換型用**）。<br>• **Reincarnation** — 轉生。**只保留狀態**；HP / 上限 / 數值全變新單位的；觸發初始行動。<br>• **Rebirth** — 重生。**不保留狀態**；HP / 上限 / 數值全變新單位的；觸發初始行動。 |
| **ClearAllStatus** | `false` | 是否清空所有狀態（與 TransformType 的「保留狀態」邏輯獨立）。 |

## 行為說明
*   描述格式：`UnitTransformDes_{TransformType}`，套入目標與單位名稱。
*   執行時依 TransformType 決定保留哪些屬性，並進行視覺切換（換 Spine 模型 / AI / 行為）。

## 注意事項
*   **Default vs Reincarnation 區別**：兩者都會換模型，但 Default 保留所有當前數值（適合 Boss 切型）；Reincarnation 重置 HP 但保留狀態（適合「轉生」）。
*   **觸發初始行動**：Reincarnation / Rebirth 會跑新單位的 `OnBattleStart`（含進場狀態 / 進場攻擊等）。
*   **ClearAllStatus 與 TransformType 衝突**：Default + ClearAllStatus 等於「保留 HP 但清狀態」 — 罕見組合，請慎用。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_UnitTransformSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_UnitTransformSetting` → 「變身」
*   **同檔頂層類**：`TransformSetting`（含 `TransformType` enum 與 `m_ClearAllStatus`）

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_Target` | 目標 | `RCG_SelectTargetData` | `Target` | |
| `m_UnitData` | `UnitData` (=「戰鬥角色」) | `RCG_UnitGenData` | `UnitData` | |
| `m_TransformSetting` | `TransformSetting` | `TransformSetting` | — | 含 `TransformType` 與 `ClearAllStatus` |

### A.3 重要 Method 摘要
*   **`AddAction`**（檔案 80 行外）：依 `TransformType` 切換實際變身邏輯。
*   **`GetDescriptionFormat`** → `UnitTransformDes_{TransformType}` i18n key，套 target / unit。
*   **`Info`** → `new CardInfoData(m_UnitData.GetData())`。

### A.4 與其他系統的互動
*   **`RCG_Unit` 變身入口**（具體 method 名見後續實作）。
*   **`RCG_UnitGenData`**：變身目標單位模板。
*   **`OnBattleStart`**：Reincarnation / Rebirth 的初始行動觸發點。
