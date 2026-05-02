---
title: Status
description: Full-feature status setting — add / decrease / remove / convert / absorb / random-pool
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Status

> Class: `RCG_StatusSetting`

## Purpose
**The status core setting** — grants stacks, removes stacks, removes whole status, batch-removes by type, A→B conversion, and more. **One of the most-used settings** alongside Attack.

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **Target** | Yes | Status target selector. |
| **StatusModifiedType** | Yes | Modification mode (**10 modes**, see below). |
| **Status** | Add/Decrease/Remove/Convert/Set modes | Status type to operate (`RCG_CustomStatusGenData`). |
| **StatusDropPool** | StatusDropPool mode | Pool to draw from. |
| **ConvertStatus** | Convert mode | Target status to convert into. |
| **ConvertLayer** | Convert mode | Stacks consumed per conversion (default 10). |
| **ConvertRatio** | Convert mode | Ratio (default 1). E.g. ConvertLayer=10 + Ratio=3 → "every 10 Daze → 3 Stun". |
| **StatusTags** | RemoveStatusOfTargetType / AbsorbStatusOfTargetType | Status type list (e.g. "all Debuffs"). |
| **Amount** | Add / Decrease / Set | Stack delta (variable). |
| **DescriptionType** | — | Sentence template: `Default` / `Then`. |
| **DetailSetting** | — | Contains `SkipAnim` (skip the anim, set value directly). |

### StatusModifiedType modes
| Value | Behaviour |
|---|---|
| **AddStatusLayer** | Add stacks (default) |
| **DecreaseStatusLayer** | Decrease stacks |
| **RemoveStatus** | Remove the specified status entirely |
| **RemoveAllStatus** | Remove all statuses |
| **RemoveAllDebuff** | Remove all negative statuses |
| **RemoveStatusOfTargetType** | Batch-remove by StatusTags |
| **AbsorbStatusOfTargetType** | Steal a target's status type onto self |
| **ConvertStatus** | A → B conversion (per Layer / Ratio) |
| **SetStatusLayer** | Set stacks to a specific value |
| **StatusDropPool** | Random pick a status from a pool |

## Behaviour
*   Per mode, applied to `m_Target.GetTargets(iData)`.
*   Modes with `Amount` support variables (e.g. "Agile = hand size").
*   `Convert`: per `ConvertLayer` consumed, produces `ConvertRatio` stacks of ConvertStatus.
*   Description varies; UI shows i18n name + icon.

### Card fusion
Only `AddStatusLayer / DecreaseStatusLayer / SetStatusLayer` modes fuse (Amount adds). Other modes don't fuse.

## Notes
*   **AbsorbStatusOfTargetType absorbs to self**, not to the target selector — fixed behavior.
*   **StatusDropPool randomness**: seeded — same seed reproduces same pulls.
*   **SkipAnim speeds up resolution**: but players may miss the visual feedback. Use sparingly for important statuses.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_StatusSetting.cs`
*   **Inherits**: `RCG_BattleSetting`, `[System.Serializable]`
*   **i18n class key**: `RCG_StatusSetting` → "Status"
*   **Same-file types**: nested `DetailSetting` (with `m_SkipAnim`), `DescriptionType` enum (`Default`, `Then`)
*   **Top-level enum**: `StatusModifiedType` (10 values)

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_Target` | Target | `RCG_SelectTargetData` | `Target` | |
| `m_StatusModifiedType` | StatusModifiedType | `StatusModifiedType` | — | Default `AddStatusLayer` |
| `m_Status` | Status | `RCG_CustomStatusGenData` | `Status` | `[Conditional(... 5 modes)]` |
| `m_StatusDropPool` | StatusDropPool | `RCG_StatusDropPoolGenData` | — | `[Conditional(... StatusDropPool)]` |
| `m_ConvertStatus` | ConvertStatus | `RCG_CustomStatusGenData` | — | `[Conditional(... ConvertStatus)]` |
| `m_ConvertLayer` | ConvertLayer | `IntVariable` | — | Default 10 |
| `m_ConvertRatio` | ConvertRatio | `IntVariable` | — | Default 1 |
| `m_StatusTags` | StatusTags | `List<RCG_StatusTagGenData>` | — | `[Conditional(... 2 OfTargetType modes)]` |
| `m_Amount` | Amount | `IntVariable` | `Amount` | `[UCL_FieldOnGUI] + [Conditional(... 3 layer modes)]` |
| `m_DescriptionType` | DescriptionType | `DescriptionType` (file) | `DescriptionType` | |
| `m_DetailSetting` | DetailSetting | `DetailSetting` (file nested) | `DetailSetting` | Contains `m_SkipAnim` |

### A.3 Key methods
*   **`Fusion`**: requires same `m_StatusModifiedType` (one of three layer modes); clone + `IntVariable.FuseAdd(m_Amount)`.
*   **`GetShortName`**: branches per mode — direct i18n / `GetDescription()` / `m_StatusDropPool.GetData().GetShortName()`.
*   **`AddAction` / `Infos`** (beyond shown range): per-mode logic.

### A.4 Cross-system interactions
*   **`RCG_UnitStatus`**: actual status container (`GetStatusLayer`, `m_Status` dict).
*   **`RCG_StatusTagGenData.StatusDebuff / StatusDot`**: type tags for batch removal.
*   **`CreateAction.StatusAction`**: Action builder shared by `RCG_ConsumeStatusSetting`.

### A.5 Known issues
*   Legacy `CanEnhence` / `Enhence` commented out (enhancement system refactor).
