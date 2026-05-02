---
title: 獲得道具 說明
description: 把指定的物品加入玩家背包；可選擇是否為臨時物品（戰鬥後自動移除）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 獲得道具

> 程式類別名稱：`RCG_AddItemSetting`

## 用途
把一個或多個指定物品加入玩家背包，常見於「戰鬥獎勵卡」「拾取道具事件」「臨時 buff 道具發放」。

## 編輯器中的樣貌
```
▼ ✓ [獲得道具(AddItem)] (物品縮圖列表)
    Items         ▶ (物品清單；展開可加入多個)
    IsTmpItem     [✓]
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Items**（道具，未本地化於此） | 是 | 要加入的物品清單；每項是 `RCG_ItemGenData`（物品模板）。 |
| **IsTmpItem** (是否為臨時物品)（未本地化於此） | — | 預設打勾。打勾 = 戰鬥結束後自動移除（不影響背包永久狀態）；不打勾 = 永久持有。 |

## 行為說明
*   觸發時依序把每個物品加入玩家背包，並彈出「獲得道具」面板秀給玩家看。
*   若 IsTmpItem 打勾，物品會被標記為臨時，戰鬥結束自動清掉。
*   描述格式為「**獲得 {物品 1, 物品 2, ...}**」，臨時物品下方會多一行 `[臨時道具]` 標記。

## 注意事項
*   **臨時物品的存檔風險**：戰鬥中存檔重新載入，臨時物品的清除規則由背包系統處理；若物品本身有持續效果，務必測試載入流程。
*   **空清單**：技術上合法，但這張卡將什麼也不會給予 — 屬於資料錯誤。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_AddItemSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_AddItemSetting` → 「獲得道具」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_Items` | `Items` (`Items` key 在通用對照中為「道具」) | `List<RCG_ItemGenData>` | `Items` | 物品模板清單 |
| `m_IsTmpItem` | `IsTmpItem` | `bool` | `IsTmpItem` (=「是否為臨時物品?」) | 預設 `true` |

### A.3 重要 Method 摘要
*   **`Infos`**：聚合每個 `m_Items[i].GetData()` 的資訊；若 `m_IsTmpItem`，再追加 `CardInfoData.TmpItemInfo`。
*   **`GetDescriptionFormat`**：以 `, ` 串接物品名，後接 `[TmpItem]`（若臨時），最終套 i18n key `AcquireItem`。
*   **`AddAction`**：包成 `AddAsyncActionTrigger`，逐個 `m_Items[i].GetData().AddItem()`，設置 `m_TmpItem` 標記，彈出 `RCG_AquireItemPanel`。

### A.4 與其他系統的互動
*   **`RCG_ItemGenData` / `RCG_Item`**：物品模板與實例。
*   **`UI.RCG_AquireItemPanel`**：獲得物品面板 UI。
*   **`CardInfoData.TmpItemInfo`**：靜態的「臨時物品」資訊區塊。
