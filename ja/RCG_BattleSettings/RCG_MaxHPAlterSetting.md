---
title: 最大体力の変更
description: ターゲットの最大 HP を変更；永久 / 本戦闘内のスコープ選択可能
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 最大体力の変更

> クラス名：`RCG_MaxHPAlterSetting`

## 用途
**ターゲットの最大 HP を変更**。よくある用途：
*   キャラクターの血量を永久に増加（バトル報酬 / レベルアップ）
*   本バトル内で血量を一時的に上げる（暫定 Buff、戦闘後リセット）

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **AlterType** | はい | 変化スコープ：<br>• **Permanently** — 永久に最大 HP を増加（戦闘後も保持）<br>• **ThisBattle** — 本バトル限定（戦闘後リセット） |
| **量** (`Amount`) | はい | 変化量（変数可、正負どちらも可）。 |
| **対象** (`Target`) | はい | 対象ユニットセレクタ。 |

## 挙動
*   生存中のターゲットに `target.AlterMaxHP(AlterType, Amount)` を呼ぶ。
*   説明形式：「**{Permanently/ThisBattle} {対象} の最大 HP を {Amount} 増加**」（i18n key `MaxHPIcon_Des`、ハートアイコン付き）。
*   実行後 0.6 秒待機して UI アニメ完了を確保。

### カードフュージョン
2 枚の本設定をフュージョンすると Amount を加算（**AlterType の同一性はチェックしない**、要注意）。

## 注意点
*   **AlterType 不一致のフュージョン**：永久 + 本戦闘がフュージョンすると AlterType は**前者**を採用、「永久アップグレード」が「暫定」と誤分類される可能性。
*   **負値 Amount = HP 上限の削減**：合法だが、ターゲットの当前 HP が新上限を超える場合の挙動は `AlterMaxHP` 実装次第（要確認）。
*   **死亡ターゲットスキップ**：既に死亡しているターゲットは変更されず、死人への加 HP を回避。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_MaxHPAlterSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_MaxHPAlterSetting` → 「最大体力の変更」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_AlterType` | `AlterType` | `AlterType` (内部 enum) | — | `Permanently` / `ThisBattle` |
| `m_Amount` | 量 | `IntVariable` | `MaxHP` (=「最大HP」) | `[UCL_FieldOnGUI]` はコメントアウト |
| `m_Target` | 対象 | `RCG_SelectTargetData` | `Target` | |

### A.3 主なメソッド
*   **`AddAction`** (async)：`aTargets` 中の生存中の target に対し `aTarget.AlterMaxHP(m_AlterType, aAmount)` を呼ぶ。後 `await Delay(0.6f)`。
*   **`Fusion`**：clone + `IntVariable.FuseAdd(m_Amount)`、**`m_AlterType` の一致はチェックしない**。
*   **`GetDescriptionFormat`** → i18n key `MaxHPIcon_Des`、4 個のパラメータ（Amount / Target / ハートスプライト / AlterType localize 名）。

### A.4 他システムとの連携
*   **`RCG_BattleUnit.AlterMaxHP(AlterType, int)`**：実際の修正エントリ；当前 HP / 上限の同期処理。
*   **`EffectIcon.Health`**：ハートアイコン。

### A.5 既知の課題
*   旧 `CanEnhence / Enhence` はコメントアウト（強化系リファクタ中）。
