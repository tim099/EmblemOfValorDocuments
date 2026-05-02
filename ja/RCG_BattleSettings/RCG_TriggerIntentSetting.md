---
title: 意図トリガー
description: ターゲットの次ターン行動を強制発動（攻撃前倒し / 敵スキルの模倣など）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 意図トリガー

> クラス名：`RCG_TriggerIntentSetting`

## 用途
**ターゲットユニットの次ターン意図を強制発動**（頭上に表示される攻撃計画）。よくある用途：
*   「敵を強制的に前倒しで攻撃させる」（プレイヤーが事前に予測 / 反撃可能）
*   「敵方スキルの模倣」（ターゲットの行動を複製）
*   「発動後に意図をリフレッシュ」（発動後に新しい意図と入れ替わる）

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **TriggerIntentType** | はい | トリガータイプ：<br>• **Default** — 意図発動<br>• **TriggerAndRefresh** — 発動後にリフレッシュ（新しい意図と入替）<br>• **ImitateIntention** — ターゲット意図を模倣（使用者がターゲットの行動を実行） |
| **対象** (`Target`) | はい | 対象ユニットセレクタ；AI を持たないものはフィルタ。 |

## 挙動
*   `Target` を解析して AI を持つもののみフィルタし、`BattleID` でソートして安定性を確保。
*   **Default / TriggerAndRefresh**：各ターゲットに対し `RCG_IntentAction(target, type)` Action を挿入、**挿入位置は Last**（キュー末尾、他動作完了後に実行）。
*   **ImitateIntention**：**最初のターゲット**の AI 当前 `Action.m_UnitAction` を取得し、使用者（`iData.User`）が発動者として実行 — その攻撃を模倣。

## 注意点
*   **AI を持たないターゲットは自動スキップ**：味方キャラは通常 AI なし、本設定は味方には無効。
*   **ImitateIntention は 1 ターゲットだけ**：複数セレクタは最初のもののみ採用、他は無視。
*   **「意図クリア」との対比**：クリアは敵に**再決定**させる；本設定は**当前意図を強制実行**。
*   **Last 挿入位置**：意図トリガーは**他動作完了後**に実行 — `InsertInOrder` と異なるため接続順序に注意。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_TriggerIntentSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_TriggerIntentSetting` 未登録（エディタ表示は stripped name `TriggerIntent`）
*   **同ファイルトップレベル enum**：`ETriggerIntentType` (`Default`, `TriggerAndRefresh`, `ImitateIntention`)

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_TriggerIntentType` | `TriggerIntentType` | `ETriggerIntentType` | — | 3 種タイプ |
| `m_Target` | 対象 | `RCG_SelectTargetData` | `Target` | |

### A.3 主なメソッド
*   **`AddAction`**：`m_Target.GetTargets(iData).Where(HasAI).OrderBy(BattleID)`、`m_TriggerIntentType` で分岐：
    *   `ImitateIntention` → `aTargets.FirstOrDefault().UnitAI.CurrentAction.m_UnitAction.AddAction(iData, InsertInOrder)`。
    *   他 → 各 target に `iData.AddAction(new RCG_IntentAction(target, m_TriggerIntentType), AddActionMode.Last)`。
*   **`GetDescriptionFormat`** → `m_TriggerIntentType.GetLocalizeDes(target)`。

### A.4 他システムとの連携
*   **`RCG_BattleUnit.HasAI / UnitAI.CurrentAction`**：AI システムの当前行動クエリ。
*   **`RCG_IntentAction`**：実際に意図をトリガーする Action class。
*   **`AddActionMode.Last`**：キュー末尾挿入、`InsertInOrder` と挙動が異なる。
