---
title: 棄牌 說明
description: 將手牌移出（棄牌、消滅、回牌堆頂、保留等）；支援多種選取方式與標籤篩選
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 棄牌

> 程式類別名稱：`RCG_CardDiscardSetting`

## 用途
**將手牌移出當前位置**到指定處（棄牌堆 / 消滅 / 回牌堆頂等），是「棄牌卡」「消滅卡」的核心設定。

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **DiscardType** | 是 | 棄牌方式：<br>• **DiscardByCount** — 玩家自行選擇 N 張<br>• **DiscardAll** — 全部手牌<br>• **DiscardAnyCount** — 玩家任意數量<br>• **RandomDiscardByCount** — 隨機 N 張<br>• **SelectedCards** — 已被前置設定選好的卡<br>• **SelectHandCard** — 透過內嵌的選取設定 |
| **DiscardCardNum** | DiscardByCount/RandomDiscardByCount 時必填 | 棄牌張數（支援變數）。`Conditional` 控制顯示。 |
| **RemoveType** | 是 | 移除方式：`Discard` 棄牌 / `Banish` 消滅 / `ToDeckTop` 回牌堆頂 / `TurnEndDiscard` 回合結束時棄牌（不觸發棄牌效果）/ `Retain` 留在手牌。 |
| **SelectHandCardSetting** | DiscardType=SelectHandCard 時必填 | 內嵌的選取設定。 |
| **DiscardCardTags** | 否 | 限定要棄的卡的標籤（如「攻擊牌」）。 |
| **DiscardCardNotIncludedTags** | 否 | 反向選取（不含這些標籤的卡）。 |
| **DiscardVariable** | 否 | 把實際棄掉的張數**寫入變數**，後續設定可引用（例如「棄牌後抽 X 張」中的 X）。 |

## 行為說明
*   依 DiscardType 取得待棄卡清單，呼叫 `CreateAction.AddDiscardCardActions(...)` 並指定 `RemoveType`。
*   有 `DiscardVariable` 時，把張數存進 `iData.VariableDic[變數名]`。
*   `DiscardAnyCount` 模式下若有 `DiscardVariable`，描述會顯示 `X` 而非 `N`。
*   描述會根據 `RemoveType` 套不同 i18n key（例如 `BanishCardIcon_Des` / `DiscardSelectedCardsIcon_Des`）。

### 卡片融合
僅 `DiscardByCount` 與 `RandomDiscardByCount` 之間可融合（張數加總）；其他模式不可融合。

## 注意事項
*   **TurnEndDiscard 不觸發棄牌效果**：故意設計用於「鎖定卡片在手中，回合結束強制清掉」的情境，不要拿來當一般棄牌。
*   **DiscardAnyCount 沒有 DiscardCardNum**：填上限請改用 `DiscardByCount`；要任意張數請保持空白並善用 `DiscardVariable`。
*   **變數命名衝突**：若多個設定都用 `DiscardVariable = "X"`，後者會覆蓋前者 — 請命名清楚。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardDiscardSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_CardDiscardSetting` → 「棄牌」
*   **同檔案巢狀類**：`SelectCardSetting`（支援以 `RCG_CardTagGenData` 過濾，含 `m_NotIncludedTags` 反選與 `m_WithoutEnhancement`）

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_DiscardType` | `DiscardType` | enum | — | 6 種模式 |
| `m_DiscardCardNum` | `DiscardCardNum` | `IntVariable` | — | `[Conditional(m_DiscardType, false, DiscardByCount, RandomDiscardByCount)]` |
| `m_RemoveType` | `RemoveType` | `RemoveType` (檔內 enum) | — | `Discard` / `Banish` / `ToDeckTop` / `TurnEndDiscard` / `Retain` |
| `m_SelectHandCardSetting` | `SelectHandCardSetting` | `RCG_SelectHandCardSetting` | — | `[Conditional(m_DiscardType, false, SelectHandCard)]` |
| `m_DiscardCardTags` | `DiscardCardTags` | `List<RCG_CardTagGenData>` | — | |
| `m_DiscardCardNotIncludedTags` | `DiscardCardNotIncludedTags` | `bool` | — | |
| `m_DiscardVariable` | `DiscardVariable` | `string` | — | |

### A.3 重要 Method 摘要
*   **`AddAction`**：依 `m_DiscardType` 走 6 種分支；共通走 `CreateAction.AddDiscardCardActions(iData, list, mode, RemoveType)`。
*   **`Fusion`**：僅 `DiscardByCount` / `RandomDiscardByCount` 模式可融合；clone + `IntVariable.FuseAdd` 合併張數。
*   **`GetDescriptionFormat`**：依 RemoveType / DiscardType 雙重分流套 i18n key（例如 `Banish{Type}Icon_Des`）。

### A.4 與其他系統的互動
*   **`CreateAction.AddSelectCardAction / AddDiscardCardActions`**：UI 選牌與棄牌動作建構工具。
*   **`RCG_GameManager.Random.RandomPick`**：RandomDiscard 的隨機來源。
*   **`SelectCardSetting`**：標籤過濾的工具類（同檔案）。
