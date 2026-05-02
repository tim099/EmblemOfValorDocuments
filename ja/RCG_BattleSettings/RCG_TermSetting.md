---
title: 用語 (Term)
description: 指定用語（Term）が定義する効果をトリガー；用語は独立した RCG_Term システムが管理
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 用語 (Term)

> クラス名：`RCG_TermSetting`

## 用途
**用語（Term）の効果をトリガー**。「用語」はゲーム中で再利用可能な名前付き効果（例：「**消費**」「**詠唱**」「**保留**」「**即死**」「**急速詠唱**」）— 各用語は独立したトリガーロジックとカード情報パネルを持つ。

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Term** | はい | トリガーする用語の enum 値（デフォルト `HighSpeedChant` 急速詠唱）。 |

## 挙動
*   発動時に `RCG_Term.GetTerm(m_Term).TriggerEffect(iData, endAction)` を呼ぶ。
*   説明には用語の本地化名を表示（緑のタイトル色）。
*   **`HasTerm(iTerm)` は true を返す** `iTerm == m_Term` のとき — 上位の用語逆引きでこの設定を見つけられる。
*   `Info` には対応する用語の完全 Tooltip を表示（`Term.Empty` のとき除く）。

## 注意点
*   **用語自体が挙動を決定**：本設定は「**定義済み用語**を発動」するだけ — 用語ロジックは `RCG_Term.GetTerm(...)` 側にある。新 Term を追加するには Term システムを参照。
*   **Term.Empty**：合法だが無効；占位や将来拡張の用途。
*   **用語のトリガー時機**：「保留」「消費」のような用語自体がトリガータイミングを持つ（例：「プレイ時」「捨て札時」）— 本設定が能動的に `TriggerEffect` を呼ぶのは別件、用語自体が能動呼出に対応するか確認を。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_TermSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_TermSetting` → 「カードタグ」（i18n 上の表示）

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_Term` | `Term` | `Term` (enum) | `Term` (=「用語」) | デフォルト `HighSpeedChant` |

### A.3 主なメソッド
*   **`AddAction`**：`RCG_Term.GetTerm(m_Term).TriggerEffect(iData, iEndAction)`。
*   **`HasTerm(Term iTerm)` (override)** → `iTerm == m_Term`；上位の `GetBattleSettings<RCG_TermSetting>` で用語の所属をさらにクエリ可能。
*   **`Info`** → `m_Term == Empty` で null；他は `new CardInfoData(RCG_Term.GetTerm(m_Term))`。
*   **`TermDes` (private property)** → `LocalizeName.GetTagColor(TagColors.Term)`。

### A.4 他システムとの連携
*   **`RCG_Term.GetTerm(Term)`**：用語ファクトリ；対応する Term ロジックインスタンスを返す。
*   **`Term` enum**：用語 ID 集合（`HighSpeedChant`、`InstantDeath`、`Retain`、`Empty`、…）。
*   **`RCG_Extensions.TagColors.Term`**：用語色票。

### A.5 既知の課題
*   旧版 `m_Term == Term.Retain` のデシリアライズ互換はコメントアウト（旧ロジックは `Term.Empty` への強制上書き）。
