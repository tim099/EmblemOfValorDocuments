---
title: Dialog
description: Show a character dialog bubble (text + duration + shake + audio); pure visual
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Dialog

> Class: `RCG_DialogSetting`

## Purpose
**Show a character dialog bubble**. Pure visual — no gameplay impact. Examples:
*   Enemy taunt before an attack
*   Story moment during a special skill
*   Boss-fight in-battle dialogue

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **Dialog** | Yes | Dialog content (`RCG_LocalizeData`, multi-language). |
| **Duration** | Yes | Display duration (seconds); default 0.8. |
| **FontSize** | — | Text size; default 40. |
| **Shake** | — | Whether to shake the bubble (emphasis). |
| **PlayAudio** | — | Whether to play SFX. |
| **Audio** | When PlayAudio | The SFX (`RCG_SEGenData`); auto-hidden when PlayAudio off. |
| **HideDescription** | — | If checked, dialog text **doesn't appear in card description** (background flavor only). |

## Behaviour
*   Spawns `VFX_Dialog` and calls `vfx.SetDialog(this, iData.User)` above the speaker.
*   Description shows the dialog text (unless `HideDescription`).
*   **Short description is always empty** (won't appear in enemy intent).

## Notes
*   **HideDescription is for "ambience-only dialogue"**: e.g. boss flavor lines that shouldn't clutter card text.
*   **Duration too short**: under 0.5s players can't read; longer text needs longer duration.
*   **PlayAudio without specifying Audio**: tries to play empty `RCG_SEGenData`, may log a warning.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_DialogSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_DialogSetting` → "Dialog"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_Dialog` | Dialog | `RCG_LocalizeData` | — | |
| `m_Duration` | Duration | `float` | — | Default 0.8f |
| `m_FontSize` | FontSize | `int` | — | Default 40 |
| `m_Shake` | Shake | `bool` | — | |
| `m_PlayAudio` | PlayAudio | `bool` | — | |
| `m_Audio` | Audio | `RCG_SEGenData` | — | `[Conditional(nameof(m_PlayAudio), false, true)]` |
| `m_HideDescription` | HideDescription | `bool` | — | |

### A.3 Key methods
*   **`AddAction`** (async): `RCG_VFXManager.CreateVFX(VFX_Dialog) as RCG_VFX_Dialog` → `vfx.SetDialog(this, iData.User)`.
*   **`GetDescriptionFormat`**: `m_HideDescription ? string.Empty : m_Dialog.Name`.
*   **`GetDescriptionShort`** → always empty.

### A.4 Cross-system interactions
*   **`CommonVFX.VFX_Dialog` / `RCG_VFX_Dialog.SetDialog`**: dialog UI entry.
