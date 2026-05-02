---
title: フィールド効果 (RCG_FieldEffectData)
description: 戦場全体に作用する環境効果テンプレート（火炎フィールド、暗闇覆い、毎ターン -HP 等）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# フィールド効果

> クラス名：`RCG_FieldEffectData`

## 用途

**戦場全体に作用する環境効果テンプレート**。状態 (CustomStatus) のようにユニットに貼られるのではなく、フィールド効果は**戦場全体共通**——例：「火炎フィールド：全ユニット毎ターン -3 HP」「暗闇：全攻撃命中率 -20%」「神聖：全治療 +50%」。

`RCG_Asset<RCG_FieldEffectData>` を継承。実装：`RCGI_Infos` / `UI.RCGI_StatusInfo`。

## エディタ上の見た目

```
RCG_FieldEffectData: <ID>
    Name(多言語)
    Icon                 ← フィールド効果アイコン
    Effects              ← 発動効果（trigger に応じて全場に適用）
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Name** | はい | フィールド効果名（多言語） |
| **Icon** | はい | アイコン、デフォルト `FieldEffectIcons_volcano.png` |
| **Effects** | はい | 各 trigger で発動する効果（`RCG_CommonEffect` リスト） |

## 動作説明

### 発動
`OnUnitState(triggerOn, data)` → `m_Effects` から該 trigger の effects を取得し順次発動。**`OnUnitState` 命名は歴史的経緯**——実際フィールド効果は必ずしも unit state 紐付けでなく、任意の trigger で発動可能（OnBattleStart / OnTurnStart / OnTurnEnd 等）。

### 説明
`Effects` から自動連結（BattleTags 解説含む）。

## 注意事項

*   **層数メカニズムなし**：フィールド効果は「ある / なし」の二元状態、CustomStatus のような層数なし。
*   **複数フィールド効果**は積層可（BattleManager が現在有効リストを管理）；UI に全アクティブフィールドを列挙。
*   **抽選元**：`RCG_BattleSceneData.m_FieldEffectDrops` が `RCG_FieldEffectDropPool` を参照、戦闘開始時にこの戦闘のフィールド効果を抽選。
*   **RCGI_Status 介面未実装**：クラス宣言に `//, RCGI_Status` のコメントあり、過去計画されたが未実装を示す。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_FieldEffectData.cs`
*   **継承**：`RCG_Asset<RCG_FieldEffectData>`
*   **実装**：`RCGI_Infos` / `UI.RCGI_StatusInfo`
*   **AssetGroup**：`EditBattleSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Icon` | Icon | `RCG_SpriteData` | デフォルト `FieldEffectIcons_volcano.png` |
| `m_Effects` | Effects | `List<RCG_CommonEffect>` | |

### A.3 主要メソッド

*   **`OnUnitState(triggerOn, data)`** — 効果発動（命名は歴史的経緯）。
*   **`TriggerOnUnitState(triggerOn)`** — quick check。
*   **`Description` (property)** — `m_Effects.GetDescription(showBattleTags = true)`。
*   **`Infos` (property)** — `m_Effects.GetInfos()`。

### A.4 他システムとの連携

*   **`RCG_BattleSceneData.m_FieldEffectDrops`** — 戦闘シーン参照のフィールド効果池。
*   **`RCG_FieldEffectDropPool`** — フィールド効果ランダム池。
*   **`RCG_BattleManager.TriggerFieldEffect`** — runtime でフィールド効果を発動する入口。
*   **`RCG_FieldEffectGenData`** / **`RCG_FieldEffectDropPoolGenData`** — Asset Entry ラッパー。

### A.5 既知の問題

*   クラス宣言の `, RCGI_Status` はコメントアウト——過去計画されたが未実装の介面。
*   `m_AcquireSkillEvents` 等プログラム内コメントアウト、UnitSkill からコピー時の dead code 残存と疑われる。
