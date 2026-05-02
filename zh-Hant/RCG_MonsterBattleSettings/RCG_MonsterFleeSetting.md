---
title: 怪物逃跑 說明
description: 讓怪物（敵人）從戰場逃跑，播放橫向移動動畫並從戰場移除
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 怪物逃跑

> 程式類別名稱：`RCG_MonsterFleeSetting`

## 用途
**讓怪物從戰場逃跑** — 播放單位向場外移動的動畫，然後從戰場上移除（**不算擊殺，不觸發擊殺類效果**）。常見用途：
*   特定敵人的「逃跑」行為（HP 低於門檻時逃離）
*   劇情類「敵人退場」演出

> [!IMPORTANT]
> 此設定**僅在編輯怪物資料（`RCG_UnitData` / `RCG_MonsterActionData`）時**才會出現在下拉選單中。一般卡牌不會看到此選項。

## 主要欄位
（無欄位 — 純行為設定）

## 行為說明
*   播放橫向移動動畫：敵方往右移 2000 單位、玩家方往左移 2000 單位（0.8 秒）。
*   動畫結束後呼叫 `aUser.Flee()` 從戰場移除。
*   描述格式為 i18n key `FleeActionDescription`（含完整描述）/ `FleeActionDescriptionShort`（簡短）。

## 注意事項
*   **不算擊殺**：與「即死」「死亡」不同，**不會觸發擊殺數記錄、擊殺類條件**。
*   **無欄位 = 行為固定**：移動方向與時長都寫死；想要客製請用其他設定組合。
*   **僅怪物用**：放在玩家卡上理論上會運作，但邏輯為「敵人逃跑」設計。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_MonsterBattleSettings/RCG_MonsterFleeSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **無 i18n 類別名 key**：編輯器顯示 stripped name `MonsterFlee`
*   **限定可選範圍**：`RCG_BattleSetting.s_MonsterDataTypes` 中註冊（編輯怪物資料時才出現於選單）。

### A.2 欄位對照
（無自有欄位）

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：依 `iData.User.UnitFaction` 決定 `aX = -2000` 或 `+2000`，呼叫 `UnitAnimController.MoveUnitLocal(token, new Vector3(aX, 0), 0.8f, false)`，後 `aUser.Flee()` 從戰場移除。
*   **`GetDescription / GetDescriptionShort`**：i18n key `FleeActionDescription` / `FleeActionDescriptionShort`。

### A.4 與其他系統的互動
*   **`RCG_BattleUnit.Flee`**：移除單位的入口（與「捕獲」共用）。
*   **`UnitAnimController.MoveUnitLocal`**：橫移動畫。
