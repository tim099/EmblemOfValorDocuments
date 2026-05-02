---
title: Background Music (RCG_BGMData)
description: BGM settings — audio file, volume; supports PlayBGM (replace) and PushBGM (stack) modes
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Background Music

> Class name: `RCG_BGMData`

## Purpose

**BGM template**. Unlike one-shot SE (`RCG_SEData`), BGM is continuously playing background music; map, battle, and menu have their own BGMs. This data defines the BGM's audio file, volume, and memo.

Inherits from `RCG_Asset<RCG_BGMData>`.

## Editor Layout

```
RCG_BGMData: <ID>
    BGM        ← audio file (RCG_AudioData; default GroupBGM)
    Volume     ← volume
    Note       ← memo
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **BGM** | yes | Audio file; default `BGM_MapBGM` |
| **Volume** | yes | Volume 0~1 (slider), default 1.0 |
| **Note** | no | Memo (for editor use) |

## Behavior

### `PlayBGM()` vs `PushBGM()`
*   **PlayBGM**: replaces the current BGM (stops old, plays new).
*   **PushBGM**: stacks onto the BGM stack (commonly used for "play battle BGM at battle start, pop back to map BGM at battle end").
Both have async versions (`PlayBGMAsync` / `PushBGMAsync`).

### `m_BGM.IsEmpty` Silently Returns
No LogError; only logs an error when the clip load fails (IsEmpty=false but GetClip returns null).

### Default ID
`RCG_BGMData.DefaultBGM = "BGM_MapBGM"`: default map BGM.

## Caveats

*   **No corresponding PopBGM in this file**: pop logic is handled by `UCL_GameAudioService` directly (caller invokes `PopBGM`).
*   **BGM and SE go through different mixers**: independent volume controls; wrong path means wrong category.
*   **No `Preview` override**: uses base default rendering; no waveform preview in editor.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_BGMData.cs`
*   **Inherits**: `RCG_Asset<RCG_BGMData>`
*   **AssetGroup**: `EditGameSetting`
*   **Constants**: `DefaultBGM = "BGM_MapBGM"`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_BGM` | BGM | `RCG_AudioData` | Default `ModResource + GroupBGM + DefaultBGM` |
| `m_Volume` | Volume | `float` | `[UCL_Slider(0, 1)]` |
| `m_Note` | Note | `string` | |

### A.3 Key Methods

*   **`PlayBGM() / PlayBGMAsync()`** — replace current BGM.
*   **`PushBGM() / PushBGMAsync()`** — stack play.

### A.4 System Interactions

*   **`UCL.Core.Game.UCL_GameAudioService`** — actual playback service (`PlayBGM` / `PushBGM`).
*   **`RCG_AudioData`** — audio file resource wrapper.
*   **`RCG_BGMGenData`** — Asset Entry.
