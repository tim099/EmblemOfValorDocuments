---
title: VFX
description: Pure-visual effect (no gameplay impact); supports VFXGenData and Timeline modes
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# VFX

> Class: `RCG_VFXSetting`

## Purpose
**Plays a pure visual effect**. No gameplay impact — just for show. Examples:
*   Pre-attack flash / charge-up aura
*   Special-skill scene VFX
*   Timeline-based audio + VFX sequences

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **VFXType** | Yes | Mode:<br>• **VFXGenData** — single VFX from `RCG_CommonVFXGenData` (default)<br>• **Timeline** — multiple SFX / VFX in sequence |
| **VFXGenData** | When VFXType=VFXGenData | VFX template. |
| **Settings** | When VFXType=Timeline | Timeline list (each item is a `VFXSetting` with time + content). |

## Behaviour
*   **VFXGenData mode**: directly `m_VFXGenData.GetData().PlayVFX(iData, token)`.
*   **Timeline mode**: each `VFXSetting.Play(token)` runs **in parallel**, all `await`'d to completion.
*   **Doesn't fuse**: `GetFusionBaseSetting` returns null.

## Notes
*   **Empty VFX**: `VFXGenData = null` doesn't crash but plays nothing.
*   **Timeline runs in parallel**: all entries fire concurrently, **not** sequentially. For strict ordering use multiple VFX settings inside a Combine.
*   **Battle pacing**: blocks subsequent actions until VFX completes — long VFX slows the fight.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_VFXSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_VFXSetting` → "VFX"
*   **Top-level enum**: `VFXType` (`VFXGenData`, `Timeline`)

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_VFXType` | VFXType | `VFXType` (file enum) | — | Default `VFXGenData` |
| `m_VFXGenData` | VFXGenData | `RCG_CommonVFXGenData` | — | `[Conditional(... VFXGenData)]` |
| `m_Settings` | Settings | `List<VFXSetting>` | — | `[Conditional(... Timeline)]` |

### A.3 Key methods
*   **`AddAction`** (async): branches on `m_VFXType`:
    *   `VFXGenData` → `await m_VFXGenData.GetData().PlayVFX(iData, iToken)`.
    *   `Timeline` → collects `setting.Play(iToken)` UniTasks, `UniTask.WhenAll(tasks)` parallel await.
*   **`GetFusionCandidateSettings`** → empty; **`GetFusionBaseSetting`** → null.

### A.4 Cross-system interactions
*   **`RCG_CommonVFXGenData / VFXSetting`**: VFX template + timeline entries.
*   **`PlayVFX(iData, token)`**: actual play entry.

### A.5 Known issues
*   Legacy `m_VFX` (`RCG_VFXResData`) and `m_VFXTime` superseded by VFXGenData; deserialization migration kept as comments.
