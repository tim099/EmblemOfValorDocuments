---
title: 單位技能掉落池 (RCG_UnitSkillDropPool) 說明
description: 定義「升級時可抽到哪些單位技能」的資料；角色升級、技能解鎖選單背後的池子
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-en
---

> [!WARNING]
> Translation pending — this file needs an English translation.
The original zh-Hant content is included below for reference.


# 單位技能掉落池

> 程式類別名稱：`RCG_UnitSkillDropPool`

## 用途

定義「**升級或選擇技能時，會抽到哪些 `RCG_UnitSkillData`**」。例如角色升級時會給玩家三選一的技能候選，這些候選就從某個 `RCG_UnitSkillDropPool` 抽。

繼承自 `RCG_Asset<RCG_UnitSkillDropPool>`，實作介面：`UCL.Core.UCLI_ShortName`。

## 編輯器中的樣貌

```
RCG_UnitSkillDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ DropPool / MixDropPools / FilterDropData
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **DropType** | 是 | `DropPool` / `MixPool` / `FilterDrop` |
| **DropPool** | DropType=DropPool | 技能清單 + 權重 |
| **MixDropPools** | DropType=MixPool | 混合其他池子 |
| **FilterDropData** | DropType=FilterDrop | 用內部 `DropFilter` 篩 |

## 行為說明

與其他 Drop Pool 同骨架。

## 注意事項

*   enum 名稱叫 `UnitSkillDropType`（沒有 `E` 前綴也沒叫 `DropType`），與其他 Drop Pool 命名不一致；歷史遺留差異。
*   結構與 `RCG_CardDropPool` 類似，差異只在掉落目標換成 `RCG_UnitSkillData`。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_UnitSkillDropPool.cs`
*   **繼承自**：`RCG_Asset<RCG_UnitSkillDropPool>`
*   **AssetGroup**：`EditDropSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_DropPool` | 掉落池 | `RCG_CommonDropSetting<RCG_UnitSkillGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_FilterDropData` | 條件式 | `FilterDropData` | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `UnitSkillDropType` enum | 預設 `DropPool` |

### A.3 重要 Method

*   **`GetDrops(int)`** / **`GetDropsWithFilterFunc(int, Func)`** — 抽 N 個技能。
*   **`GetDropRate(...)`** — 標準化權重表。

### A.4 與其他系統的互動

*   **`RCG_UnitSkillData`** — 掉落目標。
*   **`RCG_UnitSkillGenData`** / **`RCG_UnitSkillDropPoolGenData`** — Asset Entry 包裝。