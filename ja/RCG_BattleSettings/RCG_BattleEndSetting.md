---
title: バトル終了
description: 即座に現在のバトルを終了；結算タイプ（逃走・勝利・敗北など）を指定可能
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# バトル終了

> クラス名：`RCG_BattleEndSetting`

## 用途
**バトルを直接終了**。最も多い用途は「**逃走**」カード — 手札の逃走カードがこの設定を使ってバトルから離脱します。

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **WinState** | はい | 結算タイプ：`Flee`（逃走）/ `Win`（勝利）/ `Lose`（敗北）など。`WinState` enum 参照。 |

## 挙動
*   **プレイ判定**：「**敵タイプが逃走を許可している**」（`m_EnemyType.GetData().m_CanFlee`）場合のみカードがプレイ可能。Boss 戦は不可になり得る。
*   **発動**：`RCG_BattleEndAction(WinState)` を生成しキューに挿入；バトルフローが WinState に応じて結算スクリプトを実行。
*   説明文に `WinState` の i18n 説明を直接表示（例：「逃走」「勝利」）。

## 注意点
*   **Boss / 強制バトル場面**：プレイヤーがこの設定で逃げ出さないように；Boss 戦は `EnemyType.m_CanFlee = false` を設定。
*   **Win / Lose の慎重な使用**：勝敗は通常 HP 結算で自動判定；特殊シナリオ（自動失敗、強制勝利）のみ手動設定すべき。
*   **「逃走不可」状態のカード表示**：プレイ判定が false でカードはグレーアウトしますが、説明には「逃走」と出続けるためプレイヤーが混乱します — カード上に注記を。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_BattleEndSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **`[System.Serializable]`** 標記
*   **i18n クラス名 key**：`RCG_BattleEndSetting` → 「バトル終了」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_WinState` | `WinState` | `WinState` (enum) | — | デフォルト `Flee` |

### A.3 主なメソッド
*   **`CheckPlayable`** → `CanFlee`、`RCG_BattleManager.Ins.m_BattleData.m_EnemyType.GetData().m_CanFlee` を参照；BattleManager なしの場合は `true`。
*   **`GetDescriptionFormat / GetDescriptionShort`** どちらも `m_WinState.GetLocalizeDes()` を返す。
*   **`AddAction`** → `iData.AddAction(new RCG_BattleEndAction(m_WinState), iAddActionMode)`、ただし `CanFlee` のみ実行（**プレイ判定を一度通過してさらに二重チェック**）。

### A.4 他システムとの連携
*   **`RCG_BattleEndAction`**：実際の結算 Action。
*   **`RCG_EnemyType.m_CanFlee`**：Boss / 特殊敵タイプの逃走禁止スイッチ。
