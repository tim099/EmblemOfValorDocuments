---
title: 戦闘構成ドロップ池 (RCG_BattleSetDropPool)
description: 「このマップノードで出現する戦闘構成」を定義するデータ。マップイベント、章遭遇の基盤となるランダム池
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 戦闘構成ドロップ池

> クラス名：`RCG_BattleSetDropPool`

## 用途

「**マップ上の特定戦闘ノードを踏んだ際、どの戦闘構成が抽選されるか**」を定義する。例：「通常遭遇」「精鋭遭遇」「ボス戦」が別々の池になる；池内には可能な `RCG_BattleSet`（モンスター配置の具体的設定）と重みを列挙。

`RCG_Asset<RCG_BattleSetDropPool>` を継承。実装インターフェース：`UCL.Core.UCLI_ShortName`。

## エディタ上の見た目

```
RCG_BattleSetDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ DropPool / MixDropPools / FilterDropData
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **DropType** | はい | `DropPool` / `MixPool` / `FilterDrop` |
| **DropPool** | DropType=DropPool 時 | BattleSet 一覧 + 重み |
| **MixDropPools** | DropType=MixPool 時 | 他池参照し重み指定 |
| **FilterDropData** | DropType=FilterDrop 時 | 内部 `DropFilter`（Tag / EnemyType / Operator）で選別 |

## 動作説明

### 三つのモード
他のドロップ池と同様。FilterDrop の条件は「BattleSet タグ」「敵タイプ (EnemyType)」の二次元対応。

### 暗黙のフィルタ
このクラスは**unlock / skill 等の runtime フィルタなし** — 重みのみで抽選。

## 注意事項

*   **enum 名は `EDropType` で `DropType` ではない**：クラス内コメントに「DropType と命名すると GoogleSheet 言語ファイル同期が失敗するため改名」とある。元に戻そうとしないこと。
*   **DropPool モード時** デシリアライズで存在しない BattleSet ID を自動削除。
*   **MixPool 循環**は空打ち切り（`iLayer > 10`）。
*   **Name フィールドなし**：このクラスには表示名なし、`GetShortName()` は最初のドロップ名を直接返す。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_BattleSetDropPool.cs`
*   **継承**：`RCG_Asset<RCG_BattleSetDropPool>`
*   **実装**：`UCL.Core.UCLI_ShortName`
*   **AssetGroup**：`EditBattleSetting`（注意：他の DropPool は `EditDropSetting`）

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_DropPool` | ドロップ池 | `RCG_CommonDropSetting<RCG_BattleSetGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_FilterDropData` | 条件式 | `FilterDropDataBase<RCG_BattleSetGenData, RCG_BattleSet, DropFilter>` | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `EDropType` enum | デフォルト `DropPool` |

### A.3 主要メソッド

*   **`GetBattleSets(int)`** → 主入口；内蔵フィルタなし、N 個直接抽選。
*   **`GetBattleSetsWithFilterFunc(int, Func)`** → 外部カスタムフィルタ版。
*   **`GetDropRate(CheckDropConditionData, int)`** → デフォルトフィルタは `_ => true`。

### A.4 他システムとの連携

*   **`RCG_BattleSet`** — ドロップ対象型。
*   **`RCG_BattleSetGenData`** / **`RCG_BattleSetDropPoolGenData`** — Asset Entry ラッパー；後者のデフォルト ID = `"NormalBattle"`。
*   **`RCG_BattleSetTagGenData`** / **`RCG_EnemyTypeTagGenData`** — FilterDrop 用タグ型。

### A.5 既知の問題

*   `enum EDropType`（E プレフィックス）は GoogleSheet 同期競合回避の歴史的経緯。
*   再帰上限10階層。
