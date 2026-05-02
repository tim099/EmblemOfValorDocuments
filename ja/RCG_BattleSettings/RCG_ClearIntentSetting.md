---
title: 意図クリア
description: 対象ユニットの次ターンの意図（敵 AI の攻撃計画）をクリア
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 意図クリア

> クラス名：`RCG_ClearIntentSetting`

## 用途
**敵の次ターンの意図（Intent）をクリア** — 頭上に表示される「次ターンに行うアクション」のこと。よくある用途：
*   「敵の次の攻撃を無効化」風カード
*   「中断」「妨害」系設定

クリア後、敵は次回の決定タイミングで意図を再計算します。

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **対象** (`Target`) | はい | 意図をクリアする対象セレクタ；敵単体または全体を選択可能。 |

## 挙動
*   各ターゲット（**AI を持つもののみ有効**）に対し `target.UnitAI.ClearIntent()` を呼ぶ。
*   説明形式：「**{対象} の意図をクリア**」（i18n key `ClearIntentDes`）。

## 注意点
*   **AI なしのターゲットはスキップ**：味方キャラは通常 AI なし、本設定は味方には無効。
*   **即座に再決定はしない**：クリア後、**次回の AI 決定タイミングまで待ってから新しい意図を生成**；当該ターン中、敵は再度すぐに攻撃しません。
*   **既に実行済みの意図部分はロールバックしない**：敵が既に何らかの攻撃を開始していた場合、意図クリアは未実行の部分にのみ影響。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ClearIntentSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_ClearIntentSetting` → 「意図クリア」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_Target` | 対象 | `RCG_SelectTargetData` | `Target` | |

### A.3 主なメソッド
*   **`AddAction`**：各 target に対し `target.HasAI` をチェック、true なら `target.UnitAI.ClearIntent()` を呼ぶ。
*   **`GetDescriptionFormat`** → i18n key `ClearIntentDes`。

### A.4 他システムとの連携
*   **`RCG_BattleUnit.HasAI / UnitAI.ClearIntent`**：AI システムの意図クリアエントリ。
