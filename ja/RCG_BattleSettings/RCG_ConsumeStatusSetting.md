---
title: 状態を消費する
description: 自身の状態スタックを消費して効果を発動；スタック不足時はプレイ不可化または不発動を選択可能
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 状態を消費する

> クラス名：`RCG_ConsumeStatusSetting`

## 用途
**自身の状態スタックを消費して効果を発動**。例：「剣気護体 1 層を消費 → 20 物理ダメージ」。よくある用途：
*   「累積 + 爆発」スタイルのカードコンボ
*   一部の強力な効果に事前蓄積を要求する設計

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **状態** (`Status`) | はい | 消費する状態タイプ（`RCG_CustomStatusGenData`）。 |
| **量** (`Amount`) | はい | 消費する層数；変数可。 |
| **Setting** | はい | 消費成功後に実行する「**組み合わせ効果**」。 |
| **詳細設定** (`DetailSetting`) | いいえ | 「状態」設定の DetailSetting と共有（アニメ・タグ等を制御）。 |
| **CheckPlayable** | — | デフォルト OFF。ON = スタック不足のとき**プレイ不可**；OFF = プレイは可能だが効果はトリガーしない。 |

## 挙動
*   **層数判定**：`iData.User`（自身）の現在の状態層数を `Amount` と比較。
*   **CheckPlayable モード**：
    *   ON：スタック不足のときカードがグレーアウトして使用不可。
    *   OFF：スタック不足でもカードはプレイ可能だが `m_Setting` はトリガーされない。
*   **発動**：指定層数を差し引き（`-amount`）→ `m_Setting` を実行。
*   **プレビューダメージ**：条件成立時は `m_Setting` のプレビューを取得；不成立時は 0（**-1 ではない**ため「非攻撃カード」扱いにはならない）。
*   説明形式：「**{Amount} 層の {Status} アイコンを消費**」+ 子設定説明（i18n key `ConsumeStatusDes`）。

## 注意点
*   **プレビュー不正確のケース**：状態層数がカード計算時動的（例：「先にドロー、その後消費」）の場合、プレビューが実際と食い違うことが。
*   **Status は積み重ね可能な状態必須**：層数を持たない状態に対しては挙動が未定義。
*   **「状態」設定との違い**：「状態」は層数付与；本設定は**自身の**層数を消費。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ConsumeStatusSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：未登録（エディタ表示は stripped name `ConsumeStatus`）

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_Status` | 状態 | `RCG_CustomStatusGenData` | `Status` | |
| `m_Amount` | 量 | `IntVariable` | `Amount` | デフォルト 1 |
| `m_Setting` | `Setting` | `RCG_CombineSetting` | — | 子効果コンテナ |
| `m_DetailSetting` | 詳細設定 | `RCG_StatusSetting.DetailSetting` | `DetailSetting` | 「状態」設定の DetailSetting を共有 |
| `m_CheckPlayable` | `CheckPlayable` | `bool` | — | デフォルト `false` |

### A.3 主なメソッド
*   **`CheckCondition(iData, out amount)` (public)**：`iData.User.m_UnitStatus.GetStatusLayer(m_Status)` と `m_Amount.GetValue(iData)` を比較。
*   **`CheckPlayable`**：`m_CheckPlayable` が false なら常に true；そうでなければ `CheckCondition` を実行。
*   **`AddAction`**：条件成立 → `CreateAction.StatusAction(User, Status, -amount, ..., DetailSetting)` + `m_Setting.AddAction(InsertInOrder)`。
*   **`GetBattleSettings<T> / (Type)`**：自身 + `m_Setting` を再帰。
*   **`GetDescriptionFormat`**：`m_FullSentence = false` を一時保存 → i18n key `ConsumeStatusDes` + 子設定 format → 復元・文末調整。
*   **`GetPreviewDamage`**：条件不成立は 0 を返す（**-1 ではない**）；成立は `m_Setting` に委譲。
*   **`PreloadData`**：`m_Setting.PreloadData` を await。

### A.4 他システムとの連携
*   **`RCG_UnitStatus.GetStatusLayer(status)`**：当前層数取得エントリ。
*   **`CreateAction.StatusAction`**：実際に層数を変更する Action ビルダー。
*   **`RCG_CustomStatusGenData.IconTMPKey`**：説明文に埋め込まれるアイコン。
