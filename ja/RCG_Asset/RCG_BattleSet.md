---
title: 戦闘構成 (RCG_BattleSet)
description: 1つの具体的戦闘のモンスター配置 + タグ + 敵タイプ。マップノード突入時にこのデータでバトル生成
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 戦闘構成

> クラス名：`RCG_BattleSet`

## 用途

**1つの具体的戦闘のモンスター配置**。例：「ゴブリン3 + ゴブリンシェフ1」「ボス クジラ単体」「精鋭遭遇：ドラゴン + 法師」が異なる BattleSet。マップノードが戦闘を起動する時、`RCG_BattleSetDropPool` から `RCG_BattleSet` を抽選してこの戦闘を生成。

`RCG_Asset<RCG_BattleSet>` を継承。

## エディタ上の見た目

```
RCG_BattleSet: <ID>
    EnemyType  ← 敵タイプ（通常 / 精鋭 / ボス）
    Tags       ← BattleSet タグ（DropPool 選別用）
    MonsterSet ← モンスター配置（位置、ID、報酬リソース）
    Preview    ← 配置のリアルタイムプレビュー
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **MonsterSet** | はい | 完全なモンスター配置（`RCG_MonsterSet`：各位置にどのモンスター、ドロップする報酬リソース） |
| **Tags** | いいえ | BattleSet タグ（章 / シーンタイプ等）、DropPool がタグでフィルタ |
| **EnemyType** | はい | 敵タイプ（`Normal` / `Elite` / `Boss`）、報酬、戦闘音楽、UI タイトルに影響 |

## 動作説明

### デフォルト取得
`RCG_BattleSet.DefaultBattleSet` で `RCG_BattleSetGenData.DefaultID` 対応データ直接取得；欠落時の fallback。

### プレビュー
ID + EnemyType、Tags 一覧、`MonsterSet.Preview` のモンスター配置可視化を表示。

## 注意事項

*   **EnemyType は Tag オブジェクトで enum ではない**：元は enum (`m_EnemyType = EnemyType.Normal`)、現在は `RCG_EnemyTypeTagGenData` に変更し旧フィールドはコメントアウト；新タイプ追加は Tag asset 編集で済む。
*   **MonsterSet 内の報酬リソース**：以前 `m_MonsterSet.m_RewardResources.Clear()` のデシリアライズクリーンアップがあった（コメントアウト済）；特殊クリーンアップが必要なら参考可。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSet.cs`
*   **継承**：`RCG_Asset<RCG_BattleSet>`
*   **AssetGroup**：`EditBattleSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_MonsterSet` | MonsterSet | `RCG_MonsterSet` | 主配置コンテナ |
| `m_Tags` | Tags | `List<RCG_BattleSetTagGenData>` | DropPool 用 |
| `m_EnemyType` | EnemyType | `RCG_EnemyTypeTagGenData` | 旧 enum を置換 |

### A.3 主要メソッド

*   **`Preview` / `OnGUI`** — エディタ描画。
*   **`DefaultBattleSet`** (static) — `Util.GetData(RCG_BattleSetGenData.DefaultID)`。

### A.4 他システムとの連携

*   **`RCG_MonsterSet`** — モンスター位置 + 報酬の詳細コンテナ。
*   **`RCG_BattleSetDropPool`** — 戦闘構成のランダム池。
*   **`RCG_BattleSetGenData`** — Asset Entry ラッパー。

### A.5 既知の問題

*   旧 `m_EnemyType` (enum) コメントアウト済、Tag 型移行履歴を示す。
