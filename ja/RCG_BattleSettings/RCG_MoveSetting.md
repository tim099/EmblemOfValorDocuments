---
title: 移動
description: ユニットを指定位置に移動（前列 / 後列 / 反対列 / 指定位置）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 移動

> クラス名：`RCG_MoveSetting`

## 用途
**ユニットを指定位置に移動**。よくある用途：
*   「ポジション交換」カード — 後列キャラを前列に
*   「突撃」 — キャラを敵方位置に移動
*   「撤退」 — 後列に移動して避戦

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **MoveType** | はい | 移動方式：<br>• **ToOther** — 反対列に移動（前↔後切替、デフォルト）<br>• **ToFront** — 前列に移動<br>• **ToBack** — 後列に移動<br>• **ToTarget** — 指定位置に移動（`TargetPositions` と組み合わせ） |
| **MoveTarget** | はい | 移動するユニットセレクタ。 |

## 挙動
*   `ToFront` / `ToBack` / `ToOther` — 各ターゲットを対応列に移動：
    *   `ToOther` → 前列にいれば後列へ、その逆も
    *   `ToFront` → 必ず前列
    *   `ToBack` → 必ず後列
*   `ToTarget` — 外部設定された `TargetPositions` と組み合わせ、**最大 N 個のユニットを N 個の位置に移動**（インデックスでペアリング）。
*   説明形式：「**{MoveTarget} が {MoveType} 移動**」（i18n key `Move_Des`）。

## 注意点
*   **死亡ターゲットスキップ**：各移動前に `IsNullOrDead` をチェックして死人操作を回避。
*   **ToTarget は `TargetPositions` 必須**：通常前段に「**ターゲットを設定する**」(`RCG_SetTargetSetting`) などを置いて `iData.m_TriggerData.m_TargetPositions` を埋めておく必要あり、無いと行き先が無い。
*   **目標位置が既に占有されている場合**：技術的には `RCG_BattleField.MoveUnit` が決定する — 多くの場合は押し出しまたは交換。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_MoveSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_MoveSetting` → 「移動」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_MoveType` | `MoveType` | `UnitMoveType` (内部 enum) | `MoveType` (=「移動」) | 4 モード |
| `m_MoveTarget` | `MoveTarget` | `RCG_SelectTargetData` | — | |

### A.3 主なメソッド
*   **`AddAction`**：`m_MoveType` で分岐：
    *   `ToTarget` → `iData.m_TriggerData.m_TargetPositions` と `aTargets` をインデックスでペアリング、`RCG_BattleField.Ins.MoveUnit(target, position, token)` を呼ぶ。
    *   他 → 各 target に対し `aTargetPosition`（前列 / 後列 / 反対列）を計算、同様 `MoveUnit`。
*   **`GetDescriptionShort`** → 移動 sprite (`EffectIcon.Move`) を直接表示。

### A.4 他システムとの連携
*   **`RCG_BattleField.Ins.MoveUnit(unit, pos, token)`**：実際の移動エントリ（アニメ含む）。
*   **`UnitPos`**：前 / 後 enum。
*   **`iData.m_TriggerData.m_TargetPositions`**：`ToTarget` モードの目標位置ソース。
