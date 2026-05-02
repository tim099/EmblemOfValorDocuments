---
title: 戦闘タグ組み合わせ (BattleTagCombine)
description: 「組み合わせ効果」の特化版；前置の戦闘タグでメイン効果をラップし、トリガータイミングを指定可能
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 戦闘タグ組み合わせ (BattleTagCombine)

> クラス名：`RCG_BattleTagCombineSetting`

## 用途
「**組み合わせ効果**」の特化版。違いは、**前面に「戦闘タグ」グループを前提条件として束縛**することで、指定したトリガータイミング（`OnPlay` / `OnDraw` など）と**組み合わせて、特定タグを持つ場合**にメイン効果を発動できる点。

実用例：「このカードに【消費】タグがあれば、プレイ時に X 効果」「【敏捷】タグがあれば、ドロー時に Y 効果」

継承元：`RCG_CombineSetting`（よって「説明」「OverrideDescription」「組み合わせ設定」リストも持つ；「組み合わせ効果」を参照）

## 主なフィールド（「組み合わせ効果」に追加）

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **TriggerOn** | はい | トリガータイミング：`OnPlay`（プレイ時、デフォルト）/ `OnDraw`（ドロー時）/ ... `TriggerOn` enum 参照。 |
| **PrefixBattleTagSetting** | はい | 前置の「戦闘タグ」設定 — メイン効果はこれらのタグが発動するときのみ走る。 |

加えて「組み合わせ効果」基底の：**OverrideDescription** / **説明** / **組み合わせ設定（子設定リスト）**。

## 挙動
*   `CombineSettings` は `PrefixBattleTagSetting` を子設定リストの**最前面**に「条件」として配置し、後続に元の `組み合わせ設定` リストを並べる。
*   説明：「**もし {前置タグ}、なら {メイン効果}**」（i18n key `IfCardConditions2` + `ConditionFitThenDes`）。
*   発動時：先に `PrefixBattleTagSetting` の各タグに対し `OnTriggerEffect(TriggerOn)` を呼び、**その後**残りの子設定を実行。
*   `GetBattleTags()` は **prefix タグを無視** — タグがメイン効果の一部として集約されるのを避ける。

## 注意点
*   **TriggerOn はカード実際のトリガー時点と整合**：カード自体が `OnPlay` で発動するのに、ここを `OnDraw` に設定するとメイン効果は永遠に走らない。
*   **PrefixBattleTagSetting が空**：「前置条件なし」と等しく、説明が「もし 、なら …」となる — データエラー。
*   **通常の「条件判断」との違い**：「条件判断」は `RCG_Condition` で適合判定する；本設定は**特定トリガータイミング + タグ**を前置条件にする — セマンティクスは精密だが用途は狭い。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_BattleTagCombineSetting.cs`
*   **継承元**：`RCG_CombineSetting`
*   **i18n クラス名 key なし**：エディタ表示は stripped name `BattleTagCombine`

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_TriggerOn` | `TriggerOn` | `TriggerOn` (enum) | — | デフォルト `OnPlay` |
| `m_PrefixBattleTagSetting` | `PrefixBattleTagSetting` | `RCG_BattleTagSetting` | — | ネストタグ設定 |
| (継承) `m_OverrideDescription` / `m_Description` / `m_CombineSettings` | 「組み合わせ効果」と同じ | — | — | 親クラス参照 |

### A.3 主なメソッド
*   **`CombineSettings` (override)**：`m_PrefixBattleTagSetting` を最前面に挿入 + base.CombineSettings。
*   **`GetBattleTags`** (override)：**prefix をスキップ**、`m_CombineSettings.GetEnableBattleSettings()` のタグのみ集約。
*   **`AddAction`**：`i==0`（prefix）に対しては各 BattleTag の `OnTriggerEffect(m_TriggerOn, iData)` を呼ぶ；その他は通常の `AddAction(InsertInOrder)`。
*   **`GetDescriptionFormat / GetDescription / GetDescriptionShort`**：`IfCardConditions2` + `ConditionFitThenDes` で prefix 説明とメイン説明を包む。

### A.4 他システムとの連携
*   **`RCG_BattleTagSetting`**：prefix コンテナとして使用。
*   **`RCG_EffectTriggerOn.GetEffectTriggerOn(TriggerOn)`**：`TriggerOn` enum を発動可能オブジェクトに変換。
*   **i18n keys**：`IfCardConditions2` / `ConditionFitThenDes`。

### A.5 既知の課題
*   `AllTypes` 以外で i18n 登録がないため、サブクラスドロップダウンに stripped name で表示される。
