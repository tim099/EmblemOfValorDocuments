---
title: 主動能力資料 (RCG_ActivePowerData) 說明
description: 角色身上的「主動能力」：可手動觸發的能力（類似道具但綁定角色而非背包）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-ja
---

> [!WARNING]
> 翻訳待機中 — このファイルは日本語翻訳が必要です。
参考用に zh-Hant 原文を以下に掲載しています。


# 主動能力資料

> 程式類別名稱：`RCG_ActivePowerData`

## 用途

**角色身上的「主動能力」模板**。介於「裝備」與「道具」之間：玩家可在戰鬥中**手動觸發**的能力（不像被動），但**綁定在特定角色身上**（不像道具放在共用背包）。例如「治療師：每戰可釋放一次群體治療」「劍士：每回合可手動引發暴擊」。

繼承自 `RCG_Asset<RCG_ActivePowerData>`，實作介面：`RCGI_Item` / `RCGI_Unloackable`。

## 編輯器中的樣貌

```
RCG_ActivePowerData: <ID>
    Data (m_Data)               ← 主要設定（Name / Description / Icon / TargetType / Price / Unlock / ItemUseType）
    ItemEffects                 ← 主動能力的效果（OnPlay 觸發）
    Preview
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Name / Description / Icon** | 是 | 名稱、風味描述、圖示 |
| **TargetType** | 是 | 使用時的選目標範圍 |
| **Price** | 是 | 商店價（如果可購買） |
| **ItemUseType** | 是 | `Consume` / `OncePerBattle` / `OncePerTurn` / `Infinite` |
| **UseTimes** | 視 ItemUseType | 可使用次數 |
| **Unlock** | 否 | 解鎖條件 |
| **ItemEffects** | 否 | 觸發效果（`RCG_CommonEffect` list） |

## 行為說明

### 與道具的差異
*   **道具**：放共用背包，任何角色都能用。
*   **主動能力**：綁定在特定角色身上（`RCG_DataService.Ins.m_ActivePowersData`），通常隨角色加入隊伍而帶來。

### 使用 / 觸發
`CheckUsable(data)` → 所有 effect 的 `CheckPlayable` 全 true 才能用。
`TriggerEffect(data)` → 取 `OnPlay` effects 依序觸發（含 try-catch）。

### 描述生成
與 `RCG_ItemData` 類似：自動由 effects 串接 + UseType 標記 + UseTimes 顯示。

## 注意事項

*   **`ShowInfo()` 是空殼**：原本要走 `RCG_ItemInfoPanel`，但被註解掉（`// QWQ`），目前無作用。實際 UI 由其他位置顯示。
*   **沒有 ItemType 欄位**（與 `RCG_ItemData` 不同）：因為已是「主動能力」分類，不再細分 Normal / QuestItem。
*   **`InitActivePowers` 的舊註解**：原本角色加入時會自動套用 InitActivePowers，但已轉用 UnitSkill 系統取代（程式內有大段註解標示）。
*   **無 Auto Price 工具**：與 ItemData / EquipmentData 不同，編輯器頁面沒做這個。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_ActivePowerData.cs`
*   **繼承自**：`RCG_Asset<RCG_ActivePowerData>`
*   **實作介面**：`RCGI_Item` / `RCGI_Unloackable`
*   **AssetGroup**：`EditCharacter`

### A.2 欄位對照（外層）

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_Data` | Data | `ActivePowerData`（巢狀） | `[SerializeField] protected` |
| `m_ItemEffects` | Effects | `List<RCG_CommonEffect>` | |

`ActivePowerData` 巢狀含 `m_Name` / `m_Description` / `m_TargetType` / `m_Price` / `m_Icon` / `m_ItemUseType` / `m_UseTimes` / `m_Unlock`。

### A.3 重要 Method 摘要

*   **`AddItem()`** — `RCG_ActivePower.Create(ID)` → `m_ActivePowersData.AddActivePower`。
*   **`CheckUsable(TriggerEffectData)`** — 全 effect AND check。
*   **`TriggerEffect(TriggerEffectData)`** — `OnPlay` effects 觸發。
*   **`Description` / `FullDescription` / `GetDescription`** — 描述生成（`RCG_ActivePower` 可選參數帶 runtime 剩餘次數）。
*   **`UseTime`** — 依 `ItemUseType` 回傳次數（Infinite → 0）。

### A.4 與其他系統的互動

*   **`RCG_ActivePower`** — runtime 主動能力實例。
*   **`RCG_DataService.Ins.m_ActivePowersData`** — 角色綁定儲存。
*   **`RCG_CommonEffect`** — 觸發效果單位。

### A.5 已知議題

*   `ShowInfo()` 已被註解（`// QWQ`），UI 入口需確認。
*   舊版 `InitActivePowers` 自動加成邏輯已棄用（被 UnitSkill 取代）。