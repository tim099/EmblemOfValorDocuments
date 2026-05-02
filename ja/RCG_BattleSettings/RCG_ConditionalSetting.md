---
title: 条件判断
description: AND 条件で分岐するバトル設定；条件成立時は IfConditionFit、不成立時は ElseCondition
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 条件判断

> クラス名：`RCG_ConditionalSetting`

## 用途
**制御フロー型コンテナ**：`m_CardConditions` を **AND ロジック**で判定し、成立時は `m_IfConditionFit`、不成立時は `m_ElseCondition` のパスを実行。よくある用途：
*   「キル時：敏捷 1 付与、それ以外：3 ダメージ追加」
*   「HP < 50%：自己治療、それ以外：ブロック付与」
*   「手札 ≥ 5：ドロー 1、それ以外：1 枚捨てる」

## エディタ表示
```
▼ ✓ [条件判断(Conditional)] (条件描述) なら ⋯ それ以外 ⋯
    状態を判定する          ▶ (条件リスト)
    条件に合えば実行する    ▶ (子設定リスト)
    条件に合わなければ実行する ▶ (子設定リスト)
```

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **状態を判定する** | 通常はい | 条件リスト。**全部**成立で「適合」（AND）。空リスト = 常に適合。 |
| **条件に合えば実行する** | はい（最低 1 項目） | 適合時に実行する子設定リスト。 |
| **条件に合わなければ実行する** | いいえ | 不適合時に実行する子設定リスト。空でも可（不適合時に何もしない）。 |

## 挙動

### 条件判定
*   全条件が AND で結合 — 1 つでも失敗で全体失敗。
*   空リスト = **常に適合** → 常に「条件に合えば実行する」を実行。
*   各条件は独自の設定を持つ（「キル時」「HP 比例」「特定状態保有」等）；個別ドキュメント参照。

### バトル中の挙動
1. 設定発動時に条件結果を計算。
2. 適合 → 「条件に合えば実行する」内の**全有効子設定**を順次実行。
3. 不適合 → 「条件に合わなければ実行する」を同様に実行。
4. 子設定リスト内の無効（チェック外）はスキップ。

### 説明文の表示
*   「条件に合えば実行する」+ 条件あり：「**もし {条件} なら、{適合内容}**」
*   「条件に合わなければ実行する」あり：改行して「**それ以外は {不適合内容}**」
*   複数条件は「**かつ**」で連結

### プレビューダメージの制限
カード右上のプレビューは**条件を先試行**し、適合パスを選択して最大値を表示。ただし — 

> [!WARNING]
> 「**結果型条件**」（例：「**キル時**」「**クリティカル時**」）はプレビュー時にまだ発生していないため正確な判定不可能。プレビュー表示が実際のダメージと食い違うことがあります。**カードがプレイヤーにダメージ数を約束する場合は、安定した条件**（HP 比例、層数比較、状態保有など）を使ってください。

## 注意点

*   **状態を判定する空 + 条件に合わなければ実行する非空** = **設計アンチパターン**：条件は常に成立、「不適合」パスは実行されないが説明には出るため、プレイヤーが混乱します。**条件を最低 1 つ追加するか、「条件に合わなければ実行する」の内容を削除**。
*   **過度なネストを避ける**：条件判断の中に条件判断は可能ですが、説明が「もし A なら：もし B なら：…」になり可読性が悪い — **複数条件の AND で代替**を推奨。
*   **条件は AND、OR ではない**：OR が必要なら現状 2 つの条件判断で分けて処理（または OR コンテナをプログラム拡張依頼）。
*   **無効子設定**：「適合 / 不適合」両パス内のチェック外設定はスキップされますが、**説明合成には登場する**ことがあるためデータを綺麗に保ってください。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ConditionalSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_ConditionalSetting` → 「条件判断」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_CardConditions` | 状態を判定する | `List<RCG_Condition>` | `CardConditions` | AND 結合；空 = true |
| `m_IfConditionFit` | 条件に合えば実行する | `List<RCG_BattleSetting>` | `IfConditionFit` | |
| `m_ElseCondition` | 条件に合わなければ実行する | `List<RCG_BattleSetting>` | `ElseCondition` | |
| `IfConditionFit` (property) | — | `List<RCG_BattleSetting>` | — | `m_IfConditionFit.GetEnableBattleSettings()` |
| `ElseCondition` (property) | — | `List<RCG_BattleSetting>` | — | `m_ElseCondition.GetEnableBattleSettings()` |

### A.3 主なメソッド
*   **`AddAction`**：`AddActionTrigger` 内で `m_CardConditions.CheckConditions_AND(iData)` 判定 → 分岐 → 各子設定の `AddAction(iData, InsertInOrder)`。
*   **`Infos`** → `m_CardConditions[*].Infos` + `IfConditionFit[*].Infos` + `ElseCondition[*].Infos` を `AppendIfNotRepeat` で集約。
*   **`GetBattleSettings<T> / (Type)`** → 自身 + 両分岐の再帰（**OR 全再帰、分岐選択なし**）。
*   **`GetDescriptionFormat`**：
    1. `iData.m_FullSentence` を一時保存し `false` に。
    2. If 子あり + 条件あり → `IfCardConditions` モデル「もし {ConditionDes} なら」。
    3. `ConditionFitThenDes` モデルで「{IfDescription}」を続ける。
    4. Else あり → 改行 + `ElseTriggerEffectsDes` モデル「それ以外は {ElseDescription}」。
    5. `m_FullSentence` を復元、`iData.GetDescription` で文末調整。
*   **`ConditionDes` (private property)** → `m_CardConditions[*].Description` を `WordSeperator + ConditionAnd + WordSeperator` で結合。
*   **`GetPreviewDamage`** → 現在の `iData` で `CheckConditions_AND` を直接実行、対応分岐に対し `Mathf.Max`（**結果型条件には不正確**）。
*   **`PreloadData`** → 両分岐の子設定を `await`。
*   **`GetFusionCandidateSettings`** → 両分岐下の `IsEnable = true` 子設定の候補を集約（**自身は含まない**）。
*   **`GetFusionBaseSetting`**：自身 clone、両分岐を placeholder 化版で再構築、無効子はフィルタ。

### A.4 他システムとの連携
*   **`RCG_Condition`** → 条件判断の基底；`Description` / `CheckCondition(iData)` を提供。
*   **`CheckConditions_AND` (extension)** → `List<RCG_Condition>` の拡張メソッド、全 true で true。
*   **i18n keys**：`IfCardConditions` / `ConditionFitThenDes` / `ElseTriggerEffectsDes` / `ConditionAnd`。
*   **`LocalizedStringUtils.WordSeperator()`** → 中文は空白なし、英文は空白ありの言語感知セパレータ。

### A.5 既知の課題
*   旧 `ElseTriggerEffects` のロジック（直接結合 + 先頭大文字化）はコメントとして比較用に保存。
*   `m_FullSentence` の保存復元は**手動操作**。throw 発生時に状態リーク；`try/finally` のほうが安全。
*   `GetPreviewDamage` の「結果型条件」非正確性は設計上のトレードオフ；設計時に回避を。
