---
title: カードドロップ池 (RCG_CardDropPool)
description: 「どのカードがドロップし、それぞれの重みは」を定義するデータ。報酬画面、ショップ、強化分岐の基盤
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# カードドロップ池

> クラス名：`RCG_CardDropPool`

## 用途

「**この池がドロップするカードと各々の重み**」を定義する。戦闘報酬、ショップ補充、商人特売、イベントドロップなど、いずれかの `RCG_CardDropPool` からランダムにカードを抽選する。同じ池を複数の場所から参照可能 — 一度の変更で全ての参照先に適用される。

`RCG_Asset<RCG_CardDropPool>` を継承。実装インターフェース：`UCL.Core.UCLI_ShortName`。

## エディタ上の見た目

```
RCG_CardDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ 表示名
        Name(多言語)
    ▼ DropPool / MixDropPools / FilterDropData  ← DropTypeに応じて表示
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **DropType** | はい | `DropPool`（カード+重み直接列挙）/ `MixPool`（他の池を混合）/ `FilterDrop`（タグ条件で動的選別） |
| **Name** | いいえ | 表示名（多言語）。空白時は `GetShortName()` が最初のドロップ名 → ID へとフォールバック |
| **DropPool** | DropType=DropPool 時 | カード一覧 + 各重み（`RCG_CommonDropSetting<RCG_CardGenData>`） |
| **MixDropPools** | DropType=MixPool 時 | 他のドロップ池を参照し各重みを指定。最終結果は加重平均 |
| **FilterDropData** | DropType=FilterDrop 時 | `CardDropFilter` 条件式（タグ、レアリティ等）で該当するカードを動的に選別。条件なし = 全カード対象 |

## 動作説明

### 三つのモード
*   **DropPool（直接列挙）**：各カードと重みを手動列挙。最も明示的、厳選池に向く。
*   **MixPool（混合）**：複数の既存池を重み付けで合成。同一名簿の重複保守を回避。重みは階層的に乗算される（外側 weight × 内側 weight）。最大10階層まで再帰、循環防止。
*   **FilterDrop（タグ選別）**：条件式で**全カードプール**から該当カードを選別、ヒット全カードの重みは均等。条件は AND / OR / NOT のネスト対応。

### 暗黙のフィルタ（runtime 戦闘で自動適用）
重み 0.5 と書いても runtime に池から除外される可能性がある：
*   **未解放** (`UnlockData.CheckCardLocked`)：スキップ。
*   **パーティに使用可能者なし** (`CheckRequireSkill`)：カードに専門性指定があり、パーティ内に該当する専門性を持つ者がいない場合スキップ。
*   **メインメニューや非戦闘環境**：スキル判定はスキップ（UI プレビューの空転回避）。
除外後の残カードは**重みを再正規化**（合計1）する。

### プレビュー
エディタ右側の `ShowDetail` を押すと、現条件下での最終ドロップ率一覧をリアルタイムで確認できる。

## 注意事項

*   **DropPool モード時** デシリアライズで**存在しないカード ID を自動削除**（ID 改名 / 削除など）。MixPool モードも同様（不存在の子池を削除）。コンソールに `Remove Invalid DropPool` LogError があれば、不正 ID を参照していた証拠。
*   **FilterDrop の条件が空** = 全カード均等ドロップ。意図的でなければ条件設定を忘れずに。
*   **MixPool の循環**は `iLayer > 10` で空リスト返却で打ち切り。池が空になった時はチェーンに循環がないか確認。
*   **RCG_CardFilter cache**：FilterDrop のクエリは `RCG_CardFilter.GetCards` 経由のキャッシュ付き。新規カード追加後にプレビューがおかしい場合、`RCG_CardData.OnLoadModule` でキャッシュ削除をトリガー可。

---

## 付録：プログラマ参考 (Programmer Reference)

> 以下はプログラム内部用語を使用、対象はプログラマと AI agent。前半部分の内容を優先。

### A.1 クラス情報

*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_CardDropPool.cs`
*   **継承**：`RCG_Asset<RCG_CardDropPool>`
*   **実装**：`UCL.Core.UCLI_ShortName`
*   **AssetGroup**：`EditDropSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | Localize Key | 備考 |
|---|---|---|---|---|
| `m_Name` | 表示名 | `RCG_LocalizeData` | `Name` | `[SerializeField] protected` |
| `m_DropPool` | ドロップ池 | `RCG_CommonDropSetting<RCG_CardGenData>` | — | `Conditional(m_DropType == DropPool)` |
| `m_MixDropPools` | 混合池リスト | `List<MixDropPoolData>` | — | `Conditional(m_DropType == MixPool)` |
| `m_FilterDropData` | 条件式 | `FilterDropData`（内部入れ子） | — | `Conditional(m_DropType == FilterDrop)` |
| `m_DropType` | DropType | `EDropType` enum | `DropType` | デフォルト `DropPool` |

### A.3 主要メソッド

*   **`GetDropCards(int, bool)`** → 主な外部入口、`iDropCount` 枚のランダムカードを返す。
*   **`GetDropCardsWithFilterFunc(int, Func)`** → 外部カスタムフィルタ版。
*   **`GetDropRate(bool, CheckDropConditionData, int)`** → 最終重みテーブル取得（合計=1）；`iIsFilterSkill = true` の場合 `CheckRequireSkill` を自動適用。
*   **`CheckRequireSkill(RCG_CardGenData)`** (static) → 三層チェック：runtime 中 → メインメニュースキップ → 解放チェック → パーティ専門性適合。
*   **`DeserializeFromJson`** → ロード時に不正 ID をクリア（DropPool / MixPool 各モードで処理）。
*   **`Preview`** → エディタ内プレビュー UI、展開で完全なドロップ率表を表示可。

### A.4 他システムとの連携

*   **`RCG_CommonDropSetting<T>`** — 汎用ドロップコンテナ；`m_DropPool` で直接使用。
*   **`RCG_CardGenData`** — カード ID ラッパー；ドロップ結果は `List<RCG_CardGenData>`。
*   **`RCG_CardDropPoolGenData`** — Asset Entry ラッパー；他 Asset がこの池を参照する際の型。
*   **`RCG_CardFilter`** — `FilterDrop` モードでカードプールを問い合わせる入口。
*   **`RCG_DataService.Ins.m_UnlockData`** — runtime ロック判定。
*   **`RCG_CharacterDataService.Ins.GetAllSkillTags()`** — パーティ専門性問い合わせ。

### A.5 既知の問題

*   `MixPool` の重み計算は内部正規化していない（`aWeight * aDrop.m_Weight` の単純累積）；最終的に `GetDropRate()` で合計=1に正規化。
*   再帰上限10階層（`iLayer > 10` で空返却）はマジックナンバー。
