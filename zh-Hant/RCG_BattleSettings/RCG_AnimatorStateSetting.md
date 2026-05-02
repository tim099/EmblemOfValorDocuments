---
title: AnimatorState（動畫狀態）說明
description: 控制單位 Animator 狀態切換或一次性動畫播放的戰鬥設定（無描述顯示）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# AnimatorState（動畫狀態）

> 程式類別名稱：`RCG_AnimatorStateSetting`

## 用途
**純動畫驅動**的戰鬥設定，無遊戲邏輯影響：
*   **持續狀態切換**（例如「進入防禦姿勢」「站起」）
*   **一次性動畫播放**（例如「揮手」「眨眼」）

> [!NOTE]
> 此設定**不會出現在卡片描述上**（`GetDescriptionFormat` 與 `GetDescriptionShort` 都回傳空字串），純粹作為視覺呈現。

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **ActionType** | 是 | 動作類型：<br>• **SetState** — 切換持續性的 Animator bool 狀態（例如進入/離開防禦姿勢）。<br>• **PlayAnim** — 播放一次性動畫並等到結束。 |
| **AnimType** (`UnitAnimType`) | 是 | 要操作的動畫類型（攻擊、被擊、防禦…）。 |
| **Enable** | ActionType=SetState 時必填 | bool；切換持續狀態時的目標值（true = 進入，false = 離開）。`PlayAnim` 模式下會自動隱藏。 |
| **Target** | 是 | 動畫播放的目標單位選擇器。 |

## 行為說明
*   **SetState** 模式：對每個目標的 `UnitAnimController.SetAnimState(AnimType, Enable)`。
*   **PlayAnim** 模式：對每個目標 await `PlayAnim(AnimType)` 直到動畫結束（**會卡住戰鬥流程直到完成**）。
*   無動畫控制器的目標會被靜默跳過。

## 注意事項
*   **PlayAnim 會延遲後續動作**：因為是 await 式，戰鬥節奏會被它拉長，安排在迴圈內要小心節奏感。
*   **SetState 開關務必對稱**：開了一個 `SetState(true)` 卻沒對應的 `SetState(false)`，狀態會持續到戰鬥結束。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_AnimatorStateSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **無 i18n 類別名 key**：編輯器顯示 stripped name `AnimatorState`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_ActionType` | `ActionType` | `AnimActionType` (enum) | — | `SetState` / `PlayAnim` |
| `m_AnimType` | `AnimType` | `UnitAnimType` (enum) | — | 動畫類型 |
| `m_Enable` | `Enable`（=「啟用」） | `bool` | `Enable` | `[Conditional(nameof(m_ActionType), false, AnimActionType.SetState)]` 條件顯示 |
| `m_Target` | `Target` (=「目標」) | `RCG_SelectTargetData` | `Target` | |

### A.3 重要 Method 摘要
*   **`GetDescriptionFormat / GetDescriptionShort`** 都回傳 `string.Empty` — 刻意隱藏。
*   **`AddAction`** 包 `AddAsyncActionTrigger`，依 `ActionType` 走不同分支。

### A.4 與其他系統的互動
*   **`UnitAnimController.SetAnimState / PlayAnim`**：實際的動畫驅動入口。
*   **`UnitAnimType`**：動畫類型 enum（與 Spine animation name 對應）。
