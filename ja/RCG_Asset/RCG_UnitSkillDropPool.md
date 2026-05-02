---
title: ユニットスキルドロップ池 (RCG_UnitSkillDropPool)
description: 「レベルアップ時に抽選されるユニットスキル」を定義するデータ。キャラレベルアップ、スキル解放メニューの背後にある池
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# ユニットスキルドロップ池

> クラス名：`RCG_UnitSkillDropPool`

## 用途

「**レベルアップやスキル選択時に抽選される `RCG_UnitSkillData`**」を定義する。例：キャラがレベルアップ時にプレイヤーへ提示する3択のスキル候補は、いずれかの `RCG_UnitSkillDropPool` から抽選される。

`RCG_Asset<RCG_UnitSkillDropPool>` を継承。実装インターフェース：`UCL.Core.UCLI_ShortName`。

## エディタ上の見た目

```
RCG_UnitSkillDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ DropPool / MixDropPools / FilterDropData
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **DropType** | はい | `DropPool` / `MixPool` / `FilterDrop` |
| **DropPool** | DropType=DropPool 時 | スキル一覧 + 重み |
| **MixDropPools** | DropType=MixPool 時 | 他池参照 |
| **FilterDropData** | DropType=FilterDrop 時 | 内部 `DropFilter` で選別 |

## 動作説明

他のドロップ池と同じ骨格。

## 注意事項

*   enum 名は `UnitSkillDropType`（`E` プレフィックスなし、`DropType` でもない）、他のドロップ池と命名不整合 — 歴史的経緯。
*   構造は `RCG_CardDropPool` と類似、ドロップ対象が `RCG_UnitSkillData` に変わるのみ。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_UnitSkillDropPool.cs`
*   **継承**：`RCG_Asset<RCG_UnitSkillDropPool>`
*   **AssetGroup**：`EditDropSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_DropPool` | ドロップ池 | `RCG_CommonDropSetting<RCG_UnitSkillGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_FilterDropData` | 条件式 | `FilterDropData` | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `UnitSkillDropType` enum | デフォルト `DropPool` |

### A.3 主要メソッド

*   **`GetDrops(int)`** / **`GetDropsWithFilterFunc(int, Func)`** — N 個スキルを抽選。

### A.4 他システムとの連携

*   **`RCG_UnitSkillData`** — ドロップ対象。
*   **`RCG_UnitSkillGenData`** / **`RCG_UnitSkillDropPoolGenData`** — Asset Entry ラッパー。
