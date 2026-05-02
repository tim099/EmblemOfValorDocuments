---
title: Card data (RCG_CardData)
description: Master template for every card in the game — basic info, effects, enhancement branches, passive tags, unlock conditions
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Card data

> Class: `RCG_CardData`

## Purpose
**The master template for every card in the game.** Holds basic info (name, cost, rarity, icon), the actual effects, enhancement branches, passive tags, and unlock conditions. Found under the `EditItems` group in the Developer Page; every playable / shop / dropable card is an instance of this class.

Inherits `RCG_Asset<RCG_CardData>`. Implements: `RCGI_Item` (can be added to inventory) / `RCGI_CardData` (card data contract) / `RCGI_Unloackable` (unlockable).

## Inspector layout
Editing a card splits into three blocks:
```
CardData                ← Basic fields: Name / Cost / Type / Rarity / Tags / Unlock / ...
Effects (N)             ← m_Effects: actual card effects (OnPlay etc.)
PassiveEffects (N)      ← m_PassiveEffects: conditional BattleTags (affects Cost calc, etc.)
Preview                 ← Live preview of card appearance + description
```

The right-side **Preview** shows the rendered card, tooltip tags, art, unlock condition; offers **Enhanced Branch preview** and **Copy Description** buttons.

## Key fields (CardData)

### Basic info
| Inspector label | Required | Notes |
|---|---|---|
| **LocalizeName** | Yes | Card name (`RCG_LocalizeData`, multi-language). |
| **Cost** | Yes | Energy spent to play; integer. |
| **CardType** | Yes | `Default` (normal) or `Chant` (**chant card**: requires turns of accumulation before play). |
| **ChantTurn** | When CardType=Chant | Number of turns to chant (variable). Conditional on type. |
| **Rarity** | Yes | `Bronze` / `Silver` / `Gold` / `Legend` / `Cursed` etc. (`RCG_RarityTagGenData`). |
| **TargetType** | Yes | 17 values: `None`, `Friend`, `Allied(Front/Back)`, `Enemy(Front/Back)`, `All`, `AnyPos`, `AlliedNonLeader`, `AnySummoned`, etc. |
| **UsedTypeTag** | Yes | Post-play handling (e.g. `Consume`, `Reusable`, `Ethereal`). |
| **Price** | Yes | Merchant price; integer. Default 100. |
| **CardIcon** | Yes | Card art (`RCG_SpriteData`). |

### Tags & skills
| Inspector label | Required | Notes |
|---|---|---|
| **SkillTags** | No | Required character skills (Fighter / Mage / Cleric...); empty = any character. Multiple = "**any one match works**". |
| **CardTags** | No | Classification tags (Attack / Skill / Chant / Consume...). **Important**: use `HasTag()` to check chant — `m_CardTags` doesn't auto-include the Chant tag. |

### Card effects
| Block | Notes |
|---|---|
| **Effects** (`m_Effects`) | Main card effect list; each is an `RCG_CommonEffect` containing a "trigger timing (OnPlay / OnDraw / OnDiscard / ...) + Combine effect". Runs when the card is played or trigger condition fires. |
| **PassiveEffects** (`m_PassiveEffects`) | Conditional `RCG_ConditionalPassiveBattleTag` list; used for **stat queries** (e.g. "if hand size ≥ 5, -1 cost"). **Does NOT participate in OnTriggerEffect** — purely affects Cost-style attribute calculation. |

### Enhancement system
| Inspector label | Required | Notes |
|---|---|---|
| **BannedEnhence** | No | Enhancement branch IDs that must NOT be drawn for this card. |
| **EnhencePool** | Yes | The `RCG_CardEnhenceDropPoolGenData` to randomly pick branches from. |
| **ApplyEnhenceOnAllTiming** | — | Whether to apply the enhancement to all trigger timings on this card (not just OnPlay). |
| **UpgradeLv** | — | Display: ≥1 appends `+1` / `+2` to the card name. Usually system-set; manual changes not recommended. |
| **EnhenceLocalize** | No | Alternative naming for enhanced (e.g. α, β, γ); empty defaults to `+` prefix. |

> [!IMPORTANT]
> Currently **enhancement count is capped at 1** (only enhanceable when `m_UpgradeLv < 1`), and **Cursed cards cannot be enhanced**. `CanEnhence` enforces this.

### Description
| Inspector label | Required | Notes |
|---|---|---|
| **CardDescriptionType** | Yes | `Auto` (auto-composed from effects) or `Manually` (hand-written). |
| **CardDescription** | When Manually | Hand-written description body (multi-language); auto-hidden in Auto mode. |
| **DescriptionTemplate** | When Manually | Template string with placeholders like `{(OnPlay.0)}`; press `Generate Description Template` button to auto-derive from current effects. |

