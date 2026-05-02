---
title: 戰鬥組合掉落池 (RCG_BattleSetDropPool) 說明
description: 定義「這個地圖節點會出現哪些戰鬥組合」的資料；地圖事件、章節遭遇底層的隨機池
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 戰鬥組合掉落池

> 程式類別名稱：`RCG_BattleSetDropPool`

## 用途

定義「**踩到地圖上某種戰鬥節點時，會抽到哪幾種戰鬥組合**」。例如「普通遭遇」「精英遭遇」「Boss 戰」分別是不同池子；池內列出可能的 `RCG_BattleSet`（每個 BattleSet 是一場具體的怪物配置）+ 權重。

繼承自 `RCG_Asset<RCG_BattleSetDropPool>`，實作介面：`UCL.Core.UCLI_ShortName`。

## 編輯器中的樣貌

```
RCG_BattleSetDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ DropPool / MixDropPools / FilterDropData
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **DropType** | 是 | `DropPool` / `MixPool` / `FilterDrop` |
| **DropPool** | DropType=DropPool | BattleSet 清單 + 權重 |
| **MixDropPools** | DropType=MixPool | 引用其他池子並指定權重 |
| **FilterDropData** | DropType=FilterDrop | 用內部 `DropFilter`（Tag / EnemyType / Operator）篩 |

## 行為說明

### 三種模式
與其他 Drop Pool 同。FilterDrop 條件支援「BattleSet 標籤」與「敵人類型 (EnemyType)」兩種維度。

### 隱性篩選
本類別**沒有 unlock / skill 等 runtime 篩選**——直接以權重抽。

### 預覽
按 `ShowDetail` 看最終掉落率。

## 注意事項

*   **enum 名稱叫 `EDropType` 不是 `DropType`**：類別內註解明示「取名 DropType 會導致 GoogleSheet 同步語言檔失敗，先改名」。設計時不要試圖改回。
*   **DropPool 模式下** 反序列化時會自動移除不存在的 BattleSet ID。
*   **MixPool 循環**會被截斷成空（`iLayer > 10`）；池子掉空先檢查混合鏈。
*   **無 Name 欄位**：本類別沒有額外顯示名稱，`GetShortName()` 直接取第一個 drop 的名稱。

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊

*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_BattleSetDropPool.cs`
*   **繼承自**：`RCG_Asset<RCG_BattleSetDropPool>`
*   **實作介面**：`UCL.Core.UCLI_ShortName`
*   **AssetGroup**：`EditBattleSetting`（注意：與其他 DropPool 不同，這個歸在 BattleSetting）

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_DropPool` | 掉落池 | `RCG_CommonDropSetting<RCG_BattleSetGenData>` | — | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | — | `Conditional(MixPool)` |
| `m_FilterDropData` | 條件式 | `FilterDropData` = `FilterDropDataBase<RCG_BattleSetGenData, RCG_BattleSet, DropFilter>` | — | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `EDropType` enum | `DropType` | 預設 `DropPool` |

### A.3 重要 Method 摘要

*   **`GetBattleSets(int)`** → 主要入口；無內建篩選，直接抽 N 個。
*   **`GetBattleSetsWithFilterFunc(int, Func)`** → 外部自訂篩選版。
*   **`GetDropRate(CheckDropConditionData, int)`** → 預設 filter 是 `_ => true`。

### A.4 與其他系統的互動

*   **`RCG_BattleSet`** — 池子掉的目標型別。
*   **`RCG_BattleSetGenData`** / **`RCG_BattleSetDropPoolGenData`** — Asset Entry 包裝；後者預設 ID = `"NormalBattle"`。
*   **`RCG_BattleSetTagGenData`** / **`RCG_EnemyTypeTagGenData`** — FilterDrop 用的標籤型別。

### A.5 已知議題

*   類別名 `enum EDropType`（前綴 E）是為了避開 GoogleSheet 同步衝突的歷史包袱。
*   遞迴上限 10 層。
