---
title: Hand card cost alter
description: Modify hand card costs (add / subtract / zero out) across various selection ranges
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Hand card cost alter

> Class: `RCG_CardCostAlterSetting`

## Purpose
**Modify hand card costs**. Examples:
*   "All hand cards cost -1"
*   "One random hand card becomes 0-cost"
*   "The most expensive hand card costs 0"

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **CardAlterRange** | Yes | Affected cards:<br>• **AllHandCards** — entire hand<br>• **SelectedCards** — pre-chosen cards<br>• **OneRandomHandCard** — one random<br>• **ThisCard** — the card itself (card-trigger only)<br>• **MostCostCard** — highest-cost card<br>• **NewSelectedCards** — choose now (auto-expands selector) |
| **CostAlterType** | Yes | Mode:<br>• **AddHandCardCost** — increase<br>• **SubHandCardCost** — decrease<br>• **HandCardCostToZero** — zero out (no value needed) |
| **SelectHandCardSetting** | When NewSelectedCards | Embedded selector; auto-hidden in other modes. |
| **Cost** | When Add/Sub | Delta value. |

## Behaviour
*   **NewSelectedCards** triggers the embedded selector first, then applies.
*   After picking cards by range, applies CostAlterType:
    *   Add → `AlterCost(+Cost)`
    *   Sub → `AlterCost(-Cost)`
    *   ToZero → `AlterCost(-currentCost)` (cancels current)

### Card fusion
Same `CostAlterType` cards fuse by adding `Cost`. **Different types cannot fuse** (avoids "+1 + zero out" semantic conflict).

## Notes
*   **OneRandomHandCard re-rolls**: re-evaluates randomly each trigger; multiple firings pick different cards.
*   **ThisCard inert outside card context**: status / event triggers have no `iData.Card`.
*   **HandCardCostToZero is "subtract current"**: not `-∞`. After "+1 cost" then "to zero" → ends at 0.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardCostAlterSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_CardCostAlterSetting` → "Hand card cost alter"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_CardAlterRange` | CardAlterRange | `CardAlterRange` (enum) | — | 6 sources |
| `m_CostAlterType` | CostAlterType | `CostAlterType` (enum) | — | 3 modes |
| `m_SelectHandCardSetting` | SelectHandCardSetting | `RCG_SelectHandCardSetting` | — | `[Conditional(m_CardAlterRange, false, NewSelectedCards)]` |
| `m_Cost` | Cost | `IntVariable` | `Cost` | |

### A.3 Key methods
*   **`AddAction`**: in NewSelectedCards mode, calls embedded selector first; then `AlterCost(...)` per range.
*   **`Fusion(other)`**: requires same `m_CostAlterType`; clones + `IntVariable.FuseAdd(m_Cost)`.
*   **`GetDescriptionFormat`**: per CostAlterType uses `AddHandCardCost_Des` / `SubHandCardCost_Des` / `HandCardCostToZero_Des`.

### A.4 Cross-system interactions
*   **`RCG_Card.AlterCost(int)`**: actual cost mutation entry.
*   **`UCL_Random.Instance.Next`**: source for OneRandomHandCard.
