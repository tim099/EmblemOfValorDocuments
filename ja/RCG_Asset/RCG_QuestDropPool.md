---
title: クエスト/イベントドロップ池 (RCG_QuestDropPool)
description: 「この状況で発生するイベント (RCG_QuestData)」を定義するデータ。マップノードやランダムイベントのソース
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# クエスト/イベントドロップ池

> クラス名：`RCG_QuestDropPool`

## 用途

「**この状況下で発生する可能性のあるイベント (Quest)**」を定義する。例：「中型町」「森の神秘ノード」「夜遭遇」が別々の池に対応；池内には可能な `RCG_QuestData`（イベントスクリプト：会話、選択、結果）と重みを列挙。

`RCG_Asset<RCG_QuestDropPool>` を継承。実装インターフェース：`UCL.Core.UCLI_ShortName`。

## エディタ上の見た目

```
RCG_QuestDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ DropPool / MixDropPools / FilterDropData
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **DropType** | はい | `DropPool` / `MixPool` / `FilterDrop` |
| **DropPool** | DropType=DropPool 時 | イベント一覧 + 重み |
| **MixDropPools** | DropType=MixPool 時 | 他池参照し重み指定 |
| **FilterDropData** | DropType=FilterDrop 時 | 内部 `DropFilter`（FilterType: Tag / Operator）で選別 |

## 動作説明

### 三つのモード
他のドロップ池と同様。FilterDrop は「イベントタグ」一次元のみ対応。

### 暗黙のフィルタ
このクラスは**unlock フィルタなし** — デフォルトフィルタは常に true 返却（プログラム内に「同イベント連続発生防止」のコメントアウトされたロジックあり、現在無効）。

## 注意事項

*   **`enum DropType` で `EDropType` ではない**：BattleSetDropPool / ItemDropPool と命名不整合 — 歴史的経緯。
*   **Name フィールドなし**：`GetShortName()` は最初のドロップ名を直接返す。
*   **不正 ID 自動クリーンアップ用 `DeserializeFromJson` override なし** — 不正 ID は自動消去されず、手動修正が必要。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_QuestDropPool.cs`
*   **継承**：`RCG_Asset<RCG_QuestDropPool>`
*   **実装**：`UCL.Core.UCLI_ShortName`
*   **AssetGroup**：`EditDropSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_DropPool` | ドロップ池 | `RCG_CommonDropSetting<RCG_QuestGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_FilterDropData` | 条件式 | `FilterDropDataBase<RCG_QuestGenData, RCG_QuestData, DropFilter>` | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `DropType` enum | デフォルト `DropPool` |

### A.3 主要メソッド

*   **`GetDrops(int, CheckDropConditionData)`** → 主入口。
*   **`GetDropsWithFilterFunc(int, Func)`** → カスタムフィルタ版。
*   **`GetDropRate(CheckDropConditionData, int)`** → デフォルトフィルタ常に true（「同イベント連続発生防止」ロジックはコメントアウト）。

### A.4 他システムとの連携

*   **`RCG_QuestData`** — ドロップ対象型。
*   **`RCG_QuestGenData`** / **`RCG_QuestDropPoolGenData`** — Asset Entry ラッパー；後者のデフォルト ID = `"NormalDrop"`。
*   **`RCG_EventTagGenData`** — FilterDrop 条件用イベントタグ型。

### A.5 既知の問題

*   コメントアウトされた「同イベント連続発生防止」(`triggeredStory.LastElement()`) ロジック — 必要なら解除。
*   `DeserializeFromJson` override なし、不正 ID は自動クリーンされない。
