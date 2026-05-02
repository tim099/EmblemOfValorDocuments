---
title: プレイヤーエネルギー変化
description: プレイヤーの現在エネルギー（コスト）を変化 — 増加 / 減少 / 指定値設定
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# プレイヤーエネルギー変化

> クラス名：`RCG_CostAlterSetting`

## 用途
**プレイヤーの現在のエネルギー値（コスト）を変更**。よくある用途：
*   「エネルギー +1」
*   「エネルギー 2 を消費して強力な効果発動」
*   「エネルギーを 0 に設定」（過負荷）

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **CostAlter** | はい | 変化量（変数可）。 |
| **CostAlterType** | はい | 変化モード：<br>• **Add** — エネルギー増加<br>• **Sub** — エネルギー減少（**プレイ判定が掛かる**）<br>• **Set** — 指定値に設定 |

## 挙動
*   **プレイ判定**：`Sub` モード + 純数値の場合、プレイヤーの現エネルギー ≥ `CostAlter` でないとプレイ不可（負エネルギー回避）。他モードは判定なし。
*   **発動**：モードに応じて `RCG_Player.Ins.AlterCost(...)` または `SetCost(...)` を呼ぶ。
*   **VFX**：増加時は `ChargeEffect`、減少時は `OverloadEffect`、加えてキャラ上にエネルギー変化数字をフロート表示。
*   説明はモードで i18n key を切替（`AddCostIcon_Des` / `SubCostIcon_Des` / `SetCostIcon_Des`）+ エネルギーアイコン。

### カードフュージョン
同 `CostAlterType` 同士でのみフュージョン可能（変化量を加算）。

## 注意点
*   **旧データのデシリアライズ**：以前は「`CostAlter` の負値」で減少表現でしたが、現在は `CostAlterType.Sub`。`DeserializeFromJson` が古いデータの負値を `Sub` + 正値に自動変換。
*   **変数型の Sub はプレイ判定をスルー**：プレイ判定は `m_VariableType == Value`（純数値）のときのみ有効；変数型のエネルギー減少は**プレイをブロックせず**、一時的に負エネルギーが発生し得る。
*   **Set はプレイ判定なし**：常にプレイ可能；「Set 0」型カードは戦術空間を奪わないよう注意。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CostAlterSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_CostAlterSetting` → 「プレイヤーエネルギー変化」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_CostAlter` | `CostAlter` | `IntVariable` | — | |
| `m_CostAlterType` | `CostAlterType` | enum (内部) | — | `Add` / `Sub` / `Set` |

### A.3 主なメソッド
*   **`CheckPlayable`**：`Sub` + 純数値のとき `RCG_Player.Ins.Cost >= m_CostAlter`；他モードは true。
*   **`DeserializeFromJson`**：旧データの負 CostAlter → 自動的に `Sub` + 正値に変換（内部で `m_Value` または `m_Mult` を反転）。
*   **`AddAction`** (async)：モードに応じ `RCG_Player.Ins.AlterCost(aValue, isCardCost: false)` または `SetCost(aValue)` を呼び、対応 VFX (`ChargeEffect` / `OverloadEffect` + `VFX_Cost`) を再生。
*   **`Fusion`**：同 `m_CostAlterType` 必須；clone + `IntVariable.FuseAdd`。

### A.4 他システムとの連携
*   **`RCG_Player.Ins.AlterCost / SetCost`**：エネルギー修正エントリ。
*   **`RCG_CommonVFXGenData.s_ChargeEffect / s_OverloadEffect`**：VFX ソース。
*   **`CommonVFX.VFX_Cost`**（`aCostVFX.SetAlterHP` 経由）：エネルギー変化数字フロート UI。
