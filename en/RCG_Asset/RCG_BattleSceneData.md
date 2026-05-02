---
title: Battle Scene Data (RCG_BattleSceneData)
description: Battle background — 2D image / 3D scene / Prefab; ground toggle; attached field effect pool
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Battle Scene Data

> Class name: `RCG_BattleSceneData`

## Purpose

**Battle scene background and field effects**. Determines what the battle screen looks like (2D background image / 3D rendered / Prefab) and which field effects might apply at battle start. Each `RCG_QuestData` / `RCG_BattleSet` references an `RCG_BattleSceneData` as its visual background.

Inherits from `RCG_Asset<RCG_BattleSceneData>`.

## Editor Layout

```
RCG_BattleSceneData: <ID>
    SceneType         ← Scene2D / Scene3D / ScenePrefab
    Image             ← 2D background image (used as fallback by all modes)
    3DScene           ← 3D scene prefab (shown when SceneType=Scene3D)
    FieldEffectDrops  ← attached field effect drop pool
    HasGround         ← whether there's a ground
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **SceneType** | yes | `Scene2D` (image background) / `Scene3D` (3D rendered as background) / `ScenePrefab` (custom Prefab) |
| **Image** | yes | Background image (`RCG_SpriteData`); default `BattleBackGrounds_Forest3.png` |
| **3DScene** | when SceneType=Scene3D | 3D scene prefab; loaded from `Utility/3dBattleScenes` |
| **FieldEffectDrops** | no | Field effect pool drawn on entry (references `RCG_FieldEffectDropPool`) |
| **HasGround** | — | Whether there's a ground (affects positioning display and particles) |

## Behavior

### `SetBackGround(Image, RawImage, 3DSceneRoot)`
Per `m_SceneType`, sets the corresponding visual into UI:
*   **Scene2D**: load sprite to `iImage`, enable Image, disable RawImage.
*   **Scene3D**: build 3D scene under `i3DSceneRoot`, disable Image / RawImage.
*   **ScenePrefab**: not implemented (`switch` has no corresponding case).

### Field Effect Triggering
`OnTriggerEffect` is empty; the actual triggering logic **has been moved to `BattleManager.TriggerFieldEffect`**.

## Caveats

*   **ScenePrefab mode incomplete**: `SetBackGround` doesn't handle this case; selecting it shows nothing.
*   **`OnTriggerEffect` is empty**: actual field effect triggering happens in BattleManager; this is a placeholder.
*   **`FieldEffects` / `Effects` fields are commented out** (TODO marker): used to support direct listing; now unified through DropPool.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_BattleSceneData.cs`
*   **Inherits**: `RCG_Asset<RCG_BattleSceneData>`
*   **AssetGroup**: `EditBattleSetting`
*   **Constants**: `BattleScenePath = "Utility/3dBattleScenes"`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_SceneType` | SceneType | `SceneType` enum | `Scene2D` / `Scene3D` / `ScenePrefab` |
| `m_Image` | Image | `RCG_SpriteData` | Default Forest background |
| `m_3DScene` | 3DScene | `RCG_PrefabResData` | `Conditional(Scene3D)` |
| `m_FieldEffectDrops` | FieldEffectDrops | `RCG_FieldEffectDropPoolGenData` | |
| `m_HasGround` | HasGround | `bool` | Default `true` |

### A.3 Key Methods

*   **`Sprite` (property)** — `m_Image.Sprite`.
*   **`SetBackGround(token, Image, RawImage, 3DRoot)`** — set background per SceneType.
*   **`OnTriggerEffect(triggerOn, data)`** — empty placeholder; logic in BattleManager.

### A.4 System Interactions

*   **`RCG_FieldEffectDropPool`** — random field effect pool.
*   **`RCG_3dRenderScene`** — 3D scene container.
*   **`RCG_BattleManager.TriggerFieldEffect`** — where field effects actually run.

### A.5 Known Issues

*   `ScenePrefab` mode unimplemented in `SetBackGround`.
*   `m_Effects` / `m_FieldEffects` commented out, marked "remove after using DropPool".
