---
title: カード効果無効化 (Diminish)
description: 選択した手札を弱体化：指定数の葉効果を取り除き、強化効果を優先消費
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# カード効果無効化 (Diminish)

> クラス名：`RCG_DiminishSetting`

## 用途
**選択した手札を弱体化** — 一定数の「葉効果」（最末端の単一動作、例：「攻撃」「治療」「ドロー」）を取り除く。優先的に強化効果から消費し、使い切ったらベース効果に進む。よくある用途：
*   敵スキル：「あなたの次のカードの効果を半減」
*   呪いカード：「対象カードのある効果を消す」

弱体化された葉効果は `RCG_DiminishedPlaceholder` で置換され、説明には**赤字「無効化済み」**が表示。

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **SelectHandCardSetting** | はい | 弱体化対象を選ぶ設定。 |
| **DiminishEffectCount** | はい | 取り除く葉効果数（デフォルト 1）。 |

## 挙動
*   選択完了後、各カードに対し `aCard.Diminish(token, DiminishEffectCount)` を呼ぶ。
*   消費順序：**強化効果**を優先消費し、使い切ったら**ベース効果**へ。
*   消費される各葉効果は `RCG_DiminishedPlaceholder` で置換、説明には「無効化済み」（赤字）が表示。

## 注意点
*   **さらなる弱体化不可**：`RCG_DiminishedPlaceholder` で置換された葉効果は**再度弱体化できない**（無限スタックを回避）。
*   **葉効果は明確とは限らない**：「組み合わせ効果」「条件判断」内には複数の葉効果がある；`DiminishEffectCount = 1` が「最も重要なものを削れる」とは限らない。
*   **DiminishEffectCount が大きすぎる**：カードの葉効果総数を超えると、ほぼ全部が placeholder になる — 設計時に調整を。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_DiminishSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_DiminishSetting` 登録あり（値は空、エディタ表示は stripped name）

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_SelectHandCardSetting` | `SelectHandCardSetting` | `RCG_SelectHandCardSetting` | — | `[SerializeField] protected` |
| `m_DiminishEffectCount` | `DiminishEffectCount` | `int` | — | `[SerializeField] protected`、デフォルト 1 |

### A.3 主なメソッド
*   **`AddAction`** (async)：選択 → 各カードに対し `aCard.Diminish(iToken, m_DiminishEffectCount)`。
*   **`GetBattleTags`** → 常に空リスト（弱体化されたカードはタグを継承しない）。
*   **`GetDescription`**：「{選択描述}\n{弱体化説明} ({DiminishEffectCount}個)」（i18n key `DiminishSettingDes`）。
*   **`GetDescriptionParams`**：`Title`、`DC`（DiminishEffectCount 文字列）、選択設定の子 params を含む。

### A.4 他システムとの連携
*   **`RCG_CardBattleData.Diminish(token, count)`**：実際の弱体化エントリ；強化端から葉効果を消費し placeholder で置換。
*   **`RCG_DiminishedPlaceholder`**：占位クラス。
*   **`RCG_Extensions.TagColors.EnhenceTitle`**：タイトル色（強化系から流用）。
