---
title: Declare variable
description: Reads various battle conditions and stores the value into a named variable for downstream settings
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Declare variable

> Class: `RCG_VariableSetting`

## Purpose
**Reads a battle condition and stores the value into a named variable**, which downstream settings can reference. The core of "**dynamic effects**" — enables cards like "**deal 1 damage per attack card played this battle**".

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **VariableName** | Yes | Variable name (default `X`); referenced downstream. |
| **VariableCondition** | Yes | Source condition (**18 modes**, see below). |
| **Range** | TargetStatusCount/TargetAttackPower/TargetArmorCount/TargetDebuffLayersSum modes | Target range (attack-range style). |
| **UsedCardType** | CardUsedCount/TurnCardUsedCount modes | Card tag filter; empty = no filter. |
| **Status** | TargetStatusCount mode | Status type to query stacks of. |
| **MonsterTags** | MonsterCount mode | Monster type filter; empty = all. |
| **Value** | IntVariable mode | Direct value (variable). |

### 18 VariableCondition modes
| Value | Reads |
|---|---|
| **RemainCost** | Player's current energy (default) |
| **HandCardCount** | Hand size |
| **DiscardedCardCount** | Discard pile size |
| **DeckCardCount** | Draw pile size |
| **BanishedCardCount** | Banished pile size |
| **CardUsedCount** | Cards played this battle (tag-filterable) |
| **TurnCardUsedCount** | Cards played this turn (tag-filterable) |
| **ThisCardUseCount** | Total uses of this card |
| **ThisCardCost** | This card's current cost |
| **CostOfSelectedCards** | Sum of selected cards' costs |
| **TargetStatusCount** | Target's stack count of a status |
| **TargetDebuffLayersSum** | Target's total debuff stacks |
| **TargetArmorCount** | Target's armor |
| **TargetAttackPower** | Target's max attack power (across AI's actions) |
| **MonsterCount** | Live enemies (tag-filterable) |
| **PlayerUnitCount** | Live player units |
| **DarkMistLevel** | Dark Mist level (resource `Supply`) |
| **IntVariable** | Direct value or computed variable |

## Behaviour
*   At trigger: computes the value per VariableCondition → stores in `iData.VariableDic[VariableName]`.
*   Description: "**{condition desc} {VariableName}({actualValue}) = {value}**" — live-shows current value during battle.
*   Editor mode (out of battle): shows placeholder; doesn't actually evaluate.

## Notes
*   **Variable name collisions**: multiple settings using the same name overwrite each other. Use semantic names (`HandCount`, `EnemyAtk`).
*   **Variable scope**: stored in `iData.VariableDic` — shared within the same `TriggerEffectData` tree. **Child datas inherit visibility**.
*   **Empty VariableName**: `AddAction` returns early — the setting is **completely inert**.
*   **Computation timing**: evaluated at `AddAction` trigger; downstream reads see that snapshot.
*   **Preview damage**: `GetPreviewDamage` pre-writes the variable value to `VariableDic` so downstream preview can use it.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_VariableSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_VariableSetting` → "Declare variable"
*   **Top-level enum**: `VariableCondition` (18 values)

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_VariableName` | VariableName | `string` | `VariableName` | Default `"X"` |
| `m_VariableCondition` | VariableCondition | `VariableCondition` (file enum) | — | Default `RemainCost` |
| `m_Range` | Range | `AttackRange` | — | `[Conditional(... 4 range-needing conditions)]` |
| `m_UsedCardType` | UsedCardType | `List<RCG_CardTagGenData>` | — | `[Conditional(... CardUsedCount, TurnCardUsedCount)]` |
| `m_Status` | Status | `RCG_CustomStatusGenData` | `Status` | `[Conditional(... TargetStatusCount)]` |
| `m_MonsterTags` | MonsterTags | `List<RCG_MonsterTagGenData>` | — | `[Conditional(... MonsterCount)]` |
| `m_Value` | Value | `IntVariable` | — | `[Conditional(... IntVariable)]` |

### A.3 Key methods
*   **`GetVariableValue(iData)` (protected)**: 18-way switch computes value; returns 0 outside battle or when `RCG_Player.Ins == null`.
*   **`AddAction`**: empty `m_VariableName` returns; otherwise `iData.VariableDic.Set(m_VariableName, GetVariableValue(iData))`.
*   **`GetPreviewDamage`** → writes current value to VariableDic and returns `-1` (**non-attack but writes for downstream preview**).
*   **`VarName(iData)` (public)**: in-battle shows `{name}({value})`; out of battle just `{name}`.
*   **`GetDescriptionParams / GetDescriptionFormat`**: per condition, picks i18n key (`VariableCondition_*Des` / `VariableEffectDes` / `VariableEffectDes_IntVariable`).
*   **`Info`**: `TargetStatusCount` mode returns `m_Status` info; others null.

### A.4 Cross-system interactions
*   **`iData.VariableDic`**: write target; downstream `IntVariable` resolution reads it.
*   **`RCG_Player.Ins.Cost / GetHandCardCount / DiscardedCardCount / BanishedCardCount / Deck.Deck.Count`**: source data.
*   **`RCG_BattleAnalytics.Ins.GetCardUsedCount / GetTurnCardUsedCount`**: card-use stats.
*   **`RCG_BattleScene.Ins.GetUnits(UnitPos.All, faction)`**: unit count queries.
*   **`RCG_DataService.Ins.GetResource(Supply)`**: Dark Mist source.

### A.5 Known issues
*   Legacy `DeserializeFromJson` migration (e.g. `IntVariable → RemainCost`) is commented out.
*   `// QWQ2` / `// TODO` comments mark some pending work (e.g. `HandCardCount` description params).
