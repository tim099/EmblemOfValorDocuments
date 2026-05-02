---
title: Battle Setting (base)
description: Abstract base class for all battle settings (Attack, Heal, Conditional, Combine, etc.); shared IsEnable toggle, description system, and dropdown classification
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Battle Setting (base)

> Class: `RCG_BattleSetting`

## Purpose
The **abstract base class** for "one effect that can occur in battle". You don't pick this directly — it is the parent of all battle settings (Attack, Heal, Combine, Conditional, ...). When you see a "Battle Setting" field on a card / monster skill / status, the dropdown options are its subclasses.

## Inspector layout
Every `RCG_BattleSetting` subclass renders like this:
```
▼ ?  ✓  [Type(English)] short summary
```
*   **Checkbox**: the **IsEnable** field. Unchecked entries are skipped at runtime by container types (e.g. Combine).
*   **`?` icon**: opens this help doc (when `[HelpURL]` is attached).
*   **`[Type]` label**: the type's localized name (e.g. "Attack", "Heal", "Combine effect"); fallback to the stripped class name if no i18n key.
*   **Trailing text**: the subclass's own short description for quick identification when collapsed.

## Common fields

| Inspector label | Backing | Meaning |
|---|---|---|
| **IsEnable** | `m_IsEnable` | Whether this setting fires at runtime; container types auto-filter disabled entries. |

> [!NOTE]
> Subclass-specific fields (Atk, AtkTimes, etc.) are documented per subclass.

## Available subclass types

The dropdown auto-switches between two pools depending on context:

### When editing card/status/enhance data
50+ types covering:
*   **Effect**: Attack, Defense, Heal, Status, ...
*   **Resource**: Card draw / discard / cost-alter / consume item, ...
*   **Control flow**: Combine, Conditional, Loop, Foreach target, Random trigger, ...
*   **Advanced**: Summon, Unit transform, Collaboration, Counter alter, ...

### When editing `RCG_UnitData` / `RCG_MonsterActionData`
Two extra monster-only options appear:
*   **Monster flee** (`RCG_MonsterFleeSetting`)
*   **Monster move** (`RCG_MonsterMoveSetting`)

## Common behaviour (you don't configure these, but you'll see them)

*   **Auto-localized description**: each subclass produces a "long description" (card info panel) and a "short description" (enemy intent bar).
*   **Resource preload**: the system preloads icons/effects before battle to avoid in-battle hitching — **no manual setup required**.
*   **Fusion**: each subclass defines its own merge rule (e.g. two Heals add their amounts together).

## Notes

*   **Disabled settings still occupy data**: they show in the Inspector but skip at runtime. To remove entirely, click the `−` button.
*   **New subclass not appearing in dropdown?** Means the programmer hasn't added it to `AllTypes` — a code-side issue, not data.
*   **The base class itself is not selectable**: only concrete subclasses appear; the base only defines the contract.

---

## Appendix: Programmer Reference

> Below uses internal terminology aimed at programmers and AI agents. The user-facing section above is authoritative.

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_BattleSetting.cs`
*   **Inherits**: `UnityJsonSerializable`
*   **Implements**: `UCLI_ShortName`, `UCLI_TypeList`, `UCLI_GetTypeName`, `UCLI_IsEnable`, `UCLI_NameOnGUI`

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_IsEnable` | IsEnable (Enable) | `bool` | `IsEnable` | `[UCL_HideOnGUI]`; rendered manually by `NameOnGUI` |
| `IsShowOnUI` | (no GUI) | `static bool` | — | Global UI toggle; affects subclass description branches |
| `s_Types` / `s_MonsterDataTypes` | (no GUI) | `static List<Type>` | — | Subclass dropdown pool; `AllTypes` getter picks one based on context |

### A.3 Key methods
*   **`virtual CheckPlayable(TriggerEffectData)`** → default `true`; subclasses override for resource / condition checks.
*   **`virtual AddAction(...)`** → **core entry point**; pushes the corresponding Action onto `RCG_BattleManager`'s queue. Empty by default.
*   **`virtual GetDescription / GetDescriptionFormat / GetDescriptionParams`** → description trio; prefer overriding `Format + Params`.
*   **`virtual GetDescriptionShort`** → short text for enemy intent bar; default empty.
*   **`virtual GetShortName`** → 25-char truncation of description for collapsed Inspector.
*   **`virtual Info / Infos`** → tooltip info blocks; `Infos` defaults to wrapping single `Info`.
*   **`virtual GetBattleTags / HasTerm / GetCollaborators`** → tag, term, collaborator queries; container types must recurse.
*   **`virtual GetAtk / GetPreviewDamage`** → attack power / preview damage; latter returns `-1` to mark non-attack cards.
*   **`virtual PreloadData(CancellationToken)`** → preload before battle; container types must `await` children.
*   **`virtual Fusion / GetFusionCandidateSettings / GetFusionBaseSetting`** → fusion trio; default does not support / `[this]` candidate / placeholder base.
*   **`implicit operator string`** → calls `GetDescription()`; convenient for string concat but watch null.

### A.4 Cross-system interactions
*   **`AllTypes` getter**: checks `UCLI_Asset.s_CurOnGUIAsset` is `RCG_UnitData` or `RCG_MonsterActionData`; if so, returns `s_MonsterDataTypes`.
*   **JSON serialization**: inherits `UnityJsonSerializable`; default writes via `JsonConvert.SaveDataToJson(this, JsonConvert.SaveMode.Unity)`.
*   **GUI rendering**: `NameOnGUI` draws `IsEnable` checkbox + `[Type] short_desc`; type name comes from `GetTypeName(GetType().Name)` which strips `RCG_` and `Setting` then queries i18n.

### A.5 Known issues
*   `CanEnhence` / `Enhence(RestSetting)` are commented out (legacy enhancement system).
*   `DeserializeFromJson` / `SerializeToJson` overrides are commented (using base default); uncomment for customization.
