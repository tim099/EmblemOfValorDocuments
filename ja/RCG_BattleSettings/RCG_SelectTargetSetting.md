---
title: 対象選択
description: ターゲットを再選択（プレイヤークリック式）；既存のターゲットリストを上書き
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 対象選択

> クラス名：`RCG_SelectTargetSetting`

## 用途
**プレイヤーに攻撃 / 効果対象を再選択させる**（選択 UI を開く）。後続設定は新しいターゲットリストを使用。よくある用途：
*   「対象を 1 つ選び、それに 5 倍ダメージ」
*   複雑な効果でプレイヤーに手動で対象を指定させる

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **TargetType** | はい | 選択可能な対象タイプ（`TargetType` enum、例：All / Enemy / Ally / SingleEnemy など）。 |

## 挙動
*   実行時：戦闘入力をアンロック → 対象選択 UI を開く → プレイヤーがクリックすると `iData.Targets` に書込み → 再ロック。
*   説明には `TargetType` の本地化文字列を表示。
*   選択中はバトル動作が一時停止；プレイヤーがクリックするまで進まない。

## 注意点
*   **AI トリガー時は無効**：本設定はプレイヤー入力に依存；AI トリガー時はバトルがフリーズします。**AI には「**ターゲットを設定する**」**(`RCG_SetTargetSetting`) を使ってください。
*   **元ターゲットを上書き**：選択完了後 `iData.Targets` が上書きされ、後続子設定はすべて新ターゲットを基準とする。
*   **「ターゲットを設定する」との違い**：本設定は**UI でプレイヤーに選ばせる**；「ターゲットを設定する」は**自動でルール適用**。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_SelectTargetSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_SelectTargetSetting` → 「対象選択」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_TargetType` | `TargetType` | `TargetType` (enum) | — | デフォルト `All` |

### A.3 主なメソッド
*   **`AddAction`** (async)：`RCG_BattleManager.Ins.UnBlockAction(true)` → `await RCG_BattleField.Ins.SelectTargetsAsync(m_TargetType, false, iToken)` → `iData.SetTargets(targets)` → `BlockAction()`。
*   **`GetDescriptionFormat`**：`m_TargetType.GetDescription()` を直接表示。

### A.4 他システムとの連携
*   **`RCG_BattleField.Ins.SelectTargetsAsync`**：UI 対象選択エントリ。
*   **`RCG_BattleManager.UnBlockAction / BlockAction`**：戦闘入力ロック制御。
*   **`TargetType` enum**：選択可能な対象タイプ集合。
