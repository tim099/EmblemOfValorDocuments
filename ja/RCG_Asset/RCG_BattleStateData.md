---
title: 戦闘状態データ (RCG_BattleStateData)
description: 戦闘の「フェーズ」(state) 定義：当該状態突入時に発動する行動、次の状態
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 戦闘状態データ

> クラス名：`RCG_BattleStateData`

## 用途

**戦闘の「フェーズ (state)」定義** — 例：「プレイヤーターン」「敵ターン」「ダメージ集計」「ターン終了」。各 state 突入時に固定の動作を実行（カードドロー、buff 集計、ターンイベント発動等）。戦闘全体はこれらの state 間を遷移する。

`RCG_Asset<RCG_BattleStateData>` を継承。

## エディタ上の見た目

```
RCG_BattleStateData: <ID>
    Name                ← 表示名（多言語）
    NextState           ← 次の state の ID
    EnterStateActions   ← この state 突入時に実行する BattleSetting 順序
    StateData           ← state 自身の runtime データ
    BattleState         ← 対応する BattleManager.BattleState 列挙値
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Name** | いいえ | 表示名（多言語）；空白時は ID にフォールバック |
| **NextState** | はい | 当該 state 終了後に切り替わる state；ステートマシンの骨格 |
| **EnterStateActions** | いいえ | 当該 state 突入時に順次発動する `RCG_BattleSetting` 順序 |
| **StateData** | — | state 内部の `RCG_RuntimeData`（変数保存） |
| **BattleState** | — | 対応する `RCG_BattleManager.BattleState` enum 値（None / PlayerTurn / EnemyTurn 等） |

## 動作説明

### ステートマシンフロー
1. 当該 state 突入 → `OnEnterState(data)` 呼出、`EnterStateActions` を順次適用（`Enable=false` を除外）。
2. State 持続期間 → 外部から `StateUpdate()` を駆動（現状空）。
3. 終了時 → `ExitState()`（現状空）。
4. `m_NextState` に切替。

### State と BattleState の対応
`State` (property) が `m_BattleState.DefaultValue` 文字列を `RCG_BattleManager.BattleState` enum に変換；欠落時は `None` を返す。**このマッピングでデータ駆動 state がプログラム内部 enum ロジックに対応可**。

## 注意事項

*   **`StateUpdate` / `ExitState` は空実装**：現状ステートマシンは外部駆動、これら2つの hook は将来拡張用に予約。
*   **`m_StateActions` 廃止**：`m_EnterStateActions` で置換；旧データのデシリアライズは base フローを通る、**自動移行されない**（プログラム内 `m_EnterStateActions = m_StateActions.Clone();` はコメントアウト済）。
*   **`m_State` enum フィールド廃止**：`m_BattleState`（`RCG_RuntimeDataConst`）で置換；旧フィールドは base class に存在するが、書き込まれない。
*   **`EnterStateActions` フィルタ**：`Enable=false` の setting を除外、編集時に動的有効化/無効化可能。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_BattleStateData.cs`
*   **継承**：`RCG_Asset<RCG_BattleStateData>`
*   **AssetGroup**：`EditBattleSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_NextState` | NextState | `RCG_BattleStateGenData` | |
| `m_EnterStateActions` | EnterStateActions | `List<RCG_BattleSetting>` | |
| `m_StateData` | StateData | `RCG_RuntimeData` | |
| `m_BattleState` | BattleState | `RCG_RuntimeDataConst` | デフォルト `BattleState` 型 |

### A.3 主要メソッド

*   **`OnEnterState(TriggerEffectData, AddActionMode)`** — state 突入；全 `EnterStateActions` 適用。
*   **`StateUpdate()`** — 空殻、外部駆動。
*   **`ExitState()`** — 空殻。
*   **`State` (property)** — `m_BattleState.DefaultValue` から enum 値取得。
*   **`LocalizeName`** — `m_Name.HasLocalize ? m_Name.Name : ID`。
*   **`EnterStateActions` (property)** — `Enable=true` の action リストをフィルタ。

### A.4 他システムとの連携

*   **`RCG_BattleManager.BattleState` (enum)** — 対応するプログラム内部状態。
*   **`RCG_BattleStateGenData`** — Asset Entry ラッパー。
*   **`RCG_BattleSetting`** — `EnterStateActions` の要素。
*   **`RCG_RuntimeData / RCG_RuntimeDataConst`** — 自帯データコンテナ。

### A.5 既知の問題

*   `m_StateActions` と `m_State` enum 廃止、デシリアライズに移行パスなし。
