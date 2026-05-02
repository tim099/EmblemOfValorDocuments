---
title: 怪物移動 說明
description: 讓怪物在戰場上移動位置（前後排切換），由 AI 行為使用
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 怪物移動

> 程式類別名稱：`RCG_MonsterMoveSetting`

## 用途
**讓怪物在戰場上移動位置** — 通常是前後排切換。由怪物 AI 行為使用，常見用途：
*   敵人主動退到後排避戰
*   特殊單位的位移技能

> [!IMPORTANT]
> 此設定**僅在編輯怪物資料時**才會出現在下拉選單中。一般卡牌不會看到此選項。

## 主要欄位
（無欄位 — 純行為設定）

## 行為說明
*   呼叫 `CreateAction.MoveAction(iData.User)` 觸發單位移動 Action。
*   實際移動邏輯由 `MoveAction` 的內部規則決定（一般是前後排切換）。
*   描述為 i18n key `UnitAction_MoveActionDes`，精簡描述顯示移動 sprite。

## 注意事項
*   **與「移動」設定的差別**：「移動」(`RCG_MoveSetting`) 是**通用**移動設定，可指定方向與目標；本設定是**怪物 AI 專用**，行為固定，無欄位可調。
*   **無欄位 = 行為固定**：移動行為由 `CreateAction.MoveAction` 決定；想客製請改用「**移動**」。
*   **無 User 跳過**：`iData.User == null` 時什麼也不做。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_MonsterBattleSettings/RCG_MonsterMoveSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **無 i18n 類別名 key**：編輯器顯示 stripped name `MonsterMove`
*   **限定可選範圍**：`RCG_BattleSetting.s_MonsterDataTypes` 中註冊（編輯怪物資料時才出現於選單）。

### A.2 欄位對照
（無自有欄位）

### A.3 重要 Method 摘要
*   **`AddAction`**：`iData.User != null` 時 `iData.AddAction(CreateAction.MoveAction(iData.User), iAddActionMode)`。
*   **`GetDescription`** → i18n key `UnitAction_MoveActionDes`。
*   **`GetDescriptionShort`** → 移動 sprite (`EffectIcon.Move`)。

### A.4 與其他系統的互動
*   **`CreateAction.MoveAction(unit)`**：實際的移動 Action 建構工具（與通用「移動」共用？或專屬怪物 AI）。
