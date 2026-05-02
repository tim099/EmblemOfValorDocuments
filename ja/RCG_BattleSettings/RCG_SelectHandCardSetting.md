---
title: 手札選択
description: カード選択 UI を開く（または自動選択）；結果を SelectedHandCards に書き込み後続設定が参照可能
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 手札選択

> クラス名：`RCG_SelectHandCardSetting`

## 用途
**プレイヤー（またはシステム）に手札を選択させる** — 結果を `SelectedHandCards` に保存し、後続設定（捨て札、強化、コピー、フュージョン、カード→アイテム など）が利用可能に。「**前置設定**」性格の設定。

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **SelectType** | はい | 選択方式（**13 種**、最常用は下記）。 |
| **SelectRange** | SelectFromRange* 必須 | 選択範囲（手札 / 牌山 / 捨て札、複数選択可）。 |
| **SelectNum** | SelectByCount/RandomSelect/RandomOneCardWithCost/LeftSide(Of)/RightSide 必須 | 選択枚数（またはコスト値）。 |
| **CardTags** | いいえ | 限定タグのカード。 |
| **SelectCardNotIncludedTags** | — | 反対選択（これらのタグを含まないカード）。 |
| **SelectCardWithoutEnhancement** | — | 未強化のカードのみ選択。 |
| **SelectCountVariable** | いいえ | 選択枚数を変数に書き込み、後続設定が参照可能に。 |
| **DetailSetting** | いいえ | 説明文の微調整（`Default` / `Then` / `SelectPrefix`）。 |

### 主要な SelectType
| 値 | 動作 |
|---|---|
| **SelectByCount** | プレイヤーが N 枚選択（最常用、デフォルト） |
| **SelectAnyCount** | プレイヤーが任意枚数 |
| **SelectAll** | 自動全選択（UI なし） |
| **SkipSelect** | 選択をスキップ（前段で既に選択済み） |
| **CreatedCards** | 自動的に本効果中で生成されたカードを選択 |
| **RandomSelect** | ランダムに N 枚選択 |
| **SelectFromRange** | 手札 / 牌山 / 捨て札の混合範囲から選択 |
| **SelectFromRangeAll** | 多範囲から**全選択** |
| **ThisCard** / **TriggeredCard** | 発動中のこのカード |
| **LeftSide** / **RightSide** | このカードの左 / 右側 N 枚 |
| **LeftSideOfThis** | このカードの左側のカード |
| **RandomOneCardWithCost** | コスト = X のカードをランダムに 1 枚選択（X は `SelectNum` で指定） |

## 挙動
*   **実行フロー**：選択 UI を開く（またはモードに応じて自動選択）、結果を `iData.SelectedHandCards` に書き込み。
*   **後続参照**：下流設定（捨て札、強化など）が `iData.SelectedHandCards` を読む。
*   **変数書込**：`SelectCountVariable` 非空のとき、実際の選択枚数をこの変数に書き込み。
*   **説明**：`DetailSetting.DescriptionType` で文型を決定。

## 注意点
*   **本設定は単独使用しない**：「選択」して値を書くだけ；選択カードに何かするには**後段に**捨て札 / 強化 / コピー / フュージョン / ... 等を続けて。
*   **AI トリガー**：プレイヤー選択 UI は AI に無効；AI ではこの設定で `RandomSelect` / `SelectAll` / `SkipSelect` などプレイヤー入力不要のモードを使用。
*   **SelectFromRange の空範囲リスト**：「選択しない」と等価。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_SelectHandCardSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_SelectHandCardSetting` → 「手札選択」
*   **同ファイル内ネストクラス**：`SelectType` (13 値) / `DescriptionType` (3 値) / `DetailSetting`（`m_DescriptionType` を含む）

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_SelectType` | `SelectType` | enum | — | 13 モード |
| `m_SelectRange` | `SelectRange` | `List<CardPos>` | — | `[Conditional(nameof(m_SelectType), false, SelectFromRange, SelectFromRangeAll)]` |
| `m_SelectNum` | `SelectNum` | `IntVariable` | — | `[Conditional(... 6 種要枚数の SelectType)]` |
| `m_CardTags` | `CardTags` | `List<RCG_CardTagGenData>` | — | |
| `m_SelectCardNotIncludedTags` | `SelectCardNotIncludedTags` | `bool` | — | |
| `m_SelectCardWithoutEnhancement` | `SelectCardWithoutEnhancement` | `bool` | — | |
| `m_SelectCountVariable` | `SelectCountVariable` | `string` | — | |
| `m_DetailSetting` | `DetailSetting` | `DetailSetting` (内部) | `DetailSetting` | |

### A.3 主なメソッド
*   **`AddAction`**（ファイル 100+ 行外）：`m_SelectType` の 14 種分岐 — UI 開 / 自動選 / 範囲選 / ランダム選など、最終的に `iData.SelectedHandCards` に書き込み。
*   **`TagDes` (private)**：タグ説明生成；`NotIncludedDes` と `SelectCardWithoutEnhancementDes` の修飾を含む。
*   **`GetDescriptionFormat`** / **`GetDescriptionParams`**：`m_DetailSetting.m_DescriptionType` で文型を切替。

### A.4 他システムとの連携
*   **`iData.SelectedHandCards`**：書込対象；下流の `IntVariable` 解析が参照。
*   **`CardPos` enum**：選択範囲タイプ（Hand / Deck / DiscardPile）。
*   **`CreateAction.AddSelectCardAction`**：UI 選択のエントリ。
