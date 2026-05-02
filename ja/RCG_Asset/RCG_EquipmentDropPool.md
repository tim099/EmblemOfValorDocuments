---
title: 装備ドロップ池 (RCG_EquipmentDropPool)
description: 「どの装備がドロップし、それぞれの重みは」を定義するデータ。ショップ、戦闘報酬、宝箱で装備を抽選
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 装備ドロップ池

> クラス名：`RCG_EquipmentDropPool`

## 用途

「**この池がドロップする装備とそれぞれの重み**」を定義する。戦闘勝利後、ショップ、宝箱、ボス戦利品から装備（武器、防具、アクセサリー、レリック）を抽選する。カード/アイテム池と同じ骨格、違いは「**レリック類装備は重複ドロップ不可**」が runtime でここから除外される点。

`RCG_Asset<RCG_EquipmentDropPool>` を継承。実装インターフェース：`UCL.Core.UCLI_ShortName`。

## エディタ上の見た目

```
RCG_EquipmentDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    Name(多言語)
    ▼ DropPool / MixDropPools / FilterDropData
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **DropType** | はい | `DropPool` / `MixPool` / `FilterDrop` |
| **Name** | いいえ | 表示名（多言語） |
| **DropPool** | DropType=DropPool 時 | 装備一覧 + 重み |
| **MixDropPools** | DropType=MixPool 時 | 他池参照し重み指定 |
| **FilterDropData** | DropType=FilterDrop 時 | 内部 `DropFilter`（SkillTag / RarityTag / Tag / EquipmentType / Operator）で選別 |

## 動作説明

### 三つのモード
他のドロップ池と同様。FilterDrop はカード池より「**装備タイプ (EquipmentType)**」が一段多く、武器/防具/アクセサリーで選別可。

### 暗黙のフィルタ（runtime）
*   **未解放** (`UnlockData.m_LockedEquipments.CheckLocked`)：スキップ。
*   **レリック既保有** (`m_CanDropRepeatedly = false` かつプレイヤーが同 ID 装備所持)：スキップ。**これがレリックが一度のみ出現する仕組み**。
*   **パーティに専門性適合者なし**（装備に `m_SkillTags` 指定があり、パーティ内に保持者なし）：スキップ。
除外後の装備は**重みを再正規化**。

### プレビュー
エディタの `ShowDetail` で最終ドロップ率を確認可。

## 注意事項

*   **レリックの非重複ドロップ**は runtime で `RCG_DataService.Ins.m_EquipmentsData.m_Equipments` と比較する。プレビューでは確認不可、実機テスト必須。
*   **専門性制限のある装備**は特定パーティ構成で大幅縮小、最悪空池になる。設計時に全可能パーティを考慮。
*   **DropPool モード時** デシリアライズで存在しない装備 ID を自動削除。
*   **`CheckRequireSkill` はこのクラスでは常に true を返す**。フィルタロジックは `GetDropRate` 内 closure に集約。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_EquipmentDropPool.cs`
*   **継承**：`RCG_Asset<RCG_EquipmentDropPool>`
*   **実装**：`UCL.Core.UCLI_ShortName`
*   **AssetGroup**：`EditDropSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_Name` | 表示名 | `RCG_LocalizeData` | `[SerializeField] protected` |
| `m_DropPool` | ドロップ池 | `RCG_CommonDropSetting<RCG_EquipmentGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_FilterDropData` | 条件式 | `FilterDropDataBase<RCG_EquipmentGenData, RCG_EquipmentData, DropFilter>` | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `EDropType` enum | デフォルト `DropPool` |

### A.3 主要メソッド

*   **`GetDropEquipments(int, List<RCG_SkillTagGenData>)`** → 主入口；skills 指定なしならパーティの現 skills を自動取得。
*   **`GetDropRate(List<RCG_SkillTagGenData>, ...)`** → unlock + レリック + 専門性チェックを適用。
*   **`CheckRequireSkill`** (static) → 常に `true` を返す（シグネチャ整合性のため、実フィルタは `GetDropRate` 内 closure）。
*   **`DeserializeFromJson`** → 不正 ID クリア。

### A.4 他システムとの連携

*   **`RCG_EquipmentData`** — ドロップ対象型。
*   **`RCG_EquipmentGenData`** / **`RCG_EquipmentDropPoolGenData`** — Asset Entry ラッパー。
*   **`RCG_DataService.Ins.m_EquipmentsData.m_Equipments`** — プレイヤー装備リスト、レリック重複チェック用。
*   **`RCG_DataService.Ins.m_UnlockData.m_LockedEquipments`** — unlock フィルタ。
*   **`RCG_CharacterDataService.Ins.GetAllSkillTags()`** — パーティ専門性問い合わせ。

### A.5 既知の問題

*   `CheckRequireSkill` のコメント「(應該不需要)」「(おそらく不要)」 — ロジックは `GetDropRate` 内 closure へ移動済み。
*   再帰上限10階層。
