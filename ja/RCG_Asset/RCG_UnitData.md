---
title: ユニットデータ (RCG_UnitData)
description: 戦場上の全ユニット（モンスター、プレイヤーキャラ、召喚獣）の本体データ。HP、外観、AI、スキル全てここで定義
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# ユニットデータ

> クラス名：`RCG_UnitData`

## 用途

**戦場上の各ユニットの本体テンプレート** — モンスター、プレイヤーキャラ、召喚獣はすべてこのクラスのインスタンス。含む内容：HP / 戦闘力、外観（画像、Spine、ドラゴンボーン、3D Prefab）、AI 行動（状態ごとに使用するスキル）、初期動作、必要なユニット設定など。旧 `RCG_MonsterData` を置き換え。

`RCG_Asset<RCG_UnitData>` を継承。

## エディタ上の見た目

```
RCG_UnitData: <ID>
    UnitSetting (m_UnitSetting)        ← 主要設定：HP、タグ、初期Action、外観、配置
    MonsterStates (m_MonsterStates)    ← AIステートマシン：各状態で使用可能なAction
    Preview (右側)                      ← 外観、スキル一覧、HPをリアルタイム表示
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **UnitSetting** | はい | ユニットのコア設定（HP / タグ / 初期 Action / 外観 / 配置など） |
| **MonsterStates** | はい | AI ステートマシン：dictionary、key は状態 ID、value は当該状態下で使用可能な `MonsterAction` 一覧。**最低でも `Default` 状態が必要**（システムが自動補完） |

`UnitSetting` の内訳：

| サブフィールド | 説明 |
|---|---|
| **Name** | ユニット表示名（多言語） |
| **MaxHP** | 最大生命値 |
| **CombatEffectiveness** | 戦闘力指標；シリアライズ時に 0 なら自動的に `MaxHP` で補完 |
| **MonsterTags** | モンスタータグ（種族、属性など）、特定のカード/装備効果に影響 |
| **InitActions** | 出現時に発動する初期動作（バフ / デバフ / 召喚） |
| **DetailSetting** | 詳細設定（攻撃回数、カウンター、種類により異なる） |
| **SummonSetting** | 召喚時の設定 |
| **UnitDisplayerType** | 表示方式：Sprite / DragonBone / Spine / 3D Model / Prefab |
| **SpriteDisplayData / DragonBoneDisplayData / ...** | 各表示器対応データ（Conditional） |
| **Pos / PosAt** | 配置オフセット、座標 |
| **BaseLevel / UnitGenData** | 基礎レベルとユニット ID ラッパー |
| **HasAI / Classes / UnitSkills** | AI スイッチ、職業（カード専門性用）、所持ユニットスキル |

## 動作説明

### AI ステートマシン (MonsterStates)
各モンスターは複数の「状態」（例：`Default`、`Berserk`、`Phase2`）を持ち、各状態に使用可能な `MonsterAction` セットがある。実戦闘時は `MonsterStateTransition` が状態切替を判定、Action 選択ルールが当該ターンの使用招式を決定。

### 表示方式
*   **Sprite**：静止画（最も簡単、placeholder 向き）。
*   **DragonBone / Spine**：2D 骨格アニメ。
*   **Model3DDisplayer / Prebab**：3D モデル / カスタム Prefab。
`UnitDisplayerType` を切り替えて初めて対応フィールドが表示される。

### 初期動作 (InitActions)
出現時に一度だけ自動適用される動作。一般的用途：自身バフ付与、従者召喚、カウンター初期値設定。

### プレビュー（エディタ内）
右側にリアルタイム表示：アバター、最大 HP、モンスタータグ、初期動作の説明、各状態下のスキルアイコンと名称。**Low RAM モード**ではアバター描画をスキップしメモリ節約。

## 注意事項

*   **`Default` 状態は必須**：`OnGUI` で自動補完されるが、保険のため事前設定推奨。
*   **`CombatEffectiveness` の fallback**：`SerializeToJson` 時に該値が 0 なら `MaxHP` で補完；0 を保持したい場合は特殊処理が必要。
*   **`NullMonsterID = "Null"`、`DefaultUnitID = "Devil"`、`BattleSceneMonsterID = "BattleScene"`**：これらはシステム予約 ID、一般モンスターの命名に使用しない。
*   **編集画面の Skill / Action / SummonSetting** は各々 `RCG_MonsterActionData` / `RCG_UnitSkillData` 等の Asset から参照、本ファイルには ID ラッパーのみ保持。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_Battles/RCG_Monsters/RCG_UnitData.cs`
*   **継承**：`RCG_Asset<RCG_UnitData>`
*   **AssetGroup**：`EditBattleSetting`、sort = `RCG_UnitData`

### A.2 フィールドマッピング（外層）

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_UnitSetting` | UnitSetting | `RCG_UnitSetting`（入れ子） | 主要コアデータ |
| `m_MonsterStates` | MonsterStates | `Dictionary<string, MonsterState>` | AI ステートマシン；key は状態 ID |

`UnitSetting` 内に `m_Name` / `m_MaxHP` / `m_CombatEffectiveness` / `m_MonsterTags` / `m_InitActions` / `m_DetailSetting` / `m_SummonSetting` / `m_UnitDisplayerType` + 各 DisplayData / `m_Pos` / `m_PosAt` / `m_BaseLevel` / `m_UnitGenData` / `m_HasAI` / `m_Classes` / `m_UnitSkills`。

### A.3 主要メソッド

*   **`Preview` / `OnGUI`** — エディタ描画；OnGUI が `Default` MonsterState を自動補完。
*   **`SerializeToJson`** — `m_CombatEffectiveness == 0` の fallback を補う。
*   **`Avatar` / `GetAvatar(RCG_BattleUnit)`** — 表示器画像取得。
*   **`CreateSelectAssetPage`** — `RCG_UnitDataEditorPage` 起動。
*   **`LocalizeName`** — `GetData().LocalizedName`。

### A.4 他システムとの連携

*   **`RCG_BattleUnit`** — runtime 戦闘ユニットインスタンス。
*   **`MonsterState` / `MonsterStateTransition`** — ステートマシン要素（`DefaultStateID` を含む）。
*   **`RCG_MonsterAction` / `RCG_MonsterActionData`** — モンスター招式定義。
*   **`RCG_UnitSkillData`** — ユニットパッシブ / リーダースキル。
*   **`RCG_UnitDataEditorPage`** — メイン編集画面。

### A.5 既知の問題

*   `DeserializeFromJson` にコメントアウトされた `m_UnitSetting = m_MonsterData` の互換シム（旧フィールド移行）あり。
*   `// 暫時用HP當作戰鬥力指標` / 「暫定的に HP を戦闘力指標として使用」コメントは `m_CombatEffectiveness` の fallback ルールが再設計待ちであることを示す。
