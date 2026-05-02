---
title: カード強化ドロップ池 (RCG_CardEnhenceDropPool)
description: 「カード強化時に抽選される強化分岐」を定義するデータ。強化メニューの背後にあるランダム池
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# カード強化ドロップ池

> クラス名：`RCG_CardEnhenceDropPool`

## 用途

「**あるカードを強化する際、抽選される強化分岐 (`RCG_CardEnhenceData`) の候補**」を定義する。各カードの `m_EnhencePool` フィールドが `RCG_CardEnhenceDropPool` を参照；強化時にこの池から N 個の分岐を抽選してプレイヤーに選ばせる。

`RCG_Asset<RCG_CardEnhenceDropPool>` を継承。実装インターフェース：`UCL.Core.UCLI_ShortName`。

## エディタ上の見た目

```
RCG_CardEnhenceDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ DropPool / MixDropPools / FilterDropData
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **DropType** | はい | `DropPool` / `MixPool` / `FilterDrop` |
| **DropPool** | DropType=DropPool 時 | 強化分岐一覧 + 重み |
| **MixDropPools** | DropType=MixPool 時 | 他池参照 |
| **FilterDropData** | DropType=FilterDrop 時 | 内部 `DropFilter`（Operator）で選別 |

## 動作説明

他のドロップ池と同じ骨格。内部 `DropFilter` は現状**Operator 型のみ対応**（Tag はコメントアウト済）、実用上 FilterDrop モードはあまり使われず、多くは DropPool で直接列挙。

抽選時に、カード側の `BannedEnhence` リスト + `RCG_CardEnhenceCondition` で二次フィルタされる（`RCG_CardData.GetEnhenceBranchs` 参照）。

## 注意事項

*   **enum 名は `DropType`**（`EDropType` ではない）。
*   **DropFilter Tag はコメントアウト**：FilterDrop モードは現段階で機能制限あり；DropPool モードで直接列挙推奨。
*   **強化条件と BannedEnhence の二次フィルタ**はカード側で行われ、この池は「初回候補抽選」のみを担当。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_CardEnhenceDropPool.cs`
*   **継承**：`RCG_Asset<RCG_CardEnhenceDropPool>`
*   **AssetGroup**：`EditDropSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_DropPool` | ドロップ池 | `RCG_CommonDropSetting<RCG_CardEnhenceGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_FilterDropData` | 条件式 | `FilterDropData` | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `DropType` enum | デフォルト `DropPool` |

### A.3 他システムとの連携

*   **`RCG_CardEnhenceData`** — ドロップ対象（強化分岐定義）。
*   **`RCG_CardEnhenceGenData`** / **`RCG_CardEnhenceDropPoolGenData`** — Asset Entry ラッパー。
*   **`RCG_CardData.m_EnhencePool` / `RCG_CardData.GetEnhenceBranchs`** — 強化フローの呼び出し元、この池から候補抽選後に二次フィルタ実行。
