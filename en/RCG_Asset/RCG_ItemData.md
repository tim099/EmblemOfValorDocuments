---
title: 道具資料 (RCG_ItemData) 說明
description: 玩家可使用的消耗 / 任務道具模板（藥水、捲軸、鑰匙等）：稀有度、效果、使用次數、解鎖
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-en
---

> [!WARNING]
> Translation pending — this file needs an English translation.
The original zh-Hant content is included below for reference.


# 道具資料

> 程式類別名稱：`RCG_ItemData`

## 用途

**玩家可持有 / 使用的道具模板**。藥水、捲軸、食材、任務鑰匙、回復道具⋯ 全都是這個類別的實例。每個道具有自己的圖示、效果、稀有度、使用次數限制、解鎖條件。

繼承自 `RCG_Asset<RCG_ItemData>`，實作介面：`RCGI_Item`（可加入背包）/ `RCGI_Unloackable`（可解鎖）。

## 編輯器中的樣貌

```
RCG_ItemData: <ID>
    Data (m_Data)              ← 主要設定（巢狀類）
    Effects (m_ItemEffects)    ← 道具效果（OnPlay 觸發）
    Preview                    ← 即時呈現
```

## 主要欄位（Data 內）

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Name** | 是 | 道具名（多語系） |
| **Description** | 否 | 風味描述（多語系，附加在自動描述後） |
| **TargetType** | 是 | 使用時的選目標範圍（None / Friend / Enemy 等） |
| **Price** | 是 | 商人售價；預設 20，可用 `Auto Price` 按鈕重算（基礎 3 × 稀有度價值） |
| **Icon** | 是 | 道具圖示 |
| **Rarity** | 是 | 稀有度標籤 |
| **ItemType** | 是 | `Normal`（普通）或 `QuestItem`（任務道具，劇情用） |
| **ItemUseType** | 是 | `Consume`（用完即消失）/ `OncePerBattle`（每戰一次）/ `OncePerTurn`（每回合一次）/ `Infinite`（無限次） |
| **UseTimes** | 視 ItemUseType | 可使用次數（Consume / OncePer* 模式有效） |
| **Unlock** | 否 | 解鎖條件 |
| **HideInCodex** | — | 圖鑑隱藏（測試道具） |

外層另有 **m_ItemEffects** = `List<RCG_CommonEffect>`，描述使用時觸發的效果（OnPlay 是主要 trigger）。

## 行為說明

### 使用流程
1. 玩家從背包選道具 → 進入 `RCG_ItemInfoPanel`。
2. `CheckUsable(data)`：對每個 effect 跑 `CheckPlayable`，全部 true 才能使用。
3. `TriggerEffect(data)`：取所有 `OnPlay` effect 並依序觸發（含 try-catch 防爆）。
4. 依 `ItemUseType` 決定是否消失 / 鎖定到下回合 / 鎖定到下場戰鬥。

### 描述生成
*   自動由 `Effects` 串接（每個 effect 一行）。
*   外加：非 Normal 的 ItemType 標記、非 Consume 的 UseType 標記。
*   `UseTimes > 1` 時加上「剩餘 N/M 次」說明（runtime 才會顯示剩餘次數）。
*   `m_Description` 有設定時 → 在自動描述下面額外加風味文字。

### Auto Price（編輯器工具）
編輯器 `RCG_ItemDataEditorPage` 上方有 `Auto Price` 按鈕：對所有 ItemData 套用 `Price = 3 × Rarity.m_Value`。**會覆蓋手動設的價格**。

## 注意事項

*   **ItemUseType = Infinite 時 UseTime 回 0**：runtime 不消耗、不鎖定。
*   **`m_Description` 與自動描述會疊加**：手動寫的部分顯示在效果之後。
*   **QuestItem 不會被一般使用流程消耗**：通常綁定劇情事件觸發。
*   **Auto Price 會洗掉手動價**——按之前要確認。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_ItemData.cs`
*   **繼承自**：`RCG_Asset<RCG_ItemData>`
*   **實作介面**：`RCGI_Item` / `RCGI_Unloackable`
*   **AssetGroup**：`EditItems`

### A.2 欄位對照（外層）

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_Data` | Data | `ItemData`（巢狀） | `[SerializeField] protected` |
| `m_ItemEffects` | Effects | `List<RCG_CommonEffect>` | |

`ItemData` 巢狀內含 `m_Name` / `m_Description` / `m_TargetType` / `m_Price` / `m_Icon` / `m_Rarity` / `m_ItemType` / `m_ItemUseType` / `m_UseTimes` / `m_Unlock` / `m_HideInCodex`。

### A.3 重要 Method 摘要

*   **`AddItem()`** — 玩家獲得：`RCG_Item.Create(ID)` → `m_ItemsData.AddItem`。
*   **`CheckUsable(TriggerEffectData)`** — 對所有 enabled effects 跑 `CheckPlayable` 取 AND。
*   **`TriggerEffect(TriggerEffectData)`** — 取 `OnPlay` effects 並逐一觸發（try-catch）。
*   **`Description` / `FullDescription` / `GetDescription`** — 三層描述生成。
*   **`Effects` (property)** — `m_ItemEffects.GetEnableEffects()`。
*   **`Infos` (property)** — 聚合 effect infos + ItemType 描述。
*   **`CreateSelectAssetPage`** — `RCG_ItemDataEditorPage.Create()`。

### A.4 與其他系統的互動

*   **`RCG_Item`** — runtime 道具實例。
*   **`RCG_DataService.Ins.m_ItemsData`** — 玩家背包儲存。
*   **`RCG_ItemDataEditorPage`** — 編輯主畫面（含 Auto Price 工具）。
*   **`RCG_ItemInfoPanel`** — 詳細資訊 UI。

### A.5 已知議題

*   `m_UseTimes` 的「暫時未完成 先隱藏」註解暗示可變使用次數的功能還沒完整實作。
*   `SerializeToJson` / `DeserializeFromJson` override 已被註解；曾規劃過自動價格遷移邏輯。