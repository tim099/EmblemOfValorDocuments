---
title: ランダム発動
description: ランダムに N 個の効果を選んで発動 / 確率で発動 / 変数確率で発動の 3 モード
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# ランダム発動

> クラス名：`RCG_RandomTriggerSetting`

## 用途
**効果をランダムに発動**。3 モード：
*   **ランダム選択 N 個**：候補リストから N 個を抽選して発動
*   **一括確率発動**：固定確率で全効果一括発動
*   **一括変数確率発動**：上と同じだが確率を変数で指定（例：「現 HP 比率で発動確率変動」）

よくある用途：賭博系カード、確率型 Buff、多選一ランダム効果。

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **RandomTriggerMode** | はい | 3 モード（下記）。 |
| **RandomPickCount** | RandomPick モード | ランダムに何個抽選して発動するか。`Conditional` で表示制御。 |
| **TriggerRate** | TriggerByRate モード | 発動確率（0~100、slider）。 |
| **VariableRate** | TriggerByVariableRate モード | 発動確率変数（値が 0~100 のパーセンテージを表す）。 |
| **TriggerSettings** | はい | 候補効果リスト；`IsEnable = true` のもののみ採用。 |

### モード詳細
*   **RandomPick** — enable な候補から**ランダムに N 個**を抽選発動；同効果は重複発動しない。
*   **TriggerByRate** — 一度 0~99 の乱数を振り、`TriggerRate` 未満なら**全部一括**発動；そうでなければ全スキップ。
*   **TriggerByVariableRate** — TriggerByRate と同じだが、確率は `VariableRate.GetValue` で決定。

## 挙動
*   説明は全候補効果を確率情報と組み合わせて表示（例：「50% 確率で発動：[A], [B]」「ランダムに 1 個発動：[A] [B] [C]」）。

## 注意点
*   **RandomPickCount > 候補数**：`RandomPick` 内部で `RandomPick(list, count)` 使用；候補数を超える分はツール関数で処理（通常上限取得）。
*   **空リスト**：合法だが何も起きない。
*   **TriggerByRate 確率 0 / 100**：「永久に発動しない」「永久に発動する」になる極端ケース、設計ミスの可能性。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_RandomTriggerSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_RandomTriggerSetting` → 「ランダム発動」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_RandomTriggerMode` | `RandomTriggerMode` | enum | — | 3 モード |
| `m_RandomPickCount` | `RandomPickCount` | `IntVariable` | — | `[Conditional(nameof(m_RandomTriggerMode), false, RandomPick)]` |
| `m_TriggerRate` | `TriggerRate` | `int` | — | `[UCL_IntSlider(0, 100)]` + `[Conditional(... TriggerByRate)]`、デフォルト 50 |
| `m_VariableRate` | `VariableRate` | `IntVariable` | — | `[Conditional(... TriggerByVariableRate)]` |
| `m_TriggerSettings` | `TriggerSettings` | `List<RCG_BattleSetting>` | — | フィルタ後 `TriggerSettings = m_TriggerSettings.Where(IsEnable)` |

### A.3 主なメソッド
*   **`AddAction`**：`m_RandomTriggerMode` で分岐：
    *   `RandomPick` → `RCG_GameManager.Random.RandomPick(allEnabled, count)`、順次 `AddAction(InsertInOrder)`。
    *   `TriggerByRate` → `Random.Range(0, 100) < m_TriggerRate` → 発動なら全 enable 子設定を順次 AddAction。
    *   `TriggerByVariableRate` → 同上だが確率は `m_VariableRate.GetValue(iData)`。
*   **`GetFusionCandidateSettings`** → 全 enable 子の候補を集約（自身を含まない）。
*   **`GetFusionBaseSetting`**：clone + `m_TriggerSettings` を placeholder 化版で再構築。

### A.4 他システムとの連携
*   **`RCG_GameManager.Random.RandomPick / Range`**：シード制御のランダムソース（再現性に影響）。
