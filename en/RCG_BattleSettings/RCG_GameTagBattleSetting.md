---
title: Game tag (battle)
description: Triggers a game-level tag event (achievements, player events); thin wrapper layer
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Game tag (battle)

> Class: `RCG_GameTagBattleSetting`

## Purpose
**Triggers game-level tag events from within battle** — a thin wrapper that bridges `RCG_GameTagSetting` to the battle action queue. Examples:
*   Trigger achievement conditions (e.g. "defeated first boss")
*   Log player-action telemetry

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **GameTagSetting** | Yes | Embedded `RCG_GameTagSetting` — the actual game-tag setting. |

## Behaviour
*   On trigger → `m_GameTagSetting.Trigger(iData)`.
*   Description and card info delegate to `m_GameTagSetting`.

## Notes
*   **This setting is just a wrapper**: actual logic lives in `RCG_GameTagSetting` — see that class for fields and config.
*   **No gameplay impact**: pure event reporting / logging.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_GameTagBattleSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_GameTagBattleSetting` → "Game tag"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_GameTagSetting` | GameTagSetting | `RCG_GameTagSetting` | — | Wrapper's only field |

### A.3 Key methods
*   **`AddAction`**: `m_GameTagSetting.Trigger(iData)`.
*   **`Info / GetDescriptionParams / GetDescriptionFormat`**: delegate to `m_GameTagSetting`.

### A.4 Cross-system interactions
*   **`RCG_GameTagSetting`**: actual implementation; this class just adapts it to BattleSetting.
