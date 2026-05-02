---
title: フィールド効果ドロップ池 (RCG_FieldEffectDropPool)
description: 「抽選されるフィールド効果」を定義するデータ。マップノード/戦闘開始時のランダムフィールド修正ソース
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# フィールド効果ドロップ池

> クラス名：`RCG_FieldEffectDropPool`

## 用途

「**この状況下で抽選される可能性のあるフィールド効果**」を定義する。フィールド効果は**戦場全体に作用**する修正（例：本戦闘中、全ユニット毎ターン HP -1 / 火炎フィールド / 暗闇覆い）；この池がランダムフィールド時の候補と重みを決定。

`RCG_Asset<RCG_FieldEffectDropPool>` を継承。実装インターフェース：`UCL.Core.UCLI_ShortName`。

## エディタ上の見た目

```
RCG_FieldEffectDropPool: <ID>
    DropType  ▾ DropPool / MixPool      ← FilterDropなし
    ▼ DropPool / MixDropPools
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **DropType** | はい | `DropPool` / `MixPool`（**二つのモードのみ**） |
| **DropPool** | DropType=DropPool 時 | フィールド効果一覧 + 重み |
| **MixDropPools** | DropType=MixPool 時 | 他池参照 |

## 動作説明

`RCG_StatusDropPool` と同構造 — DropPool / MixPool の二つのモードのみ。

## 注意事項

*   **FilterDrop モードなし**。
*   **enum 名は `DropType`**（`EDropType` ではない）。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_FieldEffectDropPool.cs`
*   **継承**：`RCG_Asset<RCG_FieldEffectDropPool>`
*   **AssetGroup**：`EditDropSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_DropPool` | ドロップ池 | `RCG_CommonDropSetting<RCG_FieldEffectGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_DropType` | DropType | `DropType` enum | `DropPool` / `MixPool` のみ |

### A.3 他システムとの連携

*   **`RCG_FieldEffectData`** — ドロップ対象。
*   **`RCG_FieldEffectGenData`** / **`RCG_FieldEffectDropPoolGenData`** — Asset Entry ラッパー。
