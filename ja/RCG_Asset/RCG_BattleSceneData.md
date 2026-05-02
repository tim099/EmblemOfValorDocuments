---
title: 戦闘シーン設定 (RCG_BattleSceneData)
description: 戦闘背景：2D 画像 / 3D シーン / Prefab、地面有無、付随するフィールド効果池
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 戦闘シーン設定

> クラス名：`RCG_BattleSceneData`

## 用途

**戦闘シーンの背景とフィールド効果**。戦闘画面の見た目（2D 背景 / 3D 描画 / Prefab）と戦闘開始時に適用される可能性のあるフィールド効果を決定。各 `RCG_QuestData` / `RCG_BattleSet` が `RCG_BattleSceneData` を視覚背景として参照。

`RCG_Asset<RCG_BattleSceneData>` を継承。

## エディタ上の見た目

```
RCG_BattleSceneData: <ID>
    SceneType         ← Scene2D / Scene3D / ScenePrefab
    Image             ← 2D 背景画像（全モードで fallback として使用）
    3DScene           ← 3D シーン prefab（SceneType=Scene3D 時表示）
    FieldEffectDrops  ← 付随するフィールド効果ドロップ池
    HasGround         ← 地面の有無
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **SceneType** | はい | `Scene2D`（純背景画像）/ `Scene3D`（3D 描画背景）/ `ScenePrefab`（カスタム Prefab） |
| **Image** | はい | 背景画像（`RCG_SpriteData`）、デフォルト `BattleBackGrounds_Forest3.png` |
| **3DScene** | SceneType=Scene3D 時 | 3D シーン prefab；`Utility/3dBattleScenes` から読込 |
| **FieldEffectDrops** | いいえ | 突入時に抽選するフィールド効果池（`RCG_FieldEffectDropPool` 参照） |
| **HasGround** | — | 地面の有無（位置表示とパーティクル効果に影響） |

## 動作説明

### `SetBackGround(Image, RawImage, 3DSceneRoot)`
`m_SceneType` に応じて対応視覚を UI に挿入：
*   **Scene2D**：sprite を `iImage` に読込、Image 起動、RawImage 無効化。
*   **Scene3D**：3D scene を `i3DSceneRoot` に構築、Image / RawImage 無効化。
*   **ScenePrefab**：未実装（`switch` に対応 case なし）。

### フィールド効果発動
`OnTriggerEffect` は空実装；実発動ロジックは**`BattleManager.TriggerFieldEffect` に移動済み**。

## 注意事項

*   **ScenePrefab モードは未完成**：`SetBackGround` がこの case を処理しない、選択時は何も表示されない。
*   **`OnTriggerEffect` は空殻**：フィールド効果の実発動は BattleManager 側、ここはインターフェース placeholder のみ。
*   **`FieldEffects` / `Effects` の2フィールドはコメントアウト**（TODO マーク）：直接列挙を考慮していたが、現在は DropPool に統一。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_BattleSceneData.cs`
*   **継承**：`RCG_Asset<RCG_BattleSceneData>`
*   **AssetGroup**：`EditBattleSetting`
*   **定数**：`BattleScenePath = "Utility/3dBattleScenes"`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_SceneType` | SceneType | `SceneType` enum | `Scene2D` / `Scene3D` / `ScenePrefab` |
| `m_Image` | Image | `RCG_SpriteData` | デフォルト Forest 背景 |
| `m_3DScene` | 3DScene | `RCG_PrefabResData` | `Conditional(Scene3D)` |
| `m_FieldEffectDrops` | FieldEffectDrops | `RCG_FieldEffectDropPoolGenData` | |
| `m_HasGround` | HasGround | `bool` | デフォルト `true` |

### A.3 主要メソッド

*   **`Sprite` (property)** — `m_Image.Sprite`。
*   **`SetBackGround(token, Image, RawImage, 3DRoot)`** — SceneType に応じて背景設定。
*   **`OnTriggerEffect(triggerOn, data)`** — 空殻、実装は BattleManager。

### A.4 他システムとの連携

*   **`RCG_FieldEffectDropPool`** — フィールド効果ランダム池。
*   **`RCG_3dRenderScene`** — 3D シーンコンテナ。
*   **`RCG_BattleManager.TriggerFieldEffect`** — フィールド効果の真の実行場所。

### A.5 既知の問題

*   `ScenePrefab` モードの `SetBackGround` 対応 case 未実装。
*   `m_Effects` / `m_FieldEffects` コメントアウト済、「DropPool 使用後に削除」マーク。
