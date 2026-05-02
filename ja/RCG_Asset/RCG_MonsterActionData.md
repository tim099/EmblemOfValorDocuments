---
title: モンスター動作データ (RCG_MonsterActionData)
description: モンスターが使用可能な単一招式 / 動作テンプレート。選択ルール、効果、アイコン、説明
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# モンスター動作データ

> クラス名：`RCG_MonsterActionData`

## 用途

**モンスターが使用可能な単一招式 / 動作テンプレート**。例：「斬撃」「従者召喚」「自爆」。各 Action の内訳：
*   ターゲット選択ルール（どのユニットがターゲットになり得るか）
*   効果（ダメージ、バフ、召喚など）
*   アイコン
*   説明（自動 / 手動）

`RCG_UnitData` の `MonsterStates` が一連の `RCG_MonsterActionData` を参照、当該状態下で使用可能な招式プールとして使用。

`RCG_Asset<RCG_MonsterActionData>` を継承。

## エディタ上の見た目

```
RCG_MonsterActionData: <ID>
    Action (m_Action)
        SelectUnitRule         ← ターゲット選択ルール
        Effect 構造            ← 実際の効果
        Icon                   ← プレビューアイコン
        OverrideDescription    ← 説明を手動上書きするか
        UnitAction             ← 真に実行する動作
        Infos                  ← Tooltip 列挙の付加情報
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Action** | はい | 内部 `RCG_MonsterAction`、当該招式の全データを含む |

`RCG_MonsterAction` の内部：

| サブフィールド | 説明 |
|---|---|
| **SelectUnitRule** | ターゲット選択ルール（`RandomEnemy` / `Friend` / 自身 / 位置 N など） |
| **OverrideDescription** | `m_UnitAction` の説明を自動説明に上書き使用するか |
| **UnitAction** | 招式が真に実行する動作（ダメージ式、効果順序） |
| **Icon** | プレビュー / Tooltip 表示用アイコン |
| **SkillLevel** | 招式レベル（runtime に `RCG_MonsterLevelActionData.GetAction` が動的に書込） |

## 動作説明

### ターゲット選択 → 発動 → 説明
1. 戦闘中にモンスターが本 Action 使用を選定後、`SelectUnitRule` でターゲットを検索。
2. `m_UnitAction` の効果をターゲットに適用。
3. UI Tooltip に `Icon` + `GetDescription()` + `Infos` の補足タグ情報を表示。

### 自動 vs 上書き説明
`OverrideDescription = true` 時、プレビューがメイン説明の下に**追加で** `m_UnitAction.GetDescription()` を表示（両段表示）。

### デフォルト ID
`RCG_MonsterActionGenData.IdleID` (`"Idle"`) は「無動作」のデフォルト Action；動作未設定のユニットターンはこれを実行。

## 注意事項

*   **SkillLevel は外部から書込**：`RCG_MonsterLevelActionData.GetAction(level)` がレベルを `m_SkillLevel` に書込；本データはテンプレートのみ、runtime はクローン後の副本を取得。
*   **OverrideDescription 二重表示**：Tooltip が長くなる可能性、必要なら無効化。
*   **Idle 動作は削除厳禁**：多くの fallback パスが `RCG_MonsterActionGenData.Idle.GetData()` を返す、この ID 欠如で NRE。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_MonsterActionData.cs`
*   **継承**：`RCG_Asset<RCG_MonsterActionData>`
*   **AssetGroup**：`EditBattleSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_Action` | Action | `RCG_MonsterAction` | 主データコンテナ |

### A.3 主要メソッド

*   **`Preview`** — エディタ描画：アイコン + ターゲット選択ルール + 説明 + Effects 説明（OverrideDescription 時）+ Infos。
*   コンストラクタのデフォルト `ID = "New Action"`。

### A.4 他システムとの連携

*   **`RCG_MonsterAction`** — 主データモデル；`m_SelectUnitRule` / `m_UnitAction` / `m_Icon` / `m_SkillLevel` を含む。
*   **`RCG_MonsterActionGenData`**（同ファイル）— Asset Entry ラッパー；`Idle` がシステムデフォルト。
*   **`RCG_MonsterLevelActionData`** — Action をラップしレベル分けに対応。
*   **`RCG_UnitData.m_MonsterStates`** — Actions を参照するコンテナ。

### A.5 既知の問題

*   コメントアウトされた `DeserializeFromJson` 互換ロジック（`m_ID == "AttackEffect"` → `DefaultID`）あり、旧版 ID 移行履歴を示す。
