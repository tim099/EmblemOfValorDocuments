---
title: 変数宣言
description: 多種類の戦闘条件から数値を取得し名前付き変数に保存、後続設定が参照可能
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 変数宣言

> クラス名：`RCG_VariableSetting`

## 用途
**各種戦闘条件から数値を取り出して指定の変数に保存**し、後続設定がこの変数を参照可能に。「**動的効果**」のコア — 「**本戦闘で使用された各攻撃カードに 1 ダメージ**」のような戦闘状態スケールカードを実現。

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **VariableName** | はい | 変数名（デフォルト `X`）、後続設定がこの名前で参照。 |
| **VariableCondition** | はい | 取値条件（**18 種**、下表）。 |
| **Range** | TargetStatusCount/TargetAttackPower/TargetArmorCount/TargetDebuffLayersSum モード | 対象範囲（attack range スタイル）。 |
| **UsedCardType** | CardUsedCount/TurnCardUsedCount モード | 計算するカードタイプ（空リスト = 制限なし）。 |
| **Status** | TargetStatusCount モード | 層数を問い合わせる状態タイプ。 |
| **MonsterTags** | MonsterCount モード | モンスタータイプを限定；空リスト = 全部。 |
| **Value** | IntVariable モード | 直接指定する固定数値（変数可）。 |

### VariableCondition 18 モード
| 値 | 取得内容 |
|---|---|
| **RemainCost** | プレイヤーの当前エネルギー（デフォルト） |
| **HandCardCount** | 手札枚数 |
| **DiscardedCardCount** | 捨て札枚数 |
| **DeckCardCount** | 抽出牌山枚数 |
| **BanishedCardCount** | 消滅牌山枚数 |
| **CardUsedCount** | 本戦闘で使用したカード（タイプ限定可） |
| **TurnCardUsedCount** | 本ターンで使用したカード（タイプ限定可） |
| **ThisCardUseCount** | このカードの累積使用回数 |
| **ThisCardCost** | このカードの当前コスト |
| **CostOfSelectedCards** | 選択カードのコスト合計 |
| **TargetStatusCount** | 対象の指定状態の層数 |
| **TargetDebuffLayersSum** | 対象の全 Debuff 層数の合計 |
| **TargetArmorCount** | 対象のブロック層数 |
| **TargetAttackPower** | 対象の当前攻撃力（モンスター AI 行動中の最大値） |
| **MonsterCount** | 場上の生存敵数（タイプ限定可） |
| **PlayerUnitCount** | 場上の生存味方ユニット数 |
| **DarkMistLevel** | 暗霧レベル（リソース `Supply`） |
| **IntVariable** | 直接指定値または変数計算結果 |

## 挙動
*   実行時：VariableCondition で数値を計算 → `iData.VariableDic[VariableName]` に保存。
*   説明形式：「**{変数条件説明} {変数名}({実際値}) = {取得値}**」 — 戦闘中はリアルタイムで当前値を表示。
*   Editor モード（非戦闘）では変数値は占位表示（実際には実行されない）。

## 注意点
*   **変数名の衝突**：複数の `VariableSetting` が同名変数を使うと後者が前者を上書き；意味的に命名（`HandCount` / `EnemyAtk` など）。
*   **変数のスコープ**：`iData.VariableDic` に保存 — 同じ `TriggerEffectData` ツリー内で共有。**子 ChildData も見える**（継承）。
*   **VariableName が空**：`AddAction` は早期リターン — この設定は**完全に効果なし**。
*   **計算タイミング**：`AddAction` トリガーポイントで計算 — 後続設定が値を読むときはその瞬間のスナップショット。
*   **プレビューダメージ**：`GetPreviewDamage` は変数値を予め `VariableDic` に書き込み、下流のプレビュー計算で利用可能に。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_VariableSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_VariableSetting` → 「変数宣言」
*   **同ファイルトップレベル enum**：`VariableCondition`（18 個の値）

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_VariableName` | `VariableName` | `string` | `VariableName` (=「変数名」) | デフォルト `"X"` |
| `m_VariableCondition` | `VariableCondition` | `VariableCondition` (内部 enum) | — | デフォルト `RemainCost` |
| `m_Range` | `Range` | `AttackRange` | — | `[Conditional(... 4 種要 Range の条件)]` |
| `m_UsedCardType` | `UsedCardType` | `List<RCG_CardTagGenData>` | — | `[Conditional(... CardUsedCount, TurnCardUsedCount)]` |
| `m_Status` | `Status` | `RCG_CustomStatusGenData` | `Status` | `[Conditional(... TargetStatusCount)]` |
| `m_MonsterTags` | `MonsterTags` | `List<RCG_MonsterTagGenData>` | — | `[Conditional(... MonsterCount)]` |
| `m_Value` | `Value` | `IntVariable` | — | `[Conditional(... IntVariable)]` |

### A.3 主なメソッド
*   **`GetVariableValue(iData)` (protected)**：18 種 case で実際の数値を計算；非戦闘モードまたは `RCG_Player.Ins == null` で 0 を返す。
*   **`AddAction`**：`m_VariableName` が空ならリターン；そうでなければ `iData.VariableDic.Set(m_VariableName, GetVariableValue(iData))`。
*   **`GetPreviewDamage`** → 当前値を VariableDic に書き込み `-1` を返す（**非攻撃カードだが下流のプレビューのために変数を書く**）。
*   **`VarName(iData)` (public)** → 戦闘中は `{name}({value})` を表示；非戦闘は `{name}` のみ。
*   **`GetDescriptionParams / GetDescriptionFormat`**：18 種条件に応じて異なる文型を生成、対応 i18n key を選択（`VariableCondition_*Des` / `VariableEffectDes` / `VariableEffectDes_IntVariable`）。
*   **`Info`**：`TargetStatusCount` モードで `m_Status` 情報を返す；他は null。

### A.4 他システムとの連携
*   **`iData.VariableDic`**：書込先；下流の `IntVariable` 解析時に参照。
*   **`RCG_Player.Ins.Cost / GetHandCardCount / DiscardedCardCount / BanishedCardCount / Deck.Deck.Count`**：抽出データソース。
*   **`RCG_BattleAnalytics.Ins.GetCardUsedCount / GetTurnCardUsedCount`**：カード使用統計。
*   **`RCG_BattleScene.Ins.GetUnits(UnitPos.All, faction)`**：場上ユニットクエリ。
*   **`RCG_DataService.Ins.GetResource(Supply)`**：暗霧レベルデータソース。

### A.5 既知の課題
*   旧版 `DeserializeFromJson` の処理（`IntVariable → RemainCost` 置換など）はコメントアウト。
*   `// QWQ2` / `// TODO` コメントが一部の未対応箇所をマーク（`HandCardCount` 説明パラメータなど）。
