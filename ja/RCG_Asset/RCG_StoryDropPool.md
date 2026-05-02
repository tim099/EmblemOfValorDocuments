---
title: ストーリードロップ池 (RCG_StoryDropPool)
description: 「この状況で抽選されるストーリー (RCG_StoryData)」を定義するデータ。物語段落、ランダムストーリー挿入点ソース
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# ストーリードロップ池

> クラス名：`RCG_StoryDropPool`

## 用途

「**この状況下で抽選される可能性のあるストーリー段落**」を定義する。`RCG_QuestDropPool` と異なり、ここでドロップするのは `RCG_StoryData`（純粋な物語、会話、過場）であり、戦闘や選択は含まない。例：「新章突入時の導入」「夜の休息中の道中見聞」。

`RCG_Asset<RCG_StoryDropPool>` を継承。実装インターフェース：`UCL.Core.UCLI_ShortName`。

## エディタ上の見た目

```
RCG_StoryDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ DropPool / MixDropPools / FilterDropData
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **DropType** | はい | `DropPool` / `MixPool` / `FilterDrop` |
| **DropPool** | DropType=DropPool 時 | ストーリー一覧 + 重み |
| **MixDropPools** | DropType=MixPool 時 | 他池参照し重み指定 |
| **FilterDropData** | DropType=FilterDrop 時 | 内部 `DropFilter` で選別 |

## 動作説明

他のドロップ池と同じ骨格；unlock / skill 等の runtime フィルタなし。

## 注意事項

*   構造は `RCG_QuestDropPool` とほぼ同一、違いはドロップ対象型と選別タグ種類のみ。
*   **DropPool モード時** デシリアライズで不正 ID を自動クリーン（`iData.Exist()` 依拠）。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_StoryDropPool.cs`
*   **継承**：`RCG_Asset<RCG_StoryDropPool>`
*   **AssetGroup**：`EditDropSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_DropPool` | ドロップ池 | `RCG_CommonDropSetting<RCG_StoryGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_FilterDropData` | 条件式 | `FilterDropData` | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `EDropType` enum | デフォルト `DropPool` |

### A.3 主要メソッド

*   **`GetDrops(int)`** / **`GetDropsWithFilterFunc(int, Func)`** — N 個ストーリーを抽選。
*   **`GetDropRate(...)`** — 正規化後重みテーブル。

### A.4 他システムとの連携

*   **`RCG_StoryData`** — ドロップ対象。
*   **`RCG_StoryGenData`** / **`RCG_StoryDropPoolGenData`** — Asset Entry ラッパー。
