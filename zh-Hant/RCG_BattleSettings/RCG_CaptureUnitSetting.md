---
title: 捕獲單位 說明
description: 捕獲目標單位（從戰鬥中移除），並將其轉化為可召喚的物品加入背包
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 捕獲單位

> 程式類別名稱：`RCG_CaptureUnitSetting`

## 用途
**捕獲**指定目標單位 — 把它從戰場上移除，**並生成一個對應的「召喚道具」**加入玩家背包。最常見的用途是「精靈球式」捕獲卡：戰鬥後可以放出該怪物協助。

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **目標** (`Target`) | 是 | 要捕獲的單位選擇器（通常選「敵方單體」）。 |
| **CaptureUnitItem** | 是 | 捕獲後生成的物品模板；預設使用 `Vivarium_Captured`（獸籠）。 |

## 行為說明
*   執行時：
    1. 對目標播放捕獲特效（`VFX_Summon`）+ 縮小動畫 0.6 秒。
    2. 目標被「逃跑」掉（`Flee()` — 從戰場移除，**不算擊殺**）。
    3. 以 `CaptureUnitItem` 為模板 clone 出新物品，名稱改為 `{物品名}[{被捕怪物名}]` 格式。
    4. 物品 ID 變為 `{原 ID}_{被捕怪物 ID}`，避免同名衝突。
    5. 物品的「使用效果」自動寫入「召喚（被捕單位）」邏輯。
    6. 物品加入背包並彈出「獲得物品」面板。
*   描述格式：「**捕獲 {目標}**」（i18n key `CaptureUnitDes`）。

## 注意事項
*   **被捕單位不算擊殺**：透過 `Flee()` 移除戰場，所以「擊殺後觸發」「擊殺數」等條件**不會記錄**這次捕獲。設計時請留意。
*   **多目標選擇器**：技術上選擇器若回傳多個目標，**只會捕獲第一個**（程式碼 `aTargets[0]`），其他目標被忽略。請選單體目標。
*   **CaptureUnitItem 不可空**：`Vivarium_Captured` 是預設值；若改成不存在的 ID 會在生成物品時例外。
*   **物品命名衝突**：兩次捕獲同一種怪物會產生相同 ID 的物品，後者會覆蓋前者 — 這是已知行為。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CaptureUnitSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_CaptureUnitSetting` → 「捕獲單位」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_Target` | 目標 | `RCG_SelectTargetData` | `Target` | 捕獲目標選擇器 |
| `m_CaptureUnitItem` | `CaptureUnitItem` | `RCG_ItemGenData` | — | 預設 `RCG_ItemGenData("Vivarium_Captured")` |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：
    1. `m_Target.GetTargets(iData)[0]` → `aTarget`。
    2. `RCG_VFXManager.CreateVFX(CommonVFX.VFX_Summon)` 並設置位置。
    3. `aTarget.UnitAnimController.ScaleUnitLocal(...)` 縮小演出。
    4. `aTarget.Flee()` 從戰場移除。
    5. clone `m_CaptureUnitItem.GetData()` 為 `RCG_ItemData`：改名（`LocalizeDic`）、改 ID（後綴 unitID）。
    6. 內嵌 `RCG_CommonEffect`（`OnPlay`）+ `RCG_SummonSetting`（`SummonType.Default`, `IsMonster=false`, `Unit=被捕怪物`）。
    7. `RCG_DataService.Ins.AddRuntimeData(aItemData, DataType.InGameRuntime)` → 註冊為 runtime 資料。
    8. 建立 `RCG_Item(aItemData, RCG_Item.ItemType.Runtime)` 並 `AddItem()` 進背包。
    9. `RCG_AquireItemPanel` 顯示獲得結果。
*   **`GetDescriptionFormat`** → i18n key `CaptureUnitDes`，僅一個 `{Target}` 參數。

### A.4 與其他系統的互動
*   **`RCG_VFXManager` / `CommonVFX.VFX_Summon`**：捕獲特效。
*   **`RCG_BattleUnit.Flee`**：把單位以「逃跑」方式移除（不算擊殺）。
*   **`RCG_DataService` / `DataType.InGameRuntime`**：runtime 物品的註冊處。
*   **`RCG_SummonSetting`**：被組裝進物品的使用效果。
*   **`RCG_AquireItemPanel`**：UI 呈現。