> [!TIP]
> Auto mode handles 95% of cases — composes effect descriptions, target type, battle tags into clean text. Switch to Manually only when you need **precise sentence control** / **hide some effects** / **add flavor text**.

### Unlock & misc
| Inspector label | Required | Notes |
|---|---|---|
| **Unlock** | No | Unlock condition (`RCG_UnlockEntry`); won't appear in rewards / shop until unlocked. |
| **HideInCodex** | — | Hide from codex (testing / stub cards). |
| **Note** | — | Plain text note (no functional effect; designer's reference). |
| **Author** | — | Card author signature. |

## Behaviour

### Card display name
Computed from `LocalizedName`:
*   Normal: directly `m_LocalizeName.Name`
*   Enhanced (`UpgradeLv ≥ 1`):
    *   `EnhenceLocalize` non-empty → `{Name}{α/β/γ}`
    *   `UpgradeLv == 1` → `{Name}+`
    *   `UpgradeLv ≥ 2` → `{Name}+{N}`
*   Chant card: in-game prefixed with "Chant" + colored; in Editor displayed as `Chant[Name]`

### Chant cards (CardType = Chant)
*   Must "chant" for `m_ChantTurn` turns before playable.
*   Description prepends a "**Chant[N]**" line (chant color).
*   `HasTag(TagChant)` returns true (even if `CardTags` doesn't list it).

### Description generation
*   **Auto**: composed in order of "TargetType desc → per-OnTriggerOn group → trigger effects → battle tags → passive effects".
*   **Manually**: uses `m_CardDescription` as the body, substituting placeholders like `(OnPlay.0)` from `GetDescriptionParams()`.

### Card info panel (right side of tooltip)
Display order:
1. Required skills (if any)
2. Each effect's `Infos` (deduplicated)
3. Battle tag explanations
4. Passive effects' conditions + tag info
5. UsedType (when `m_ShowTag = true`)
6. Chant cards add chant explanation at the top

### Enhancement branches
*   `GetEnhenceBranchs(N)` draws N branches from `EnhencePool`.
*   Filters: `BannedEnhence` excluded; `RCG_CardEnhenceCondition` failures excluded.
*   Each branch clones the card and bumps `UpgradeLv`, then applies the enhancement.

### Card fusion
This class **doesn't define fusion rules directly**; leaf-effect merging is delegated to each `RCG_BattleSetting` subclass (see `GetFusionCandidateSettings / GetFusionBaseSetting`).

## Notes

*   **Chant tag isn't manually added to CardTags**: the system auto-includes it in `HasTag` checks. But you must set `CardType = Chant`.
*   **Auto Price button**: the editor page has an "Auto Price" button that overwrites all card prices with "Rarity value × 5". **Manually-set prices will be wiped** — avoid the button if you need custom prices.
*   **Forgetting Unlock**: empty `Unlock` = always unlocked; remember to fill for hidden achievement cards.
*   **m_Note / m_Author are invisible to players**: developer notes / signature only.
*   **Multi-language consistency for Description**: in Manually mode, fill all languages; missing entries fall back to default language.
*   **Enhancement Pool is per-instance**: branches are drawn from the card's own `EnhencePool`; if same-name cards need different routes, configure each.

---

## Appendix: Programmer Reference

> Below uses internal terminology aimed at programmers and AI agents. The user-facing section above is authoritative.

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_CardData.cs`
*   **Inherits**: `RCG_Asset<RCG_CardData>`
*   **Implements**: `RCGI_Item` / `RCGI_CardData` / `RCGI_Unloackable`
*   **AssetGroup**: `AssetGroup.EditItems`, sort = `RCG_CardData`
*   **i18n class key**: not directly defined; uses AssetGroup name for EditItems classification.

### A.2 Field map (outer `RCG_CardData`)

| Code | Inspector | Type | Notes |
|---|---|---|---|
| `m_Data` | CardData | `CardData` (nested) | Main data; details in A.3 |
| `m_Effects` | Effects | `List<RCG_CommonEffect>` | Trigger effect list |
| `m_PassiveEffects` | PassiveEffects | `List<RCG_ConditionalPassiveBattleTag>` | Conditional passive tags; for stat queries, NOT in OnTriggerEffect |

### A.3 Field map (nested `CardData`)

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_LocalizeName` | LocalizeName | `RCG_LocalizeData` | `LocalizeName` | |
| `m_Cost` | Cost | `int` | `Cost` | Default 0 |
| `m_CardType` | CardType | `CardType` (file enum) | `CardType` | `Default` / `Chant` |
| `m_ChantTurn` | ChantTurn | `IntVariable` | `ChantTurn` | `[Conditional(nameof(m_CardType), false, CardType.Chant)]` |
| `m_Rarity` | Rarity | `RCG_RarityTagGenData` | `Rarity` | |
| `m_TargetType` | TargetType | `TargetType` (file enum) | `TargetType` | 17 values |
| `m_UsedTypeTag` | UsedTypeTag | `RCG_UsedTypeTagGenData` | `UsedTypeTag` | |
| `m_Price` | Price | `int` | `Price` | Default 100 |
| `m_SkillTags` | SkillTags | `List<RCG_SkillTagGenData>` | `SkillTags` | |
| `m_CardTags` | CardTags | `List<RCG_CardTagGenData>` | `CardTags` | |
| `m_BannedEnhence` | BannedEnhence | `List<RCG_CardEnhenceGenData>` | `BannedEnhence` | |
| `m_EnhencePool` | EnhencePool | `RCG_CardEnhenceDropPoolGenData` | — | |
| `m_ApplyEnhenceOnAllTiming` | ApplyEnhenceOnAllTiming | `bool` | — | |
| `m_UpgradeLv` | UpgradeLv | `int` | `UpgradeLv` | Default 0 |
| `m_EnhenceLocalize` | EnhenceLocalize | `RCG_LocalizeData` | — | Replaces `+1`/`+2` display |
| `m_CardIcon` | CardIcon | `RCG_SpriteData` | `CardIcon` | Default points to `Card_AngelsWings` |
| `m_CardDescriptionType` | CardDescriptionType | `CardDescriptionType` (file enum) | `CardDescriptionType` | `Auto` / `Manually` |
| `m_CardDescription` | CardDescription | `RCG_LocalizeData` | `CardDescription` | `[Conditional(... Manually)]` |
| `m_DescriptionTemplate` | DescriptionTemplate | `string` | `DescriptionTemplate` | `[Conditional(... Manually)]` |
| `m_Unlock` | Unlock | `RCG_UnlockEntry` | `Unlock` | |
| `m_HideInCodex` | HideInCodex | `bool` | `HideInCodex` | |
| `m_Note` | Note | `string` | `Note` | |
| `m_Author` | Author | `string` | — | |
| `m_DataType` | (hidden) | `DataType` | — | `[UCL_HideOnGUI]`, `BuiltIn` / `InGameRuntime` etc. |

### A.4 Key methods

#### Interface implementation
*   **`AddItem()`** → `RCG_DataService.Ins.m_DeckData.AddCard(ID)`, returns self; entry point when player obtains the card.
*   **`ToCardData`** → `this`.
*   **`ToCardBattleData`** → `null`; **deprecated**, use `RCG_CardBattleData.CreateCard()`.
*   **`Icon` / `IconSpriteData` / `IconTexture`** → `m_Data.m_CardIcon` properties.
*   **`ItemName` / `LocalizedName`** → `m_Data.LocalizeName` (with enhancement-level suffix logic).
*   **`ShowInfo()`** → opens `RCG_CardInfoPanel`.
*   **`GetPreviewDamage(target)`** → default -1 (non-attack at CardData layer).
*   **`UnlockEntry`** → `m_Data.m_Unlock`.

#### Description system
*   **`Description` (property)** → `DisplayDescription()`.
*   **`DisplayDescription(iData)`** → chant cards prepend chant title, append UsedType (if `m_ShowTag`).
*   **`GetDescription(iData, iShowBattleTags)`**: `Manually` mode + non-empty `m_CardDescription` → applies params and returns. `Auto` mode → composes "target desc + per-effect desc + battle tags".
*   **`GetDescriptionTemplate(iData)`** → produces `(OnPlay.0)`-style template (for `Generate` button in Manually mode).
*   **`GetDescriptionParams(iData)`** → collects all effects' params (for Manually template substitution).
*   **`CardDisplayName`** → card name (chant card adds chant prefix / color).
*   **`CostStr`** → `Cost.ToString()`.
*   **`ChantTitle(iData)`** → "Chant[X]" string (green).

#### Tags / terms / collaborators
*   **`CardTags`** → `m_Data.m_CardTags` (excluding chant tag).
*   **`HasTag(tag)` / `HasTag(tags)`** → contains the "chant card auto-includes `TagChant`" special case.
*   **`HasTerm(term)`** → recurses through all `CardEffects`.
*   **`GetBattleTags()`** → aggregates over all `CardEffects`.
*   **`GetPassiveTags(iData)`** → returns `m_PassiveEffects`; for stat queries like Cost.
*   **`GetCollaborators()`** → aggregates over all `CardEffects`, deduplicated.
*   **`GetTagsDescription(showHiddenTags)`** → tag description string (hidden tags wrapped in `[]`).

#### Battle / fusion related
*   **`CheckPlayable(iData)`** → false if UsedType contains `Unplayable`; otherwise AND across all `CardEffects.CheckPlayable`.
*   **`CheckRequireSkill(skill)` / `CheckRequireSkill(skills)`** → all-conditions AND.
*   **`CanPlayThisCard(skills)`** → "**any one skill match**" (different from `CheckRequireSkill`).
*   **`GetBattleSettings<T>() / (Type)`** → recurses through all `m_Effects[i].m_CombineSetting`.
*   **`OnTriggerEffect(triggerOn, iData)`** → fetches `m_Effects.GetEffects(triggerOn)`; creates child if iData null; calls `aEffects.TriggerEffects(iData)` and logs stats.

#### Enhancement branches
*   **`CanEnhence`** → `m_Data.m_UpgradeLv < 1 && !CardRarity.Equals(Cursed)`.
*   **`CreateEnhenceCard(ID)` (private)** → clones self, sets new ID, `m_UpgradeLv += 1`, marks `InGameRuntime`.
*   **`GetEnhenceBranchs(branchCount)` (yield)** → draws from `EnhencePool`, filters via `BannedEnhence` and `CheckCondition`, clones + applies each branch.

#### Serialization / data flow
*   **`SaveToJson(iData)` (static)** → writes `ID` + `IsRuntimeCardData`.
*   **`LoadFromJson(iData)` (static)** → if `IsRuntimeCardData = true` → `m_GameRuntimeAsset.GetRuntimeData<RCG_CardData>(aID)`; else `Util.GetData(aID)`.
*   **`Save()`** → base + clears `RCG_CardFilter` cache.
*   **`CloneCard()`** → deep copy via `SerializeToJson + DeserializeFromJson`.
*   **`CreateSelectAssetPage()`** → `RCG_CardDataEditorPage.Create()`.
*   **`OnLoadModule()` (static)** → `RCG_CardFilter.Clear()` + `Util.PrewarmAllAssets()`.
*   **`PreloadData(token)`** → marks `Preloaded`, preloads card icon, rarity icon, each effect's `PreloadData`.

#### Edit / preview
*   **`OnGUI(iDataDic)`** → three-section render: CardData → Effects → PassiveEffects → Preview. Manually mode adds "Generate Description Template" button + description params editor.
*   **`Preview(iDic, iIsShowEditButton)`** → icon + name + tags + description + enhance branch button + unlock + DisplayCardInfos.
*   **`DisplayCardInfos(iShowOnUI)`** → toggles `RCG_BattleSetting.IsShowOnUI` then assembles full InfoPanel content (with chant info).

### A.5 Cross-system interactions

*   **`RCG_CardBattleData`** — actual battle-runtime card instance; `RCG_CardData` is the template, `RCG_CardBattleData.CreateCard(cardData, isChanted)` produces instances.
*   **`RCG_CommonEffect`** — card effect unit; `m_Effects` is a `List` of these, containing `m_EffectTriggerOn` + `m_CombineSetting`.
*   **`RCG_ConditionalPassiveBattleTag`** — passive conditional tag; contains `m_Conditions` + `m_Tags`, affects stat queries but not OnTrigger.
*   **`RCG_CardEnhenceData / RCG_CardEnhenceDropPoolGenData`** — enhancement branch system; core for `GetEnhenceBranchs`.
*   **`RCG_CardFilter`** — card filter cache; cleared by `Save()` and `OnLoadModule()`.
*   **`RCG_DataService.Ins.m_DeckData`** — player deck storage; `AddItem` writes here.
*   **`RCG_DataService.Ins.m_GameRuntimeAsset`** — runtime card storage (e.g. those produced by CardToItem); `LoadFromJson` reads when `IsRuntimeCardData = true`.
*   **`UI.RCG_CardInfoPanel` / `RCG_CardDataEditorPage` / `RCG_PreviewEnhenceCardPage`** — main UI entries.
*   **`RCG_CardDataComparer`** (same file) — sort helper: skill class → rarity → cost descending.

### A.6 Known issues

*   `ToCardBattleData` is deprecated but kept (`return null`); comment states "do NOT use this for CardBattleData conversion".
*   `CanEnhence` hard-codes "enhancement count ≤ 1"; lifting that requires editing this property.
*   `m_PassiveEffects`'s `GetPassiveTags` has comments `// QWQ23` and `// Append does not work QWQ?` — uses `Add` instead of `Append` (implementation concern noted).
*   `InitData()` is virtual but currently empty; subclass `RCG_CardBattleData` actually uses it.
*   Legacy `m_UpgradeCards` field is commented out (originally for listing enhancement targets).
