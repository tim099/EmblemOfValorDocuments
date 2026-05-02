---
title: Attack command
description: Like Attack but with a designated attacker; for "command another unit to attack" scenarios
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Attack command

> Class: `RCG_AttackCommandSetting`

## Purpose
Almost identical to **Attack**, but with an extra **Attacker** field — for "**commanding another unit to attack**":
*   Summons attacking on the player's behalf
*   Ally co-op attacks
*   Multi-unit combo skills

> [!NOTE]
> Use the regular **Attack** by default; reach for this only when you need an explicit attacker.

## Key fields

Almost the same as Attack, plus one extra:

| Inspector label | Required | Notes |
|---|---|---|
| **Attacker** | Yes | Selector for the attack initiator (overrides the `User` in iData). |
| **AttackTarget** | Yes | Same as Attack. |
| **Atk** | Yes | Same. |
| **AtkTimes** | Yes | Same. |
| **AttackType** | Yes | Same (Counter type auto-inherits from incoming damage). |
| **AttackTagGenDatas** | No | Same. |
| **AttackVFX** | Yes | Same. |
| **DetailSetting** | No | Same (AntiAttack, AttackAtOnce, ...). |

## Behaviour
*   Resolves **Attacker** to the actual attacker; if none found, **falls back to the first alive unit**.
*   With IgnoreBuff off, AtkTimes is filtered through `m_UnitStatus.GetAtkTimes(...)`.
*   Description: "**{attacker} deals {atk} damage to {target}**" (i18n `AttackDommand_Des`).
*   Other behaviour (preview damage, anti-attack display, AttackAtOnce, ...) matches Attack.

## Notes
*   **Attacker fallback can backfire**: if the selector returns nothing, the system **forces in the first alive unit** to avoid NRE — this can flip "enemy command" into "player attack". Design selectors carefully.
*   **vs Attack**: prefer the regular **Attack**; this setting exists only for the rare "command another" case.
*   **Counter type without inbound damage**: degrades to `Normal` (physical).

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_AttackCommandSetting.cs`
*   **Inherits**: `RCG_BattleSetting`, `[System.Serializable]`
*   **i18n class key**: `RCG_AttackCommandSetting` → "Attack command"
*   **TODO**: source has `// TODO: 測試 QWQ 能可以跟 RCG_AttackSetting 共用邏輯?` — heavy code duplication with `RCG_AttackSetting`.

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_Attacker` | Attacker | `RCG_SelectTargetData` | — | Override iData.User |
| `m_AttackTarget` | AttackTarget | `RCG_SelectTargetData` | `AttackTarget` | |
| `m_Atk` | Atk | `IntVariable` | `Atk` | `[UCL_FieldOnGUI]` |
| `m_AtkTimes` | AtkTimes | `IntVariable` | `AtkTimes` | `[UCL_FieldOnGUI]` |
| `m_AttackType` | AttackType | `AttackType` | `AttackType` | |
| `m_AttackTagGenDatas` | AttackTagGenDatas | `List<RCG_AttackTagGenData>` | `AttackTagGenDatas` | |
| `m_AttackVFX` | AttackVFX | `RCG_AttackVFXGenData` | `AttackVFX` | |
| `m_DetailSetting` | DetailSetting | `RCG_AttackSetting.DetailSetting` | `DetailSetting` | **Reuses** `RCG_AttackSetting.DetailSetting` |

### A.3 Key methods
*   **`AddAction`**: resolves `m_Attacker` → `aUser`; fallback to `RCG_BattleManager.Ins.AllAliveUnits[0]`. Branches on `m_DetailSetting.m_AttackAtOnce` (slam-all vs per-hit retarget); builds `AttackData`, queues `RCG_AttackAction`.
*   **`GetDescriptionFormat`**: per-attack description first (shared `LocalizeKey` logic), then wraps with `AttackDommand_Des` i18n.

### A.4 Cross-system interactions
*   **`RCG_AttackSetting.DetailSetting / DescriptionType`**: directly reused.
*   **`AttackData / RCG_AttackAction`**: same downstream as Attack.
*   **`RCG_BattleField.CurActiveUnit / RCG_BattleManager.AllAliveUnits`**: attacker resolution fallback.

### A.5 Known issues
*   Massive code duplication with `RCG_AttackSetting` (atk str / times / VFX / DetailSetting / preview) — should refactor to a shared base or helper.
