---
title: 裝備資料 (RCG_EquipmentData) 說明
description: 角色穿戴的裝備模板（武器、防具、飾品、遺物）：類型、稀有度、效果、強化分支、不重複掉落
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-ja
---

> [!WARNING]
> 翻訳待機中 — このファイルは日本語翻訳が必要です。
参考用に zh-Hant 原文を以下に掲載しています。


# 裝備資料

> 程式類別名稱：`RCG_EquipmentData`

## 用途

**角色身上穿戴的裝備模板**。武器、防具、飾品、遺物（特殊加成物品）皆是。每個裝備有自己的稀有度、類型、效果（戰鬥中各 trigger 觸發）、強化分支、是否能重複掉落。

繼承自 `RCG_Asset<RCG_EquipmentData>`，實作介面：`RCGI_Item` / `RCGI_Unloackable`。

## 編輯器中的樣貌

```
RCG_EquipmentData: <ID>
    Name / Description / Rarity / Tags / EquipmentType / Icon / Price
    Effects                    ← 戰鬥中觸發的效果
    InitCounters               ← 起始計數器
    SkillTags                  ← 職業專屬限制
    Unlock                     ← 解鎖條件
    UpgradeBranch              ← 可強化成哪些裝備
    EnhenceLevel / HideInCodex / CanDropRepeatedly
    Preview
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Name** | 是 | 裝備名（多語系） |
| **Description** | 否 | 風味描述 |
| **Rarity** | 是 | 稀有度 |
| **Tags** | 否 | 一般標籤（DropPool 用） |
| **EquipmentType** | 是 | `Weapon` / `Armor` / `Accessory` / `Relic`（**遺物：不佔裝備格、強制裝在主角身上**） |
| **Icon** | 是 | 裝備圖 |
| **Price** | 是 | 商店售價；預設 500，可用 `Auto Price` 按鈕重算（基礎 10 × 稀有度價值） |
| **Effects** | 否 | 戰鬥中各 trigger 觸發的效果 |
| **InitCounters** | 否 | 給 effect 中計數器類效果用的初值 |
| **SkillTags** | 否 | 職業專屬限制（隊伍中要有對應職業才掉落；OR 關係：任一符合即可） |
| **Unlock** | 否 | 解鎖條件 |
| **UpgradeBranch** | 否 | 可強化成哪些裝備（透過遊戲內強化機制）；空 = 不能強化 |
| **EnhenceLevel** | — | 強化等級 > 0 表示「強化版」，**不顯示在圖鑑** |
| **HideInCodex** | — | 圖鑑隱藏（測試 / 強化版會自動隱藏） |
| **CanDropRepeatedly** | — | 是否能重複掉落；遺物 (`Relic`) 通常設 false |

## 行為說明

### 觸發
`OnTriggerEffect(data, triggerOn)` → 從 `m_Effects` 取對應 trigger 的 effects 並依序觸發。`TriggerOnUnitState(triggerOn)` 是 quick check。

### 圖鑑隱藏判斷
`HideInCodex` (property) = `m_HideInCodex || m_EnhenceLevel > 0`：強化版自動隱藏，避免圖鑑出現「+1 / +2」等同名重複。

### 描述
`GetFullDescription` = `Effects 描述 + m_Description（風味文字）`。

### Auto Price
編輯器頁面有 `Auto Price` 按鈕：對所有裝備套用 `Price = 10 × Rarity.m_Value`。

### 排序
`RCG_EquimpmentComparer` 按「裝備類型（Weapon=10, Armor=20, Accessory=30）」→「價格降序」排序。

## 注意事項

*   **遺物 (`Relic`) 規則**：不佔裝備格、強制在主角身上、`CanDropRepeatedly` 通常設 false（DropPool 會檢查身上是否已有）。
*   **`SkillTags` 與卡牌專精的差異**：裝備是 OR 關係（任一隊員有就能掉），卡牌的 `RequireSkills` 才是 AND。
*   **強化版 (`EnhenceLevel > 0`) 自動隱藏圖鑑**：圖鑑只會看到原版。
*   **`NullEquipmentID = "NullEquipment"`** 是系統保留 ID，代表「無裝備」；不要拿來命名一般裝備。
*   **Auto Price 會洗手動價**。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_EquipmentData.cs`
*   **繼承自**：`RCG_Asset<RCG_EquipmentData>`
*   **實作介面**：`RCGI_Item` / `RCGI_Unloackable`
*   **AssetGroup**：`EditItems`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Description` | Description | `RCG_LocalizeData` | |
| `m_Rarity` | Rarity | `RCG_RarityTagGenData` | |
| `m_Tags` | Tags | `List<RCG_ItemTagGenData>` | |
| `m_EquipmentType` | EquipmentType | `EquipmentType` enum | `Weapon` 預設 |
| `m_Icon` | Icon | `RCG_SpriteData` | |
| `m_Price` | Price | `int` | 預設 500 |
| `m_Effects` | Effects | `List<RCG_CommonEffect>` | |
| `m_InitCounters` | InitCounters | `List<int>` | |
| `m_SkillTags` | SkillTags | `List<RCG_SkillTagGenData>` | |
| `m_Unlock` | Unlock | `RCG_UnlockEntry` | |
| `m_UpgradeBranch` | UpgradeBranch | `List<RCG_EquipmentGenData>` | |
| `m_EnhenceLevel` | EnhenceLevel | `int` | 預設 0 |
| `m_HideInCodex` | HideInCodex | `bool` | |
| `m_CanDropRepeatedly` | CanDropRepeatedly | `bool` | 預設 `true` |

### A.3 重要 Method 摘要

*   **`AddItem()`** — 玩家獲得：`m_EquipmentsData.AddEquipment(new RCG_Equipment(ID))`。
*   **`OnTriggerEffect(data, triggerOn)`** — 戰鬥觸發。
*   **`HideInCodex` (property)** — `m_HideInCodex || m_EnhenceLevel > 0`。
*   **`GetDescription` / `GetFullDescription`** — 描述生成。
*   **`CreateSelectAssetPage`** — `RCG_EquipmentDataEditorPage.Create()`。

### A.4 與其他系統的互動

*   **`RCG_Equipment`** — runtime 裝備實例。
*   **`RCG_DataService.Ins.m_EquipmentsData`** — 玩家裝備儲存。
*   **`RCG_EquipmentDropPool`** — 掉落池（含 `m_CanDropRepeatedly` 檢查）。
*   **`RCG_EquipmentInfoPanel`** — 詳細資訊 UI。
*   **`RCG_EquimpmentComparer`** — 排序 helper。

### A.5 已知議題

*   `m_CanDropRepeatedly` 的 TODO 註解（"需要記錄掉落 & 排除"）暗示「不重複掉落」的紀錄機制有歷史改動；目前由 DropPool runtime 比對玩家裝備清單實現。
*   `DeserializeFromJson` 的自動價格 `m_Price = m_Rarity.GetData().m_Value.GetValue(null) * 15` 已註解，被 Auto Price 工具取代。