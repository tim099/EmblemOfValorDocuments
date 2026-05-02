---
title: カードフュージョン
description: プレイヤーが手札を選んでフュージョンし、新カードを生成；元のカードは消滅
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# カードフュージョン

> クラス名：`RCG_CardFusionSetting`

## 用途
**プレイヤーが手札を選んでフュージョン**：選んだカードは消滅し、新しい融合カードが手札に追加されます。よくある用途：
*   「錬金術師」風の合成カード
*   「2 枚の攻撃カードを強化攻撃に合成」

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **SelectHandCardSetting** | はい | ネスト選択設定；選択 UI の条件、枚数、制限を決定。 |

## 挙動
*   ネスト `SelectHandCardSetting` を先にトリガーしてプレイヤーに選択させる。
*   選択完了後：
    1. 選択カードに対し `CardFusion()` を呼んでフュージョン後の `RCG_CardData` を計算。
    2. フュージョン結果から新 `RCG_CardBattleData` を生成。
    3. 選択カードを全て **`Banish` 消滅**（捨て札ではない！）。
    4. 新カードを手札に追加。
*   説明形式：「**選択カードをフュージョン**」（i18n key `CardFusion_Des`）。

## 注意点
*   **フュージョンロジックは `CardFusion()` が決定**：実際の融合ルールは**カードシステム層**にあり（本設定ではない）。2 枚のカードを融合すると何ができるかは `RCG_CardData` / `IList<RCG_CardData>.CardFusion()` を参照。
*   **元カードは消滅**：`RemoveType.Banish` であり捨て札にはならず、**捨て札関連の効果はトリガーしない**。
*   **フュージョン候補がない場合**：選択 UI は出るが選択肢なし；プレイヤーはキャンセルのみ可能。`SelectHandCardSetting` で適切な最低枚数を設定してください。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardFusionSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_CardFusionSetting` → 「カードフュージョン」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_SelectHandCardSetting` | `SelectHandCardSetting` | `RCG_SelectHandCardSetting` | — | |

### A.3 主なメソッド
*   **`AddAction`**：`m_SelectHandCardSetting.AddAction(InsertInOrder)` を先に呼び、後続 ActionTrigger 内：
    1. `iData.SelectedHandCards` → `RCG_CardData` リストを集めて `aDiscardCardList` に追加。
    2. `aSelectedCards.CardFusion()` → 融合された `RCG_CardData` を取得。
    3. `RCG_CardBattleData.CreateCard(aFusionCardData, false)` で新 battle data を生成。
    4. `CreateAction.AddDiscardCardActions(..., RemoveType.Banish)` で元カードを消滅。
    5. `CreateAction.CreateCard(aNewCard, CreateCardType.AddToHandCard)` で手札に追加。
*   **`GetDescriptionFormat`**：i18n key `CardFusion_Des`、`m_SelectHandCardSetting.GetDescriptionFormat` を含む。

### A.4 他システムとの連携
*   **`IList<RCG_CardData>.CardFusion()`**：実際のフュージョンロジック（拡張メソッド）。
*   **`RCG_SelectHandCardSetting`**：選択 UI トリガー。
*   **`CreateAction.AddDiscardCardActions / CreateCard`**：消滅 + 生成の Action ビルダー。
