---
title: 裝備掉落池 (RCG_EquipmentDropPool) 說明
description: 定義一組「會掉哪些裝備、權重」的資料；商店、戰鬥獎勵、寶箱靠它抽裝備
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-ja
---

> [!WARNING]
> 翻訳待機中 — このファイルは日本語翻訳が必要です。
参考用に zh-Hant 原文を以下に掲載しています。


# 裝備掉落池

> 程式類別名稱：`RCG_EquipmentDropPool`

## 用途

定義「**這個池子會掉哪些裝備、各自權重**」。戰鬥勝利後、商店、寶箱、Boss 戰利品從這裡抽裝備（武器、護甲、飾品、遺物⋯）。與卡牌/道具池骨架相同，差異是「**遺物類裝備不能重複掉落**」由此處 runtime 篩除。

繼承自 `RCG_Asset<RCG_EquipmentDropPool>`，實作介面：`UCL.Core.UCLI_ShortName`。

## 編輯器中的樣貌

```
RCG_EquipmentDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    Name(多國語言)
    ▼ DropPool / MixDropPools / FilterDropData  ← 視 DropType 顯示
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **DropType** | 是 | `DropPool` / `MixPool` / `FilterDrop` |
| **Name** | 否 | 顯示名稱（多語系） |
| **DropPool** | DropType=DropPool | 裝備清單 + 權重 |
| **MixDropPools** | DropType=MixPool | 引用其他池子並指定權重 |
| **FilterDropData** | DropType=FilterDrop | 用內部 `DropFilter`（SkillTag / RarityTag / Tag / EquipmentType / Operator）篩 |

## 行為說明

### 三種模式
與其他 Drop Pool 同：DropPool（直接列）/ MixPool（合併）/ FilterDrop（條件篩）。FilterDrop 比卡牌池多一層「**裝備類型 (EquipmentType)**」可篩武器/防具/飾品。

### 隱性篩選（runtime）
*   **未解鎖** (`UnlockData.m_LockedEquipments.CheckLocked`)：跳過。
*   **遺物已擁有** (`m_CanDropRepeatedly = false` 且玩家身上已有同 ID 裝備)：跳過。**這就是遺物只會出現一次的機制**。
*   **隊伍無人符合專精**（裝備有指定 `m_SkillTags` 但隊伍無人持有）：跳過。
被剔除後剩餘裝備**重新標準化權重**。

### 預覽
編輯器按 `ShowDetail` 即時看最終掉落率。

## 注意事項

*   **遺物的不重複掉落**完全靠 runtime 比對 `RCG_DataService.Ins.m_EquipmentsData.m_Equipments`；單機預覽看不到此邏輯，要實機測試才會生效。
*   **裝備有專精限制**時池子可能在某些隊伍組合下大幅縮水甚至空池。設計時要考慮所有可能隊伍。
*   **DropPool 模式下** 反序列化時會自動移除不存在的裝備 ID。
*   **`CheckRequireSkill` 在這個類別永遠回 true**，過濾邏輯都集中在 `GetDropRate` 內的 closure 裡。

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊

*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_EquipmentDropPool.cs`
*   **繼承自**：`RCG_Asset<RCG_EquipmentDropPool>`
*   **實作介面**：`UCL.Core.UCLI_ShortName`
*   **AssetGroup**：`EditDropSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_Name` | 顯示名稱 | `RCG_LocalizeData` | `Name` | `[SerializeField] protected` |
| `m_DropPool` | 掉落池 | `RCG_CommonDropSetting<RCG_EquipmentGenData>` | — | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | — | `Conditional(MixPool)` |
| `m_FilterDropData` | 條件式 | `FilterDropData` = `FilterDropDataBase<RCG_EquipmentGenData, RCG_EquipmentData, DropFilter>` | — | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `EDropType` enum | `DropType` | 預設 `DropPool` |

### A.3 重要 Method 摘要

*   **`GetDropEquipments(int, List<RCG_SkillTagGenData>)`** → 主要入口；不指定 skills 時自動取隊伍當前 skills。
*   **`GetDropRate(List<RCG_SkillTagGenData>, ...)`** → 套用 unlock + 遺物 + 專精檢查。
*   **`CheckRequireSkill`** (static) → 永遠回 `true`（保留簽名一致性，實際過濾在 `GetDropRate` 的 closure）。
*   **`DeserializeFromJson`** → 清理失效 ID。

### A.4 與其他系統的互動

*   **`RCG_EquipmentData`** — 池子掉的目標型別。
*   **`RCG_EquipmentGenData`** / **`RCG_EquipmentDropPoolGenData`** — Asset Entry 包裝。
*   **`RCG_DataService.Ins.m_EquipmentsData.m_Equipments`** — 玩家身上裝備清單，用於遺物去重。
*   **`RCG_DataService.Ins.m_UnlockData.m_LockedEquipments`** — unlock 篩選。
*   **`RCG_CharacterDataService.Ins.GetAllSkillTags()`** — 隊伍專精查詢。

### A.5 已知議題

*   `CheckRequireSkill` 註解 `(應該不需要)` — 邏輯已搬到 `GetDropRate` 內 closure。
*   遞迴上限 10 層。