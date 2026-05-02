---
title: Map Event Data (RCG_MapEventData)
description: Bundle of events that can fire on small-map nodes — dialogue, cutscenes, granting items, battle preludes
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Map Event Data

> Class name: `RCG_MapEventData`

## Purpose

**Bundle of events that can fire on small-map nodes**. For example, stepping on a "mystery node" triggers dialogue → choice → reward; "campfire" opens a heal menu directly; "merchant" opens a shop. Each `RCG_MapEventData` is a container of `RCG_MapEvent`; on trigger, queues them in order to the event manager.

Inherits from `RCG_Asset<RCG_MapEventData>`.

## Editor Layout

```
RCG_MapEventData: <ID>
    Name                     ← event display name (localized)
    Icon                     ← custom icon (when blank, uses first Event's default icon)
    CanTriggerRepeatedly     ← whether repeatable
    Events                   ← actual event list (RCG_MapEvent)
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **Name** | no | Display name (localized); falls back to first Event's ShortName when empty |
| **Icon** | no | Icon; when empty, uses first Event's `DefaultIcon` |
| **CanTriggerRepeatedly** | — | Whether repeatable; false → after firing, clears the events on the node |
| **Events** | yes | Actual event list (`List<RCG_MapEvent>`) |

## Behavior

### Trigger (`StartEvent(node)`)
Clones each `RCG_MapEvent` and queues to `RCG_MapEventManager`. Cloning (rather than direct use) ensures each trigger gets an independent instance.

### Icon and Name Fallback
*   `Icon`: `m_Icon` empty → `m_Events[0].DefaultIcon` → null.
*   `LocalizedName`: if `m_Name` set → use it; else `m_Events[0].GetShortName()`.
*   `EventHoverTips`: same as `LocalizedName` (mouse-hover tooltip on node).

## Caveats

*   **`CanTriggerRepeatedly = false`** clears the node's events after firing: suits "one-time events" like story key points, unique rewards.
*   **Empty Events**: icon and name fallback fail (return null / empty); UI looks empty.
*   **Default ID `Start`**: typically used as the dedicated event for the start node.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_MapEventData.cs`
*   **Inherits**: `RCG_Asset<RCG_MapEventData>`
*   **AssetGroup**: `EditQuestSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Icon` | Icon | `RCG_SpriteData` | Addressable default |
| `m_CanTriggerRepeatedly` | CanTriggerRepeatedly | `bool` | Default `false` |
| `m_Events` | Events | `List<RCG_MapEvent>` | |

### A.3 Key Methods

*   **`StartEvent(RCG_MapNode)`** — main trigger entry; clones each event and queues.
*   **`Icon` (property)** — with fallback logic.
*   **`LocalizedName / EventHoverTips`** — display properties.

### A.4 System Interactions

*   **`RCG_MapEvent`** — actual event unit (dialogue / choice / reward / battle prelude).
*   **`RCG_MapEventManager`** — runtime event queue management.
*   **`RCG_MapNode`** — source trigger node.
*   **`RCG_MapEventGenData`** — Asset Entry; default ID = `"Start"`.
