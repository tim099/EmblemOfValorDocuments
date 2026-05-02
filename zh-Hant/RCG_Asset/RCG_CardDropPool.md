---
title: 卡牌掉落池 (RCG_CardDropPool) 說明
description: 定義一組「會掉哪些卡、各自的權重」的資料；獎勵畫面、商店、強化分支底層都靠它抽卡
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 卡牌掉落池

> 程式類別名稱：`RCG_CardDropPool`

## 用途

定義「**這個池子會掉哪些卡牌、各張的權重各是多少**」。戰鬥獎勵、商店補貨、商人特賣、活動掉落都從某個 `RCG_CardDropPool` 隨機抽卡。同樣的池子可被多個來源引用，改一次到處生效。

繼承自 `RCG_Asset<RCG_CardDropPool>`，實作介面：`UCL.Core.UCLI_ShortName`。

## 編輯器中的樣貌

```
RCG_CardDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ 顯示名稱
        Name(多國語言)
    ▼ DropPool / MixDropPools / FilterDropData  ← 視 DropType 顯示對應區塊
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **DropType** | 是 | `DropPool`（直接列卡 + 權重）/ `MixPool`（混合別的池子）/ `FilterDrop`（用標籤條件篩） |
| **Name** | 否 | 顯示名稱（多語系）。空白時 `GetShortName()` fallback 到第一個 drop 的名稱、再 fallback 到 ID |
| **DropPool** | DropType=DropPool | 卡片清單 + 各自權重（`RCG_CommonDropSetting<RCG_CardGenData>`） |
| **MixDropPools** | DropType=MixPool | 引用其他掉落池並各自指定權重；最終結果是合併後的加權平均 |
| **FilterDropData** | DropType=FilterDrop | 用 `CardDropFilter` 條件式（標籤、稀有度、等等）動態篩出符合的卡，無條件 = 全部卡 |

## 行為說明

### 三種模式
*   **DropPool（直接列）**：手動列出每張卡 + 權重；最直觀，適合精選池。
*   **MixPool（混合）**：把多個既有池子按權重合併，避免重複維護同一份名單。權重會層層加權（外層 weight × 內層 weight）。最多遞迴 10 層，避免循環。
*   **FilterDrop（標籤篩）**：用條件式從**全卡庫**篩出符合的卡，所有命中卡權重均等。條件支援 AND / OR / NOT 巢狀。

### 隱性篩選（runtime 戰鬥中自動套用）
即使你寫了權重 0.5 的卡，runtime 也可能把它從池子裡剔除：
*   **未解鎖** (`UnlockData.CheckCardLocked`)：跳過。
*   **隊伍無人能用** (`CheckRequireSkill`)：卡有指定專精且隊伍中無任何角色擁有對應專精，跳過。
*   **主選單與非戰鬥環境**：不做技能檢查（避免 UI 預覽空轉）。
被剔除後剩餘卡片會**重新標準化權重**（總和回到 1）。

### 預覽
編輯器右側按 `ShowDetail` 可即時看到當前條件下的最終掉落率清單。

## 注意事項

*   **DropPool 模式下** 反序列化時會**自動移除不存在的卡 ID**（例如 ID 改名 / 被刪）；MixPool 模式類似（移除不存在的子池）。看 console 有 `Remove Invalid DropPool` 的 LogError 表示這個池子曾經引用了壞 ID。
*   **FilterDrop 條件全空** = 全卡庫均等掉落。除非你真的想要這個效果，否則記得加條件。
*   **MixPool 的循環**會被 `iLayer > 10` 截斷成空清單；發現某個池子掉空時先檢查混合鏈是否成環。
*   **RCG_CardFilter cache**：FilterDrop 的查詢走 `RCG_CardFilter.GetCards`，有快取；新增卡牌後若預覽不對，可從 `RCG_CardData.OnLoadModule` 觸發 cache 清除。

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊

*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_CardDropPool.cs`
*   **繼承自**：`RCG_Asset<RCG_CardDropPool>`
*   **實作介面**：`UCL.Core.UCLI_ShortName`
*   **AssetGroup**：`EditDropSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_Name` | 顯示名稱 | `RCG_LocalizeData` | `Name` | `[SerializeField] protected` |
| `m_DropPool` | 掉落池 | `RCG_CommonDropSetting<RCG_CardGenData>` | — | `Conditional(m_DropType == DropPool)` |
| `m_MixDropPools` | 混合池清單 | `List<MixDropPoolData>` | — | `Conditional(m_DropType == MixPool)` |
| `m_FilterDropData` | 條件式 | `FilterDropData`（內部巢狀） | — | `Conditional(m_DropType == FilterDrop)` |
| `m_DropType` | DropType | `EDropType` enum | `DropType` | 預設 `DropPool` |

### A.3 重要 Method 摘要

*   **`GetDropCards(int, bool)`** → 主要外部入口，回傳 `iDropCount` 張隨機卡。
*   **`GetDropCardsWithFilterFunc(int, Func)`** → 外部自訂篩選版（例如指定稀有度）。
*   **`GetDropRate(bool, CheckDropConditionData, int)`** → 取最終權重表（總和=1）；`iIsFilterSkill = true` 時自動套用 `CheckRequireSkill`。
*   **`CheckRequireSkill(RCG_CardGenData)`** (static) → 三層檢查：runtime 中 → 主選單跳過 → 解鎖檢查 → 隊伍專精符合性。
*   **`DeserializeFromJson`** → 載入時清理失效 ID（DropPool / MixPool 兩種模式各自處理）。
*   **`Preview`** → 編輯器內預覽 UI，可展開看完整掉落率表。

### A.4 與其他系統的互動

*   **`RCG_CommonDropSetting<T>`** — 通用掉落容器；`m_DropPool` 直接用它。
*   **`RCG_CardGenData`** — 卡片 ID 包裝；掉落結果是 `List<RCG_CardGenData>`。
*   **`RCG_CardDropPoolGenData`** — Asset Entry 包裝；其他 Asset 引用此池子時用這個型別欄位。
*   **`RCG_CardFilter`** — `FilterDrop` 模式下查詢卡庫的入口。
*   **`RCG_DataService.Ins.m_UnlockData`** — runtime 鎖定查詢。
*   **`RCG_CharacterDataService.Ins.GetAllSkillTags()`** — 隊伍專精查詢。

### A.5 已知議題

*   `MixPool` 的 weight 計算對權重總和未做標準化（直接累加 `aWeight * aDrop.m_Weight`）；最終透過 `GetDropRate()` 標準化為總和=1。
*   遞迴上限 10 層（`iLayer > 10` 直接回空），這個閾值是 magic number。
