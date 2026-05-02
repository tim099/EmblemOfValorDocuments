---
title: 每個目標分別觸發 說明
description: 對選取的每個目標分別執行內含的組合效果（拆分 AOE 為單體迴圈）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 每個目標分別觸發

> 程式類別名稱：`RCG_ForeachTargetSetting`

## 用途
**對每個目標分別觸發**內含效果。把 AOE 拆成「**對每個敵人各做一次**」的迴圈。常見用途：
*   「對每個血量低於 10 的敵人即死」（AOE 即死必須拆開判定才正確）
*   「對全敵各回 5 點 HP（同時播放動畫）」

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Targets** | 是 | 要分別觸發的目標選擇器（通常是 AOE）。 |
| **Content** | 是 | 對每個目標執行的「**組合效果**」。 |

## 行為說明
*   解析 `Targets` 取得目標清單。
*   對每個目標：建立 child `TriggerEffectData`，將 `Targets` 設為該目標單體，再呼叫 `Content.AddAction`。
*   描述格式：「**對每個 {Targets} 分別 {Content 描述}**」（i18n key `ForeachTargetDes`）。

## 注意事項
*   **與「重複觸發」的差別**：「重複」是「**對同一目標做 N 次**」；本設定是「**對 N 個不同目標各做一次**」。
*   **預覽傷害不適用**：本設定**不覆寫 `GetPreviewDamage`**，會走父類預設值 `-1`（非攻擊牌）。要顯示傷害請在外層用其他方式表達。
*   **巢狀 Foreach**：Foreach 內再放 Foreach 是 N×M 的迴圈 — 描述會非常複雜，請避免。
*   **空目標清單**：合法，但什麼也不會發生。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ForeachTargetSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_ForeachTargetSetting` → 「每個目標分別觸發」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_Targets` | `Targets` | `RCG_SelectTargetData` | — | |
| `m_Content` | `Content` | `RCG_CombineSetting` | — | 子效果 |

### A.3 重要 Method 摘要
*   **`AddAction`**：對 `m_Targets.GetTargets(iData)` 中每個 target，`iData.CreateChildData(target.BattleName)` → `data.Targets = [target]` → `m_Content.AddAction(data, InsertInOrder)`。
*   **透傳到 `m_Content`**：`Infos` / `HasTerm` / `GetCollaborators` / `GetAtk`。
*   **`GetBattleSettings<T> / (Type)`**：自身 + 遞迴 `m_Content`。
*   **`GetFusionCandidateSettings`** → 直接代理 `m_Content` 的候選。
*   **`GetFusionBaseSetting`**：clone 自身結構，內容換為 placeholder 化的 Combine。

### A.4 與其他系統的互動
*   **`TriggerEffectData.CreateChildData / Targets`**：每目標獨立的 child data，避免子效果共用 Targets 干擾。
*   **`LocalizedStringUtils.CapitalizeString`**：暫時的首字大寫處理（與 LoopSetting 共用 hack）。
