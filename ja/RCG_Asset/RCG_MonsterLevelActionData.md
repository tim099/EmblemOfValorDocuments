---
title: レベル化モンスター動作 (RCG_MonsterLevelActionData)
description: 同じ招式の異なるレベル版を束ねる。難易度上昇時に自動的により強い版に切替
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# レベル化モンスター動作

> クラス名：`RCG_MonsterLevelActionData`

## 用途

**「同じ招式の複数レベル版」を束ねる**。例：「攻撃_近距_x1」が `Lv0`、「攻撃_近距_x2」が `Lv1`、「攻撃_近距_AOE」が `Lv2`… モンスターレベル上昇時にシステムがインデックスで自動的により強い版に切替。または「**単一動作モード**」(`m_UseLevelAsIndex = false`) を選択：1つの Action のみ配置、内部の HiddenVariable でレベル読込し調整。

`RCG_Asset<RCG_MonsterLevelActionData>` を継承。

## エディタ上の見た目

```
RCG_MonsterLevelActionData: <ID>
    UseLevelAsIndex (bool)
    MaxSkillLevel
    SkillLevelOffest
    ▼ Actions (UseLevelAsIndex = true)   ← インデックスモード：各レベル対応 Action を列挙
    ▼ SingleAction (UseLevelAsIndex = false) ← 単一モード：Action 1つのみ
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **UseLevelAsIndex** | はい | `true`：`Actions[level]` で対応動作取得；`false`：常に `SingleAction` 使用、内部でレベル処理 |
| **MaxSkillLevel** | はい | レベル上限；超過はクランプ（デフォルト 100） |
| **SkillLevelOffest** | — | レベル基準オフセット（>0 で有効）；このグループの動作の有効レベルを**全体引き上げ** |
| **SingleAction** | UseLevelAsIndex=false 時 | 唯一の Action（自身で `m_SkillLevel` 変数を使い威力調整） |
| **Actions** | UseLevelAsIndex=true 時 | レベル順 Action 一覧；`Actions[i]` がレベル i に対応 |

## 動作説明

### `GetAction(level)`
1. **グローバル難易度スキルレベル加算**：`level += DifficultyData.m_EnemySkillLevel`。
2. `SkillLevelOffest > 0` なら更にオフセット加算。
3. `[0, MaxSkillLevel]` にクランプ。
4. **インデックスモード** (`UseLevelAsIndex = true`)：
    *   `Actions` が空 → `Idle` を返す。
    *   それ以外 `result = Actions[clamp(level, 0, Actions.Count - 1)]`。
5. **単一モード**：直接 `result = SingleAction`。
6. `level` を `result.m_Action.m_SkillLevel` に書込、返却。

### プレビュー
*   インデックスモード：全 Action を列挙（個別展開プレビュー可）。
*   単一モード：SingleAction の1行のみ表示。

## 注意事項

*   **`Actions[i]` は `level i` に対応**：リストインデックスがレベル、未列挙レベルは最後の要素にクランプ。
*   **`MaxSkillLevel = 100` が `Actions.Count` より大きい時**：`Actions.Count - 1` 超過の全レベルが最後の Action 使用（強度天井）。
*   **`SkillLevelOffest` は >0 時のみ有効**：負値は無視；強度低減はグローバル難易度設定経由。
*   **単一モードの `m_SkillLevel`**：runtime 書込はクローンに対し、原 Asset は影響なし。
*   **`m_SingleAction` の HiddenVariable** は設計上の拡張点：Action 内で変数を `SkillLevel` にバインドすると「同招式のレベルに応じた線形強化」を 10 個の Action なしで実現可。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_MonsterLevelActionData.cs`
*   **継承**：`RCG_Asset<RCG_MonsterLevelActionData>`
*   **AssetGroup**：`EditBattleSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_UseLevelAsIndex` | UseLevelAsIndex | `bool` | デフォルト `true` |
| `m_MaxSkillLevel` | MaxSkillLevel | `int` | デフォルト 100 |
| `m_SkillLevelOffest` | SkillLevelOffest | `int` | デフォルト 0；> 0 時のみ加算 |
| `m_SingleAction` | SingleAction | `RCG_MonsterActionGenData` | `Conditional(UseLevelAsIndex == false)` |
| `m_Actions` | Actions | `List<RCG_MonsterActionGenData>` | `Conditional(UseLevelAsIndex == true)` |

### A.3 主要メソッド

*   **`GetAction(int level)`** — 主入口；難易度加算 / オフセット / クランプ / level の Action 書戻し含む。
*   **`Preview`** — 全 Action 列挙（インデックスモード）または単一 Action（単一モード）。

### A.4 他システムとの連携

*   **`RCG_MonsterActionData`** — ここでドロップする Action テンプレート。
*   **`RCG_MonsterActionGenData`** — Asset Entry ラッパー；デフォルト `IdleID = "Idle"`。
*   **`RCG_MonsterLevelActionGenData`**（同ファイル）— このデータを外部参照する型、`m_Level` (`IntVariable`) 自帯；`GetAction()` がこの level で `RCG_MonsterLevelActionData` を問い合わせる。
*   **`RCG_DataService.Ins.m_DifficultyData.m_EnemySkillLevel`** — グローバルスキルレベル加算。

### A.5 既知の問題

*   `RCG_MonsterLevelActionGenData` の内部構築失敗時に LogError 後 `Idle` に fallback；バグを隠蔽するリスク。
