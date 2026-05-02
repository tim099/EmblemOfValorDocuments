---
title: ドロー
description: 牌山 / 捨て札からカードを抽出；カードタグ限定可能
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# ドロー

> クラス名：`RCG_CardDrawSetting`

## 用途
カードを**ドロー**して手札に追加。**牌山頂点**（通常ドロー）、**牌山選択**（プレイヤーが選ぶ）、**捨て札選択**（拾う）の 3 ソースに対応。

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **DrawCardNum** | はい | ドロー枚数（変数可）。実際は `[0, プレイヤー手札空き]` にクランプ。 |
| **DrawType** | はい | ドロー方式：<br>• **FromDeckTop** — 牌山頂点からドロー（通常、デフォルト）<br>• **PickFromDeck** — 牌山から**選択**してドロー<br>• **PickFromDiscardPile** — 捨て札から**選択**してドロー（拾い） |
| **CardTags** | いいえ | 限定タグ（例：「治療カード」）。 |
| **DrawCardNotIncludedTags** | いいえ | 反対：これらのタグを**含まない**カードをドロー。 |

## 挙動
*   `FromDeckTop` は直接 N 枚を手札にドロー、**手札が満杯なら超過分は破棄**。
*   `Pick*` モードは選択 UI を出してプレイヤーに N 枚選ばせる。
*   説明形式：「**{枚数} 枚の {タグ} カードをドロー**」（i18n key `DrawCardIcon_{DrawType}`）。

### カードフュージョン
2 枚の「ドロー」カードをフュージョンすると枚数加算（**DrawType / CardTags の同一性はチェックしない**、要注意）。

## 注意点
*   **手札空き上限**：`DrawCardNum` は `RCG_Player.Ins.CardSpace` でクランプされるため、ドロー予定枚数が空きを超える分は捨てられて捨て札イベントもトリガーされません。
*   **タグフィルタと牌山枯渇**：限定タグドロー時、牌山に該当カードがない場合**捨て札を自動シャッフルしない**ため、N より少なくドローされる可能性。
*   **PickFrom* モードは AI 無効**：選択 UI はプレイヤー専用；敵がこの設定をトリガーすると無動作（AI は `FromDeckTop` を推奨）。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardDrawSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **`[System.Serializable]`** 標記
*   **i18n クラス名 key**：`RCG_CardDrawSetting` → 「ドロー」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_DrawCardNum` | `DrawCardNum` | `IntVariable` | — | `[UCL_FieldOnGUI]` |
| `m_DrawType` | `DrawType` | `DrawType` (内部 enum) | — | 3 種ソース |
| `m_CardTags` | `CardTags` | `List<RCG_CardTagGenData>` | — | |
| `m_DrawCardNotIncludedTags` | `DrawCardNotIncludedTags` | `bool` | — | |

### A.3 主なメソッド
*   **`AddAction`**：`Mathf.Clamp(m_DrawCardNum, 0, RCG_Player.Ins.CardSpace)` → `m_DrawType` で分岐：
    *   `FromDeckTop` → `CreateAction.AddDrawCardAction(iData, ...)`。
    *   `PickFromDeck` → `CreateAction.AddSelectCardAction(CardPos.Deck, ...)` + `AddPickFromDeckAction`。
    *   `PickFromDiscardPile` → 同上だが `CardPos.DiscardPile`。
*   **`Fusion(other)`**：clone + `IntVariable.FuseAdd(m_DrawCardNum)`、**他のフィールドはチェックしない**。
*   **`LocalizeKey`** → `"DrawCardIcon_" + m_DrawType.ToString()`。

### A.4 他システムとの連携
*   **`SelectCardSetting`**：`RCG_CardDiscardSetting.cs` で定義された helper、タグフィルタに使用。
*   **`CreateAction.AddSelectCardAction / AddPickFromDeckAction / AddDrawCardAction`**：ドロー Action ビルダー。
*   **`RCG_Player.Ins.CardSpace`**：手札空き上限。
