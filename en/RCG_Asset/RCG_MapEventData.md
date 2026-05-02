---
title: 地圖事件資料 (RCG_MapEventData) 說明
description: 小地圖節點上可觸發的事件包：對話、過場、給予物品、戰鬥前奏等
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-en
---

> [!WARNING]
> Translation pending — this file needs an English translation.
The original zh-Hant content is included below for reference.


# 地圖事件資料

> 程式類別名稱：`RCG_MapEventData`

## 用途

**小地圖節點上可觸發的事件包**。例如踩到「神祕節點」會跳出對話 → 選擇 → 給予獎勵；「篝火」直接給回血選單；「商人」開店。每個 `RCG_MapEventData` 是一組 `RCG_MapEvent` 的容器，觸發時依序加入事件管理器。

繼承自 `RCG_Asset<RCG_MapEventData>`。

## 編輯器中的樣貌

```
RCG_MapEventData: <ID>
    Name                     ← 事件顯示名（多語系）
    Icon                     ← 自訂圖示（空白時用第一個 Event 的預設圖示）
    CanTriggerRepeatedly     ← 是否可重複觸發
    Events                   ← 實際事件清單（RCG_MapEvent）
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Name** | 否 | 顯示名（多語系）；空白時 fallback 到第一個 Event 的 ShortName |
| **Icon** | 否 | 圖示；空白時用第一個 Event 的 `DefaultIcon` |
| **CanTriggerRepeatedly** | — | 是否可重複觸發；false → 觸發後節點上的事件會清除 |
| **Events** | 是 | 實際的事件清單（`List<RCG_MapEvent>`） |

## 行為說明

### 觸發 (`StartEvent(node)`)
逐一 clone 每個 `RCG_MapEvent` 並加入 `RCG_MapEventManager`。Clone 而非直接用是為了讓同一個 Asset 多次觸發時，每次都是獨立 instance。

### 圖示與名稱的 fallback
*   `Icon`：`m_Icon` 空 → `m_Events[0].DefaultIcon` → null。
*   `LocalizedName`：`m_Name` 有 → 用；否則 → `m_Events[0].GetShortName()`。
*   `EventHoverTips`：`LocalizedName`（滑鼠在節點上時的提示）。

## 注意事項

*   **`CanTriggerRepeatedly = false`** 觸發後節點上的事件會清除：適合「一次性事件」如劇情關鍵點、獨特獎勵。
*   **Events 為空時**：圖示與名稱會 fallback 失敗（回 null / 空字串），UI 上看起來像沒事件。
*   **預設 ID `Start`**：通常作為起點節點專用事件。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_MapEventData.cs`
*   **繼承自**：`RCG_Asset<RCG_MapEventData>`
*   **AssetGroup**：`EditQuestSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Icon` | Icon | `RCG_SpriteData` | Addressable 預設 |
| `m_CanTriggerRepeatedly` | CanTriggerRepeatedly | `bool` | 預設 `false` |
| `m_Events` | Events | `List<RCG_MapEvent>` | |

### A.3 重要 Method

*   **`StartEvent(RCG_MapNode)`** — 主觸發入口；clone 每個 event 加入 manager。
*   **`Icon` (property)** — 含 fallback 邏輯。
*   **`LocalizedName / EventHoverTips`** — 顯示用屬性。

### A.4 與其他系統的互動

*   **`RCG_MapEvent`** — 實際事件單位（對話 / 選擇 / 獎勵 / 戰鬥前奏）。
*   **`RCG_MapEventManager`** — runtime 事件佇列管理。
*   **`RCG_MapNode`** — 觸發來源節點。
*   **`RCG_MapEventGenData`** — Asset Entry；預設 ID = `"Start"`。