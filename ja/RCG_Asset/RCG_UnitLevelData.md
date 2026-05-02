---
title: ユニットレベルデータ (RCG_UnitLevelData)
description: 「ユニットの HP / 攻撃力がレベルに応じて成長する」曲線を定義。難易度システムと連動して敵強度に影響
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# ユニットレベルデータ

> クラス名：`RCG_UnitLevelData`

## 用途

「**ユニットの HP と攻撃力がレベルに応じてどう成長するか**」の曲線を定義。各モンスター / キャラが1つの `RCG_UnitLevelData` を参照可能；ゲーム中は**現難易度**でレベルに換算し、この曲線で当該レベルの HP 倍率と攻撃力倍率を決定。

`RCG_Asset<RCG_UnitLevelData>` を継承。

## エディタ上の見た目

```
RCG_UnitLevelData: <ID>
    LevelCurve  ← レベル曲線：難易度 0~1 → レベル成長値
    HPCurve     ← HP 曲線：レベル 0~1 → HP 倍率
    AtkCurve    ← 攻撃曲線：レベル 0~1 → 攻撃倍率
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **LevelCurve** | はい | レベル曲線。入力は `aDifficultyVal`(0~1)、出力は `BaseLevel` に加算するレベル値 |
| **HPCurve** | はい | HP 倍率曲線。入力はレベル値 (0~1)、出力は HP 倍率 |
| **AtkCurve** | はい | 攻撃倍率曲線。同上 |

各 `CurveData` の内部：
*   **CurveType**：`Linear`（線形 `Lerp(min, max, x)`）または `Power`（累乗 `pow(base, x * segment)`）
*   **MinVal / MaxVal**（Linear）：開始値と最大値
*   **PowerBase / PowerSegment**（Power）：累乗底数とセグメント数

## 動作説明

### レベル換算 (`GetLevel(BaseLevel)`)
1. **現難易度**を取得：`RCG_BigMapManager.Difficulty + RCG_DataService.Ins.Difficulty`。
2. 難易度を 0~1 範囲にマッピング（**区分関数**）：
    *   0~10 → 0.00~0.10（線形）
    *   10~50 → 0.10~0.30
    *   50~100 → 0.30~0.40
    *   100+ → 0.40~1.00
3. `LevelCurve.GetValue(0~1) → 加算値`、`Level = BaseLevel + round(加算値)`、[1, 100] にクランプ。

### 属性計算
*   `GetMaxHP(baseHP, level)` = `round(baseHP × HPCurve(level/99) × DifficultyData.HealthMult)`
*   `GetAtkMult(level)` = `AtkCurve(level/99) × DifficultyData.AtkMult`
注意：「グローバル難易度倍率」(`DifficultyData.HealthMult / AtkMult`) はこの層で再度乗じられる。

### プレビュー
エディタ内のデフォルト Preview は ID ラベルのみ表示、曲線視覚化は描画しない（曲線形状確認は数値で行うのみ）。

## 注意事項

*   **MaxLevel = 100**：上限がハードコード、超過時はクランプ。修正時は `m_PowerSegment` のデフォルト値も連動。
*   **MaxDifficulty = 100**（理論値）：実際は 100+ の難易度も 0.001 線形で成長続けるが、`aDifficulty > 1` 時に `aDifficulty` を 1 に設定する（**バグの可能性**：`aDifficultyVal` を設定すべき）。
*   **`Debug.LogWarning` が `GetMaxHP` / `GetAtkMult` 内**：毎回計算で log を出力、ログ氾濫；release 前に削除必須。
*   **グローバル難易度倍率**（`DifficultyData.HealthMult / AtkMult`）はこの層で再度乗じられる；総体強度 = レベル曲線 × グローバル倍率。
*   **`m_LevelCurve` のデフォルト `Linear(0, 100)`** は「最大難易度時に +100 レベル」を意味、`BaseLevel = 1` と組合せると MaxLevel に到達。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_UnitLevelData.cs`
*   **継承**：`RCG_Asset<RCG_UnitLevelData>`
*   **AssetGroup**：`EditBattleSetting`
*   **定数**：`MinLevel = 1` / `MaxLevel = 100` / `MaxDifficulty = 100`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_LevelCurve` | LevelCurve | `CurveData` | デフォルト `Linear(0, 100)` |
| `m_HPCurve` | HPCurve | `CurveData` | デフォルト `Linear(1, 10)` |
| `m_AtkCurve` | AtkCurve | `CurveData` | デフォルト `Linear(1, 4)` |

### A.3 主要メソッド

*   **`Difficulty` (static property)** — `BigMapManager.Difficulty + DataService.Difficulty`。
*   **`GetLevel(BaseLevel)`** — 主入口；区分難易度→0~1 マッピング含む。
*   **`GetMaxHP(baseHP, level)` / `GetAtkMult(level)`** — 曲線適用 + グローバル難易度倍率。
*   **`GetLevelValue(level)`** (private) — `(level - 1) / (MaxLevel - 1f)`、1~100 を 0~1 にマッピング。

### A.4 他システムとの連携

*   **`RCG_BigMapManager.Difficulty`** — 大マップ層の難易度。
*   **`RCG_DataService.Ins.Difficulty`** — プレイ / チャレンジ runtime 難易度。
*   **`RCG_DataService.Ins.m_DifficultyData.m_HealthMult / m_AtkMult`** — グローバル難易度倍率。
*   **`RCG_UnitLevelGenData`**（同ファイル）— Asset Entry ラッパー、デフォルト ID = `"Default"`。

### A.5 既知の問題

*   `GetLevel` 内 `aDifficultyVal > 1` 時に `aDifficulty = 1` を変更（型不正、バグ疑い；`aDifficultyVal = 1` にすべき）。
*   `GetMaxHP` / `GetAtkMult` 内の `Debug.LogWarning` がログ氾濫。
