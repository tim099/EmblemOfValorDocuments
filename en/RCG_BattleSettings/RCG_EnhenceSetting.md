---
title: Enhance
description: Attaches an extra effect (OnPlay) to selected hand cards, permanently modifying their behavior
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Enhance

> Class: `RCG_EnhenceSetting`

## Purpose
**Attaches an additional effect to selected hand cards**: when the enhanced card is played, it triggers `EnhenceSetting` **in addition to** its base effects. Examples:
*   "Enhance the next attack: add lifesteal"
*   "Enhance all hand cards: each draws 1 extra card"

The enhancement is permanent until the card is diminished or destroyed.

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **SelectHandCardSetting** | Yes | Card selector. |
| **EnhenceSetting** | Yes | The **Combine effect** to attach — trigger timing is fixed to `OnPlay`. |

## Behaviour
*   After selection, per card → `aCard.Enhence(commonEffect)` where `commonEffect` wraps `EnhenceSetting` with `OnPlay` trigger.
*   Description: "**{selector} : {enhance title} {enhance desc}**" (i18n `EnhenceSettingDes`, with `EnhenceTitle` color).
*   Tooltip (`Infos`): first entry is the enhancement summary; followed by the enhancement effect's own infos.

### Tags
The enhancement does **not** expose its inner tags via `m_EnhenceSetting.GetBattleTags` to the parent — by design, to avoid mis-counting on the original card.

## Notes
*   **Enhancements are permanent**: once attached, they stay with the card until destroyed or diminished.
*   **Trigger timing fixed to `OnPlay`**: cannot currently be changed. For other timings, use Conditional + Status combinations.
*   **Diminish counterpart**: when diminished, enhancement leaves are consumed first.
*   **Tags don't leak through**: don't expect "Brittle", "Agile" etc. inside the enhancement to show on the original card's tooltip.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_EnhenceSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_EnhenceSetting` → "Enhance"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_SelectHandCardSetting` | SelectHandCardSetting | `RCG_SelectHandCardSetting` | — | `[SerializeField] protected` |
| `m_EnhenceSetting` | EnhenceSetting | `RCG_CombineSetting` | — | `[SerializeField] protected` |

### A.3 Key methods
*   **`AddAction`**: select → per card build `RCG_CommonEffect { m_CombineSetting = m_EnhenceSetting, m_EffectTriggerOn.m_TriggerOn = OnPlay }`, call `aCard.Enhence(aEffect)`.
*   **`GetBattleTags`** → always empty list (**intentional override**, doesn't leak inner tags).
*   **`GetDescriptionFormat`**: `{selector desc}\n{Title} : {Enhance desc + inner BattleTags}`.
*   **`GetEnhenceSettingDescription` (private)**: combines `m_EnhenceSetting.GetDescription` + its battle tag descriptions.
*   **`Infos`**: prepends `"RCG_EnhenceSetting" + "\n" + EnhenceSettingInfo(...)` as item 0.

### A.4 Cross-system interactions
*   **`RCG_CommonEffect`**: enhancement wrapper (with `TriggerOn`).
*   **`RCG_CardBattleData.Enhence(commonEffect)`**: actual attachment entry.
*   **`RCG_Extensions.TagColors.EnhenceTitle`**: title color.
*   **i18n keys**: `RCG_EnhenceSetting`, `EnhenceSettingDes`, `EnhenceSettingInfo`.

### A.5 Known issues
*   Legacy `m_CostAlter` / `m_EffectTriggerTiming` fields are commented out (enhancement system refactor in progress); cost adjustment and custom trigger timing are currently unavailable.
