---
title: Story Data (RCG_StoryData)
description: Complete story definition — start event, sub-story dictionary, conditions, random start, tag filtering
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Story Data

> Class name: `RCG_StoryData`

## Purpose

**Full script data of a "story"**. More complex than `RCG_MapEventData` — contains a dialogue tree composed of multiple sub-stories (`SubStory`); player choices jump to different sub-stories. Example: "Merchant" can be a Story: starting screen → choice → "Buy" / "Leave" / "Kill merchant" → respective sub-stories → endings.

Inherits from `RCG_Asset<RCG_StoryData>`. Implements: `UCLI_ShortName`.

## Editor Layout

```
RCG_StoryData: <ID>
    StoryData (m_StoryData)         ← story metadata
        Tags / CanTriggerRepeatedly / Conditions
        IsRandomStart / RandomStartStories
    Story (StartEvent)              ← start SubStory (key = "Start")
    SubStory (m_EventDic)           ← all sub-stories dict: { id → RCG_OptionEventData }
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **Tags** | no | Story tags (used by StoryDropPool filtering) |
| **CanTriggerRepeatedly** | — | Whether repeatable |
| **Conditions** | no | Trigger conditions (OR; empty = unconditional) |
| **IsRandomStart** | — | Whether to randomly select among multiple start points |
| **RandomStartStories** | when IsRandomStart=true | List of random start points + weights |
| **Story** | yes | Main start event (key fixed to `"Start"`) |
| **SubStory** | no | All sub-stories dict (key = state name, value = `RCG_OptionEventData`) |

Each `RCG_OptionEventData` (sub-story) contains dialogue / options / result effects; can specify the next SubStory's ID, forming a tree.

## Behavior

### Start Selection (`GetStartSubStory(iSubStory)`)
*   If `iSubStory` specified → use it.
*   Else if `m_IsRandomStart` and `m_RandomStartStories` non-empty → weighted random pick.
*   Else falls back to `StartStateName = "Start"`.

### Trigger (`StartStory(token, iSubStory, iIsRoot)`)
1. Use `GetStartSubStory(iSubStory)` to determine the start point.
2. Write to `RCG_DataService.Ins.StartStory(ID, iSubStory, iIsRoot)` (records triggered story).
3. Set `s_CurStoryData = this` (lets other systems query the current story).
4. Get the corresponding SubStory's `RCG_OptionEventData.StartEvent(...)` and start playing.
5. Clear `s_CurStoryData = null` on end.

### Condition Check (`StoryData.CheckCondition`)
*   Empty `m_Conditions` → always returns true.
*   Non-empty → `CheckConditions_OR` (OR relation).

## Caveats

*   **`StartStateName = "Start"` is a magic string**: the start event key is hardcoded to this; other names won't be treated as start.
*   **Repeat-trigger flag**: `CanTriggerRepeatedly` defaults to true (opposite of MapEventData); stories usually allow repetition.
*   **`s_CurStoryData` is global static**: external code can query the current story during execution; **but stories can't run in parallel** (would overwrite each other).
*   **`m_EventDic` auto-fills empty entries**: `GetSubStory(state)` adds an empty `RCG_OptionEventData` if the key is missing — so callers don't NRE, but you'll see an empty story.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_StoryData.cs`
*   **Inherits**: `RCG_Asset<RCG_StoryData>`
*   **Implements**: `UCLI_ShortName`
*   **AssetGroup**: `EditQuestSetting`
*   **Constants**: `StartStateName = "Start"`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_StoryData` | StoryData | `StoryData` (nested) | metadata: tags / conditions / random start |
| `m_EventDic` | SubStory | `Dictionary<string, RCG_OptionEventData>` | sub-story tree |

`StoryData` nested: `m_Tags` / `m_CanTriggerRepeatedly` / `m_Conditions` / `m_IsRandomStart` / `m_RandomStartStories`.

### A.3 Key Methods

*   **`StartStory(token, iSubStory, iIsRoot)`** — main trigger; async; includes start selection + DataService record + s_CurStoryData switching.
*   **`StartEvent` (property)** — `GetSubStory(StartStateName)`, start event shortcut.
*   **`GetSubStory(state)`** — fetches sub-story; auto-adds empty if missing.
*   **`StoryData.CheckCondition(data)`** — trigger condition OR check.
*   **`StoryData.GetStartSubStory(iSubStory)`** — start selection logic.
*   **`DefaultStory` (static)** — default fallback.

### A.4 System Interactions

*   **`RCG_OptionEventData`** — content of each SubStory (dialogue, options, results).
*   **`RCG_StoryDropPool`** — random pool source.
*   **`RCG_DataService.Ins.StartStory`** — runtime triggered story record.
*   **`RCG_StoryGenData`** — Asset Entry wrapper.
*   **`RCG_GameManager.Random`** — random start point picking.

### A.5 Known Issues

*   `s_CurStoryData` doesn't support concurrent stories; extreme cases need interrupt handling.
*   `// public class StoryState` and `m_OptionEventData` commented out, marking the legacy single-event structure → dictionary migration history.
*   `DeserializeFromJson`'s legacy migration logic (`m_EventDic[StartStateName] = m_OptionEventData`) commented out.
