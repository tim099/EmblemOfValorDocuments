---
title: Sound Effect (RCG_SEData)
description: Single sound effect setting — audio file, volume, AudioType, optional wait-until-end
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Sound Effect

> Class name: `RCG_SEData`

## Purpose

**Single sound effect (SE) setting**. Examples: "card play sound", "hit sound", "button click sound", "victory fanfare". Each `RCG_SEData` is one audio file + volume + type, can be played fire-and-forget or awaited until end.

Inherits from `RCG_Asset<RCG_SEData>`.

## Editor Layout

```
RCG_SEData: <ID>
    SE          ← audio file (RCG_AudioData)
    Volume      ← volume (0~1, slider)
    AudioType   ← SE type (SE / UI, etc.)
    Note        ← memo
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **SE** | yes | Audio file (`RCG_AudioData`); default loaded via `ModResource` |
| **Volume** | yes | Volume 0~1 (slider), default 1.0 |
| **AudioType** | yes | SE type (`UCL.Core.Game.AudioType`); default `SE` |
| **Note** | no | Memo (for editor use only) |

## Behavior

### `PlaySE()`
Non-blocking: `PlaySEAsync().Forget()` returns immediately, doesn't wait for the SE to finish.

### `PlaySEAsync()` (UniTask)
async fetches clip → `UCL_GameAudioService.Ins.Play(clip, audioType, volume)`, returns an `AudioPlayer`.

### `PlaySEUntilEnd(token)`
After getting the player, sets an `EndAct` flag → `WaitUntil` it ends; usable for "must finish SE before continuing" scenarios.

### Default ID
`RCG_SEGenData.DefaultID = "Null"`: meaning "no SE"; `s_Victory = "SoundEffect_Victory"` is the built-in victory SE.

## Caveats

*   **`m_SE.IsEmpty` doesn't LogError**: silently returns null; manually verify SE played.
*   **`PlaySE()` is non-blocking**: use `PlaySEUntilEnd(token)` to wait for completion.
*   **AudioType affects mixer routing**: UI / SE / BGM each have independent volume controls; wrong type means wrong volume control.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_SEData.cs`
*   **Inherits**: `RCG_Asset<RCG_SEData>`
*   **AssetGroup**: `EditGameSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_SE` | SE | `RCG_AudioData` | Default `ModResource + GroupSoundEffect` |
| `m_Volume` | Volume | `float` | `[UCL_Slider(0, 1)]`, default 1.0 |
| `m_AudioType` | AudioType | `UCL.Core.Game.AudioType` | Default `SE` |
| `m_Note` | Note | `string` | |

### A.3 Key Methods

*   **`PlaySE()`** — fire-and-forget.
*   **`PlaySEAsync()`** — fetches clip and plays, returns `AudioPlayer`.
*   **`PlaySEUntilEnd(token)`** — wait for completion.

### A.4 System Interactions

*   **`UCL.Core.Game.UCL_GameAudioService`** — actual playback service.
*   **`RCG_AudioData`** — audio file resource wrapper.
*   **`RCG_SEGenData`** — Asset Entry; default ID `Null`, with `s_Victory` built in.
