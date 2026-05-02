---
title: 状態効果ドロップ池 (RCG_StatusDropPool)
description: 「ドロップする状態効果」を定義するデータ。ランダムバフ/デバフ、神秘の祝福などの場面で使用
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 状態効果ドロップ池

> クラス名：`RCG_StatusDropPool`

## 用途

「**この状況下で抽選される可能性のある状態効果**」を定義する。例：「祭壇からのランダムな祝福」「呪われた宝箱からのランダムなデバフ」「特殊イベントの全体バフ」など、すべてここから `RCG_CustomStatusData` を抽選する。

`RCG_Asset<RCG_StatusDropPool>` を継承。実装インターフェース：`UCL.Core.UCLI_ShortName`。

## エディタ上の見た目

```
RCG_StatusDropPool: <ID>
    DropType  ▾ DropPool / MixPool      ← FilterDropなし
    ▼ DropPool / MixDropPools
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **DropType** | はい | `DropPool` / `MixPool`（**二つのモードのみ**、FilterDropなし） |
| **DropPool** | DropType=DropPool 時 | 状態一覧 + 重み |
| **MixDropPools** | DropType=MixPool 時 | 他池参照 |

## 動作説明

他のドロップ池の骨格と類似だが、**二つのモードのみ**：直接列挙、他池混合。「条件による動的選別」オプションなし。

## 注意事項

*   **FilterDrop モードなし**：動的選別は手動列挙が必要。
*   **enum 名は `DropType`**（`EDropType` ではない）。
*   unlock 等の runtime フィルタなし。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_StatusDropPool.cs`
*   **継承**：`RCG_Asset<RCG_StatusDropPool>`
*   **AssetGroup**：`EditDropSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_DropPool` | ドロップ池 | `RCG_CommonDropSetting<RCG_CustomStatusGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_DropType` | DropType | `DropType` enum | `DropPool` / `MixPool` のみ |

### A.3 他システムとの連携

*   **`RCG_CustomStatusData`** — ドロップ対象。
*   **`RCG_CustomStatusGenData`** / **`RCG_StatusDropPoolGenData`** — Asset Entry ラッパー。
