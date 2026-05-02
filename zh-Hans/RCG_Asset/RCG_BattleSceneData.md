---
title: 战斗场景设定 (RCG_BattleSceneData) 说明
description: 战斗背景：2D 图 / 3D 场景 / Prefab、是否有地面、附带的场地效果池
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 战斗场景设定

> 程式类别名称：`RCG_BattleSceneData`

## 用途

**战斗场景的背景与场地效果**。决定战斗画面看起来是什么样子（2D 背景图 / 3D 渲染 / Prefab）以及战斗开始时可能套用哪些场地效果。每个 `RCG_QuestData` / `RCG_BattleSet` 引用一个 `RCG_BattleSceneData` 作为视觉背景。

继承自 `RCG_Asset<RCG_BattleSceneData>`。

## 编辑器中的样貌

```
RCG_BattleSceneData: <ID>
    SceneType         ← Scene2D / Scene3D / ScenePrefab
    Image             ← 2D 背景图（所有模式都会用作 fallback）
    3DScene           ← 3D 场景 prefab（SceneType=Scene3D 时显示）
    FieldEffectDrops  ← 附带的场地效果掉落池
    HasGround         ← 是否有地面
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **SceneType** | 是 | `Scene2D`（纯背景图）/ `Scene3D`（3D 渲染作背景）/ `ScenePrefab`（自订 Prefab） |
| **Image** | 是 | 背景图（`RCG_SpriteData`），预设 `BattleBackGrounds_Forest3.png` |
| **3DScene** | SceneType=Scene3D | 3D 场景 prefab；从 `Utility/3dBattleScenes` 载入 |
| **FieldEffectDrops** | 否 | 进场时抽取的场地效果池（引用 `RCG_FieldEffectDropPool`） |
| **HasGround** | — | 是否有地面（影响站位显示与粒子效果） |

## 行为说明

### `SetBackGround(Image, RawImage, 3DSceneRoot)`
依 `m_SceneType` 把对应的视觉塞到 UI：
*   **Scene2D**：载 sprite 到 `iImage`，启动 Image，关 RawImage。
*   **Scene3D**：建立 3D scene 到 `i3DSceneRoot`，关 Image / RawImage。
*   **ScenePrefab**：未实作（`switch` 没有对应 case）。

### 场地效果触发
`OnTriggerEffect` 是空实作；实际触发逻辑**已搬到 `BattleManager.TriggerFieldEffect`**。

## 注意事项

*   **ScenePrefab 模式未完成**：`SetBackGround` 没处理此 case，选了会什么都不显示。
*   **`OnTriggerEffect` 是空壳**：场地效果的实际触发在 BattleManager，这里只是介面留位。
*   **`FieldEffects` / `Effects` 两栏已注解**（标 TODO）：曾经考虑过直接列场地效果，目前统一走 DropPool。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_BattleSceneData.cs`
*   **继承自**：`RCG_Asset<RCG_BattleSceneData>`
*   **AssetGroup**：`EditBattleSetting`
*   **常数**：`BattleScenePath = "Utility/3dBattleScenes"`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_SceneType` | SceneType | `SceneType` enum | `Scene2D` / `Scene3D` / `ScenePrefab` |
| `m_Image` | Image | `RCG_SpriteData` | 预设 Forest 背景 |
| `m_3DScene` | 3DScene | `RCG_PrefabResData` | `Conditional(Scene3D)` |
| `m_FieldEffectDrops` | FieldEffectDrops | `RCG_FieldEffectDropPoolGenData` | |
| `m_HasGround` | HasGround | `bool` | 预设 `true` |

### A.3 重要 Method 摘要

*   **`Sprite` (property)** — `m_Image.Sprite`。
*   **`SetBackGround(token, Image, RawImage, 3DRoot)`** — 依 SceneType 设置背景。
*   **`OnTriggerEffect(triggerOn, data)`** — 空壳，实际在 BattleManager。

### A.4 与其他系统的互动

*   **`RCG_FieldEffectDropPool`** — 场地效果随机池。
*   **`RCG_3dRenderScene`** — 3D 场景容器。
*   **`RCG_BattleManager.TriggerFieldEffect`** — 真正执行场地效果的地方。

### A.5 已知议题

*   `ScenePrefab` 模式未实作 `SetBackGround` 对应 case。
*   `m_Effects` / `m_FieldEffects` 已注解，标示「使用 DropPool 后移除」。
