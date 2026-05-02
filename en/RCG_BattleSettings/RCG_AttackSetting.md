---
title: Attack
description: Deal damage to targets; covers attack power, hit count, type, tags, VFX, and detailed advanced parameters
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Attack

> Class: `RCG_AttackSetting`

## Purpose
Deal damage to **specified targets**. The most common and most parameter-rich battle setting — used by virtually every "hit a monster" card, enemy skill, and counter status.

## Inspector layout
```
▼ ?  ✓  [Attack(Attack)] Deal 1 physical damage
    ▶ AttackTarget (target)
      Atk             [Value] 1
      AtkTimes        [Value] 1
      AttackType      [Normal]
    ▶ AttackTagGenDatas (0)
    ▶ AttackVFX       [AttackEffectPlain]
    ▶ DetailSetting
```

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **AttackTarget** | Yes | Target selector. Expand for range (single / AOE / all enemies / all allies / ...) and target type. |
| **Atk** | Yes | Damage per hit (variable-bound — e.g. "= armor value", "= some status stack"). **Negative values clamp to 0**. |
| **AtkTimes** | Yes | Hit count. Each hit resolves independently; combined with DetailSetting it can re-pick targets per hit. |
| **AttackType** | Yes | Affects damage resolution. See table below. |
| **AttackTagGenDatas** | No | Attached attack tags (lifesteal, fire, crit, ...). Tags can carry their own rules (e.g. lifesteal converts X% damage to heal). |
| **AttackVFX** | Yes | Hit animation; you must pick a base attack VFX or there'll be no swing animation. |
| **DetailSetting** | No | Advanced params — see below. |

### AttackType reference
| Display | Internal | Meaning |
|---|---|---|
| **Normal** | Normal | Affected by armor and physical damage reductions |
| **Magic** | Magic | Affected by magic resist |
| **Status** | Status | Ignores armor and all reductions (status damage isn't a regular attack) |
| **Counter** | Counter | For counter-attacks; type matches the incoming damage |
| **Any** | Any | Wildcard for trigger conditions |

## DetailSetting (advanced)

| Inspector label | Default | Notes |
|---|---|---|
| **AntiAttack** | (empty) | Bonus damage vs specific monster tags (e.g. "anti-Dragon" for double damage). |
| **AttackAtOnce** | ✗ | Checked = same hit slams all targets at once (synced animation); unchecked = sequential per-target. |
| **StopAttackAfterDeath** | ✓ | On multi-hit, stop remaining hits if the target dies (avoids hitting air). |
| **CounterAttackVFX** | ✗ | Reuse the incoming attack's VFX when countering. |
| **IgnoreBuff** | ✗ | Ignore the attacker's buffs/debuffs (display + resolution). |
| **DescriptionType** | Default | Affects sentence template; `InflictDamageOn` uses "inflict X damage on Y" wording. |

## Behaviour

### Preview damage (small number top-right of card)
*   **Only computed when AttackTarget = "single chosen target"** (other modes return "non-attack card").
*   Includes current buffs/debuffs/anti-attack bonuses.
*   **Per-hit value**, NOT multiplied by AtkTimes.

### Atk display in description
*   IgnoreBuff off → shows post-buff value (e.g. "5+2" format).
*   IgnoreBuff on → shows raw `Atk` value.

### Card info (tooltip)
Auto-listed:
1. AttackType
2. Each attack tag
3. Anti-attack rules (if set)
4. AttackTarget extra info

## Notes

*   **Multi-hit + AOE re-picks targets per hit**: "AtkTimes=3 + AOE all enemies" re-resolves enemies each hit. For "each enemy hit 3 times", wrap an AOE attack inside **Loop**.
*   **Anti-attack vs description**: if AntiAttack says ×1.5 vs Dragons but card text says "×2 vs Dragons", players see inconsistency. **Both are manual — keep them in sync**.
*   **AttackVFX cannot be empty**: pick at least `AttackEffectPlain` even for "no-VFX counter".
*   **Variable Atk display**: use "Hidden Variable" mode for clean numbers (no `(varName)` suffix).

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_AttackSetting.cs`
*   **Inherits**: `RCG_BattleSetting`, `[System.Serializable]`
*   **i18n class key**: `RCG_AttackSetting` → "Attack"

### A.2 Field map (main class)

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_AttackTarget` | AttackTarget | `RCG_SelectTargetData` | `AttackTarget` | Replaces legacy `AttackRange` enum |
| `m_Atk` | Atk | `IntVariable` | `Atk` | `[UCL_FieldOnGUI]` |
| `m_AtkTimes` | AtkTimes | `IntVariable` | `AtkTimes` | `[UCL_FieldOnGUI]` |
| `m_AttackType` | AttackType | `AttackType` (enum) | `AttackType` | Subkeys `AttackType_Normal/_Magic/_Status/_Counter/_Any` |
| `m_AttackTagGenDatas` | AttackTagGenDatas | `List<RCG_AttackTagGenData>` | `AttackTagGenDatas` | Aliases: `AttackTags`, `AttackTag` |
| `m_AttackVFX` | AttackVFX | `RCG_AttackVFXGenData` | `AttackVFX` | |
| `m_DetailSetting` | DetailSetting | `DetailSetting` (nested) | `DetailSetting` | See A.3 |

### A.3 Field map (`DetailSetting` nested)

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_AntiAttack` | AntiAttack | `List<RCG_MonsterTagGenData>` | `AntiAttack` | |
| `m_AttackAtOnce` | AttackAtOnce | `bool` | `AttackAtOnce` | |
| `m_StopAttackAfterDeath` | StopAttackAfterDeath | `bool` | `StopAttackAfterDeath` | Default `true` |
| `m_CounterAttackVFX` | CounterAttackVFX | `bool` | `CounterAttackVFX` | |
| `m_IgnoreBuff` | IgnoreBuff | `bool` | `IgnoreBuff` | |
| `m_DescriptionType` | DescriptionType | `DescriptionType` (enum) | `DescriptionType` | `Default` / `InflictDamageOn` |

### A.4 Key methods
*   **`Infos`** → composes `[AttackType, ...AttackTagGenDatas, AntiAttakDes (if any), ...AttackTarget.Infos]` with dedup on the last segment.
*   **`GetAtkStr`**: with user + non-IgnoreBuff → `iUser.p_BattleUnit.m_UnitStatus.GetAtkDescription(...)`. `Value` / `HiddenVariable` modes set `iHasModification=false`.
*   **`GetAttackData`** → preview-only `AttackData` (no counter handling).
*   **`GetAtk`** → `Mathf.Max(0, m_Atk.GetValue(iData))`.
*   **`GetPreviewDamage`** → only if `m_SelectTargetType == Range && m_UnitRange == Target`; else `-1`.
*   **`DeserializeFromJson`** → calls base; legacy migrations commented out.

### A.5 Cross-system interactions
*   **`AttackData / RCG_PlayerAtkAction`**: actual attack runtime path.
*   **`RCG_SelectTargetData / SelectTargetType / UnitRange`**: shared target selector; old `AttackRange` enum is deprecated but kept for JSON migration.
*   **`IntVariable`**: numeric container.
*   **`RCG_MonsterTagGenData`**: anti-attack matcher.

### A.6 Known issues
*   Legacy `AttackRange` enum kept for old-JSON deserialization.
*   Legacy `CanEnhence` / `Enhence(RestSetting)` commented out.
*   Several deprecated description-related lines retained as comments.
