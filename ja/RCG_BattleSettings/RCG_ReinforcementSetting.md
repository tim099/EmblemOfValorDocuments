---
title: 増援
description: 増援ユニットを召喚（現在は敵側のみ）；通常戦はランダム抽出、精英 / Boss 戦はデフォルトテンプレート使用
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 増援

> クラス名：`RCG_ReinforcementSetting`

## 用途
**敵側の増援ユニットを戦場に召喚**。現在は**モンスター増援のみ対応**（プレイヤー側には使用不可）。よくある用途：
*   ボスが雑兵を召喚
*   特定ターンで敵を自動補充

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **DefaultUnit** | はい | デフォルト増援ユニット（**精英 / 魔王戦専用**）。通常戦闘ではこのフィールドは使われず、`MonsterSet` からランダム抽出。 |

## 挙動
*   `RCG_BattleField.Ins.MonsterSet` を読み、現在の敵タイプの利用可能ユニットリストを取得。
*   **通常戦闘**：リストからランダムに 1 体抽選。
*   **精英 / Boss 戦**：`DefaultUnit` を直接使用 — 過剰な精英の生成によるバランス崩れを回避。
*   `iData.TargetPositions` で指定された空き位置を優先；指定なしのときは自動的に空き位置を探す。
*   説明形式：「**モンスター増援を召喚**」（i18n key `ReinforcementDes_Monster`）。

## 注意点
*   **モンスター増援のみ**：プログラム内 `bool aIsMonster = true` がハードコードされている；味方友軍の召喚は「**召喚**」設定を使ってください。
*   **空き位置がない場合の挙動**：補完できないと黙ってスキップ — プレイヤーは「カードが無効？」と感じるかも。
*   **精英 / Boss のデフォルト制限**：「増援カード + 精英」が連鎖して大量の精英を召喚する事態を回避するため — この設計選択には `// TODO: 應該移到 Condition` のコメントあり、将来リファクタの可能性。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ReinforcementSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_ReinforcementSetting` → 「増援」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_DefaultUnit` | `DefaultUnit` | `RCG_UnitGenData` | — | 精英 / Boss 戦専用 |

### A.3 主なメソッド
*   **`LocalizeKey` (private)** → ハードコードで `"ReinforcementDes_Monster"` を返す（コメント：現在は敵増援のみ）。
*   **`AddAction`** (async)：
    1. `MonsterSet.GetUnitSpawnDatas(EnemyType)` からランダム抽出 → `aTargetUnit`。
    2. 敵タイプ ≠ Normal なら `m_DefaultUnit.GetData()` を使用。
    3. `iData.TargetPositions` から空き位置取得を試みる；失敗時は `RCG_BattleField` で自動分配（ファイルの 80 行外の続き）。

### A.4 他システムとの連携
*   **`RCG_BattleField.Ins.MonsterSet.GetUnitSpawnDatas`**：候補モンスターリストソース。
*   **`RCG_BattleManager.Ins.EnemyType`** / **`RCG_EnemyTypeTagGenData.s_EnemyType_Normal`**：戦闘タイプ判定。
*   **`UCL_Random.Instance.Next`**：候補選択のランダムソース。

### A.5 既知の課題
*   `// TODO: 目前只有普通戰鬥獲得增援 (應該移到Condition)` — 条件判断に抽出すべきで、増援ロジック内に直接書かない方が良い。
*   `bool aIsMonster = true` がハードコード、味方増援はできない。
