---
title: 卡牌強化掉落池 (RCG_CardEnhenceDropPool) 說明
description: 定義「強化卡牌時可抽到哪些強化分支」的資料；強化選單背後的隨機池
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-ja
---

> [!WARNING]
> 翻訳待機中 — このファイルは日本語翻訳が必要です。
参考用に zh-Hant 原文を以下に掲載しています。


# 卡牌強化掉落池

> 程式類別名稱：`RCG_CardEnhenceDropPool`

## 用途

定義「**強化某張卡時，會抽到哪些強化分支 (`RCG_CardEnhenceData`) 作為候選**」。每張卡的 `m_EnhencePool` 欄位會引用一個 `RCG_CardEnhenceDropPool`；強化時系統從此池抽 N 個分支讓玩家挑。

繼承自 `RCG_Asset<RCG_CardEnhenceDropPool>`，實作介面：`UCL.Core.UCLI_ShortName`。

## 編輯器中的樣貌

```
RCG_CardEnhenceDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ DropPool / MixDropPools / FilterDropData
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **DropType** | 是 | `DropPool` / `MixPool` / `FilterDrop` |
| **DropPool** | DropType=DropPool | 強化分支清單 + 權重 |
| **MixDropPools** | DropType=MixPool | 混合其他池子 |
| **FilterDropData** | DropType=FilterDrop | 用內部 `DropFilter`（Operator）篩 |

## 行為說明

與其他 Drop Pool 同骨架。內部 `DropFilter` 目前**只支援 Operator 類型**（Tag 已被註解掉），實務上 FilterDrop 模式較少使用，多數採 DropPool 直接列。

抽取時還會被卡牌端 `BannedEnhence` 列表 + `RCG_CardEnhenceCondition` 二次過濾（見 `RCG_CardData.GetEnhenceBranchs`）。

## 注意事項

*   **enum 名稱叫 `DropType`**（不是 `EDropType`）。
*   **DropFilter Tag 已被註解掉**：FilterDrop 模式現階段功能受限；建議用 DropPool 模式直接列。
*   **強化條件與 BannedEnhence** 的二次篩選在卡牌端進行，本池只負責「初次抽出候選」。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_CardEnhenceDropPool.cs`
*   **繼承自**：`RCG_Asset<RCG_CardEnhenceDropPool>`
*   **AssetGroup**：`EditDropSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_DropPool` | 掉落池 | `RCG_CommonDropSetting<RCG_CardEnhenceGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_FilterDropData` | 條件式 | `FilterDropData` | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `DropType` enum | 預設 `DropPool` |

### A.3 與其他系統的互動

*   **`RCG_CardEnhenceData`** — 掉落目標（強化分支定義）。
*   **`RCG_CardEnhenceGenData`** / **`RCG_CardEnhenceDropPoolGenData`** — Asset Entry 包裝。
*   **`RCG_CardData.m_EnhencePool` / `RCG_CardData.GetEnhenceBranchs`** — 強化流程的呼叫端，會於此池抽出候選後再做二次過濾。