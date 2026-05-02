---
title: アイテムドロップ池 (RCG_ItemDropPool)
description: 「どのアイテムがドロップし、それぞれの重みは」を定義するデータ。イベント報酬、宝箱、ショップ補充に使用
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# アイテムドロップ池

> クラス名：`RCG_ItemDropPool`

## 用途

「**この池がドロップする消耗アイテムとそれぞれの重み**」を定義する。宝箱、イベント報酬、商人補充などからアイテム（ポーション、巻物、食材など）を抽選する。`RCG_CardDropPool` と同じ骨格、ドロップ対象が `RCG_ItemData` に変わるのみ。

`RCG_Asset<RCG_ItemDropPool>` を継承。実装インターフェース：`UCL.Core.UCLI_ShortName`。

## エディタ上の見た目

```
RCG_ItemDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    Name(多言語)
    ▼ DropPool / MixDropPools / FilterDropData  ← DropTypeに応じて表示
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **DropType** | はい | `DropPool`（直接列挙）/ `MixPool`（他池混合）/ `FilterDrop`（タグ条件選別） |
| **Name** | いいえ | 表示名（多言語）。空白時は最初のドロップ名 → ID へフォールバック |
| **DropPool** | DropType=DropPool 時 | アイテム一覧 + 重み（`RCG_CommonDropSetting<RCG_ItemGenData>`） |
| **MixDropPools** | DropType=MixPool 時 | 他池を参照し重み指定 |
| **FilterDropData** | DropType=FilterDrop 時 | 内部 `DropFilter`（FilterType: Tag / RarityTag / Operator）で動的選別 |

## 動作説明

### 三つのモード
*   **DropPool**：アイテムと重みを手動列挙。
*   **MixPool**：既存池を重み付けで合成。最大10階層まで再帰。
*   **FilterDrop**：条件式（アイテムタグ、レアリティ）、AND / OR / NOT 対応。条件なし = 全アイテムプール均等。

### 暗黙のフィルタ（runtime）
*   **未解放**（`UnlockData.m_LockedItems.CheckLocked`）：スキップ。
除外後のアイテムは**重みを再正規化**。

### プレビュー
エディタの `ShowDetail` ボタンで現在のドロップ率一覧を確認可。

## 注意事項

*   **DropPool モード時** デシリアライズで存在しないアイテム ID を自動削除（`iData.Exist()` チェック）。
*   **内部 `DropFilter`** は `RCG_CardDropPool` の `CardDropFilter` とは別の型 — アイテムには SkillTag や EquipmentType などカード専用フィルタは不要。
*   **MixPool 循環**は `iLayer > 10` で打ち切り。
*   **未解放アイテム**は runtime 池から直接除外される。エディタプレビューはこの runtime ロジックを反映しない場合あり。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_ItemDropPool.cs`
*   **継承**：`RCG_Asset<RCG_ItemDropPool>`
*   **実装**：`UCL.Core.UCLI_ShortName`
*   **AssetGroup**：`EditDropSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_Name` | 表示名 | `RCG_LocalizeData` | |
| `m_DropPool` | ドロップ池 | `RCG_CommonDropSetting<RCG_ItemGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_FilterDropData` | 条件式 | `FilterDropDataBase<RCG_ItemGenData, RCG_ItemData, DropFilter>` | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `EDropType` enum | |

### A.3 主要メソッド

*   **`GetDropItems(int)`** → 主入口；unlock フィルタを自動適用後 N 個抽選。
*   **`GetDropItemsWithFilterFunc(int, Func)`** → カスタムフィルタ版。
*   **`GetDropRate(...)`** → 正規化後重みテーブル。
*   **`DeserializeFromJson`** → 不正 ID クリア。

### A.4 他システムとの連携

*   **`RCG_ItemData`** — ドロップ対象型。
*   **`RCG_ItemGenData`** / **`RCG_ItemDropPoolGenData`** — Asset Entry ラッパー。
*   **`RCG_DataService.Ins.m_UnlockData.m_LockedItems`** — unlock フィルタソース。
*   **`FilterDropDataBase<TGenData, TData, TFilter>`** — 汎用条件選別 base クラス。

### A.5 既知の問題

*   コメント `// 清理不存在的道具 QWQ` がデシリアライズ段階のクリーンアップを示す。
*   再帰上限10階層（`iLayer > 10`）。
