---
title: 捨て札
description: 手札を移動（捨て札・消滅・牌山の頂点・保留など）；複数の選択方式とタグフィルタに対応
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 捨て札

> クラス名：`RCG_CardDiscardSetting`

## 用途
**手札を現在の位置から指定の場所へ移動**（捨て札 / 消滅 / 牌山の頂点など）。「捨て札カード」「消滅カード」のコアな設定。

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **DiscardType** | はい | 捨て札方式：<br>• **DiscardByCount** — プレイヤーが N 枚選択<br>• **DiscardAll** — 全手札<br>• **DiscardAnyCount** — プレイヤーが任意枚数<br>• **RandomDiscardByCount** — ランダム N 枚<br>• **SelectedCards** — 既に前段で選択されたカード<br>• **SelectHandCard** — ネスト選択設定で選択 |
| **DiscardCardNum** | DiscardByCount/RandomDiscardByCount 必須 | 捨て札枚数（変数可）。`Conditional` で表示制御。 |
| **RemoveType** | はい | 移動方式：`Discard`（捨て札）/ `Banish`（消滅）/ `ToDeckTop`（牌山の頂点）/ `TurnEndDiscard`（ターン終了時に捨て札、捨て札効果はトリガーしない）/ `Retain`（手札に保留）。 |
| **SelectHandCardSetting** | DiscardType=SelectHandCard 必須 | ネストした選択設定。 |
| **DiscardCardTags** | いいえ | 対象タグ限定（例：「攻撃カード」）。 |
| **DiscardCardNotIncludedTags** | いいえ | 反対選択（これらのタグを含まないカード）。 |
| **DiscardVariable** | いいえ | 実際に捨てた枚数を**変数に書き込み**、後続設定が参照可能（例：「捨て札後 X 枚ドロー」の X）。 |

## 挙動
*   DiscardType に応じて捨て札対象リストを取得し、`CreateAction.AddDiscardCardActions(...)` を `RemoveType` 指定で呼ぶ。
*   `DiscardVariable` ありの場合、枚数を `iData.VariableDic[変数名]` に保存。
*   `DiscardAnyCount` モードで `DiscardVariable` 設定ありの場合、説明には `N` でなく `X` を表示。
*   説明は `RemoveType` で異なる i18n key を使う（例：`BanishCardIcon_Des` / `DiscardSelectedCardsIcon_Des`）。

### カードフュージョン
`DiscardByCount` と `RandomDiscardByCount` の間でのみフュージョン可能（枚数加算）；他のモードはフュージョン不可。

## 注意点
*   **TurnEndDiscard は捨て札効果をトリガーしない**：「カードを手札にロック、ターン終了で強制クリア」のシーン用に意図設計；通常の捨て札には使わないで。
*   **DiscardAnyCount に DiscardCardNum なし**：上限を入れたい場合は `DiscardByCount` を；任意枚数を保持したい場合は空にして `DiscardVariable` を活用。
*   **変数命名衝突**：複数設定が `DiscardVariable = "X"` を使うと後者が前者を上書き；明確に命名を。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardDiscardSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_CardDiscardSetting` → 「捨て札」
*   **同ファイル内ネストクラス**：`SelectCardSetting`（`RCG_CardTagGenData` フィルタ対応、`m_NotIncludedTags` 反転と `m_WithoutEnhancement` を含む）

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_DiscardType` | `DiscardType` | enum | — | 6 モード |
| `m_DiscardCardNum` | `DiscardCardNum` | `IntVariable` | — | `[Conditional(m_DiscardType, false, DiscardByCount, RandomDiscardByCount)]` |
| `m_RemoveType` | `RemoveType` | `RemoveType` (内部 enum) | — | `Discard` / `Banish` / `ToDeckTop` / `TurnEndDiscard` / `Retain` |
| `m_SelectHandCardSetting` | `SelectHandCardSetting` | `RCG_SelectHandCardSetting` | — | `[Conditional(m_DiscardType, false, SelectHandCard)]` |
| `m_DiscardCardTags` | `DiscardCardTags` | `List<RCG_CardTagGenData>` | — | |
| `m_DiscardCardNotIncludedTags` | `DiscardCardNotIncludedTags` | `bool` | — | |
| `m_DiscardVariable` | `DiscardVariable` | `string` | — | |

### A.3 主なメソッド
*   **`AddAction`**：`m_DiscardType` で 6 種分岐；共通パスは `CreateAction.AddDiscardCardActions(iData, list, mode, RemoveType)`。
*   **`Fusion`**：`DiscardByCount` / `RandomDiscardByCount` モードのみフュージョン可能；clone + `IntVariable.FuseAdd` で枚数を統合。
*   **`GetDescriptionFormat`**：RemoveType / DiscardType の二重分岐で i18n key を切替（例：`Banish{Type}Icon_Des`）。

### A.4 他システムとの連携
*   **`CreateAction.AddSelectCardAction / AddDiscardCardActions`**：UI 選択と捨て札 Action のビルダー。
*   **`RCG_GameManager.Random.RandomPick`**：RandomDiscard のランダムソース。
*   **`SelectCardSetting`**：タグフィルタの helper（同ファイル内）。
