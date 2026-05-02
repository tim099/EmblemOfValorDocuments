---
title: 戰鬥場景設定 (RCG_BattleSceneData) 說明
description: 戰鬥背景：2D 圖 / 3D 場景 / Prefab、是否有地面、附帶的場地效果池
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-en
---

> [!WARNING]
> Translation pending — this file needs an English translation.
The original zh-Hant content is included below for reference.


# 戰鬥場景設定

> 程式類別名稱：`RCG_BattleSceneData`

## 用途

**戰鬥場景的背景與場地效果**。決定戰鬥畫面看起來是什麼樣子（2D 背景圖 / 3D 渲染 / Prefab）以及戰鬥開始時可能套用哪些場地效果。每個 `RCG_QuestData` / `RCG_BattleSet` 引用一個 `RCG_BattleSceneData` 作為視覺背景。

繼承自 `RCG_Asset<RCG_BattleSceneData>`。

## 編輯器中的樣貌

```
RCG_BattleSceneData: <ID>
    SceneType         ← Scene2D / Scene3D / ScenePrefab
    Image             ← 2D 背景圖（所有模式都會用作 fallback）
    3DScene           ← 3D 場景 prefab（SceneType=Scene3D 時顯示）
    FieldEffectDrops  ← 附帶的場地效果掉落池
    HasGround         ← 是否有地面
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **SceneType** | 是 | `Scene2D`（純背景圖）/ `Scene3D`（3D 渲染作背景）/ `ScenePrefab`（自訂 Prefab） |
| **Image** | 是 | 背景圖（`RCG_SpriteData`），預設 `BattleBackGrounds_Forest3.png` |
| **3DScene** | SceneType=Scene3D | 3D 場景 prefab；從 `Utility/3dBattleScenes` 載入 |
| **FieldEffectDrops** | 否 | 進場時抽取的場地效果池（引用 `RCG_FieldEffectDropPool`） |
| **HasGround** | — | 是否有地面（影響站位顯示與粒子效果） |

## 行為說明

### `SetBackGround(Image, RawImage, 3DSceneRoot)`
依 `m_SceneType` 把對應的視覺塞到 UI：
*   **Scene2D**：載 sprite 到 `iImage`，啟動 Image，關 RawImage。
*   **Scene3D**：建立 3D scene 到 `i3DSceneRoot`，關 Image / RawImage。
*   **ScenePrefab**：未實作（`switch` 沒有對應 case）。

### 場地效果觸發
`OnTriggerEffect` 是空實作；實際觸發邏輯**已搬到 `BattleManager.TriggerFieldEffect`**。

## 注意事項

*   **ScenePrefab 模式未完成**：`SetBackGround` 沒處理此 case，選了會什麼都不顯示。
*   **`OnTriggerEffect` 是空殼**：場地效果的實際觸發在 BattleManager，這裡只是介面留位。
*   **`FieldEffects` / `Effects` 兩欄已註解**（標 TODO）：曾經考慮過直接列場地效果，目前統一走 DropPool。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_BattleSceneData.cs`
*   **繼承自**：`RCG_Asset<RCG_BattleSceneData>`
*   **AssetGroup**：`EditBattleSetting`
*   **常數**：`BattleScenePath = "Utility/3dBattleScenes"`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_SceneType` | SceneType | `SceneType` enum | `Scene2D` / `Scene3D` / `ScenePrefab` |
| `m_Image` | Image | `RCG_SpriteData` | 預設 Forest 背景 |
| `m_3DScene` | 3DScene | `RCG_PrefabResData` | `Conditional(Scene3D)` |
| `m_FieldEffectDrops` | FieldEffectDrops | `RCG_FieldEffectDropPoolGenData` | |
| `m_HasGround` | HasGround | `bool` | 預設 `true` |

### A.3 重要 Method 摘要

*   **`Sprite` (property)** — `m_Image.Sprite`。
*   **`SetBackGround(token, Image, RawImage, 3DRoot)`** — 依 SceneType 設置背景。
*   **`OnTriggerEffect(triggerOn, data)`** — 空殼，實際在 BattleManager。

### A.4 與其他系統的互動

*   **`RCG_FieldEffectDropPool`** — 場地效果隨機池。
*   **`RCG_3dRenderScene`** — 3D 場景容器。
*   **`RCG_BattleManager.TriggerFieldEffect`** — 真正執行場地效果的地方。

### A.5 已知議題

*   `ScenePrefab` 模式未實作 `SetBackGround` 對應 case。
*   `m_Effects` / `m_FieldEffects` 已註解，標示「使用 DropPool 後移除」。