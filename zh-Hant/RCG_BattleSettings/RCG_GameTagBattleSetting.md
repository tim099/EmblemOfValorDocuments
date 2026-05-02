---
title: 遊戲標籤 說明
description: 觸發遊戲層級的標籤效果（成就、玩家事件等）；薄包裝層
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 遊戲標籤

> 程式類別名稱：`RCG_GameTagBattleSetting`

## 用途
**在戰鬥中觸發遊戲層級的標籤事件** — 是 `RCG_GameTagSetting` 的薄包裝層，將其行為轉接到戰鬥動作佇列。常見用途：
*   觸發成就條件（例如「擊敗第一個 Boss」）
*   標記玩家戰鬥行為紀錄

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **GameTagSetting** | 是 | 嵌套的 `RCG_GameTagSetting` — 實際的遊戲標籤設定。 |

## 行為說明
*   觸發時呼叫 `m_GameTagSetting.Trigger(iData)`。
*   描述、卡片資訊全部代理到 `m_GameTagSetting`。

## 注意事項
*   **此設定本身只是包裝**：實際邏輯都在 `RCG_GameTagSetting` — 配置欄位請查該類別說明。
*   **不影響戰鬥數值**：純粹是事件回報 / 紀錄類效果。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_GameTagBattleSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_GameTagBattleSetting` → 「遊戲標籤」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_GameTagSetting` | `GameTagSetting` | `RCG_GameTagSetting` | — | 嵌套；包裝層的唯一欄位 |

### A.3 重要 Method 摘要
*   **`AddAction`**：`m_GameTagSetting.Trigger(iData)`。
*   **`Info / GetDescriptionParams / GetDescriptionFormat`**：全部代理到 `m_GameTagSetting`。

### A.4 與其他系統的互動
*   **`RCG_GameTagSetting`**：實際邏輯所在；包裝層只負責 Battle Setting 介面對接。
