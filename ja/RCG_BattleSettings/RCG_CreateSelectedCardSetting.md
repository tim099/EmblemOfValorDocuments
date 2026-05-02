---
title: カード生成(複数選択)
description: 候補リストからプレイヤーに選ばせて生成（または牌山からランダム選択）；追加位置と枚数を指定可能
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# カード生成(複数選択)

> クラス名：`RCG_CreateSelectedCardSetting`

## 用途
**候補リスト**または**牌山**からプレイヤーに選ばせてカード生成 — 「**カード生成**」より「**多択から選ぶ**」段階が追加。よくある用途：
*   「3 つのランダム報酬から 1 つ選んで手札に追加」
*   「牌山全体から 1 枚選んで手札に追加」

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **CreateType** | はい | 候補ソース：<br>• **Default** — `CardGenDatas` で設定したリストから選択<br>• **FromDeck** — 指定牌山から選択 |
| **CreateCardType** | はい | 追加位置（「カード生成」と同じ）：手札 / 牌山 / 捨て札 / 牌山頂点。 |
| **CreateCount** | はい | プレイヤーが最大選択可能な枚数。 |
| **IsRandomSelect** | — | チェック = **ランダム**選択（UI なしで直接抽出）；チェックなし = プレイヤーが手動選択。 |
| **CardGenDatas** | CreateType=Default | 候補カードリスト（複数のカードテンプレート）。 |
| **Deck** | CreateType=FromDeck | 候補牌山データ（`RCG_DeckGenData`）。 |

## 挙動
*   `Default` + プレイヤー選択 → `CardGenDatas` から選択 UI を出してプレイヤーに N 枚選ばせる。
*   `FromDeck` + プレイヤー選択 → 指定牌山の全カードから選択 UI。
*   `IsRandomSelect = true` → UI を出さず、N 枚をランダム抽出。
*   選択完了後、`CreateCardType` に応じて指定位置に追加。

## 注意点
*   **「カード生成」との違い**：「カード生成」は固定カードを直接生成；本設定には**プレイヤー選択**段階がある（`IsRandomSelect` のとき除く）。
*   **CardGenDatas の重複処理**：候補リスト内で重複するカードは `Infos` で自動的に重複排除；ただし選択 UI には複数回表示される可能性あり。
*   **AI トリガー**：プレイヤー選択 UI は AI には無効；AI には `IsRandomSelect = true` を推奨（フリーズ回避）。
*   **FromDeck で牌山全体を引く**：候補が大量になり UI が混雑することも — 事前にフィルタリングを検討。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CreateSelectedCardSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_CreateSelectedCardSetting` → 「カード生成(複数選択)」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_CreateType` | `CreateType` | enum (内部) | — | `Default` / `FromDeck` |
| `m_CreateCardType` | `CreateCardType` | `CreateCardType` (共有) | — | 4 種位置 |
| `m_CreateCount` | `CreateCount` | `IntVariable` | — | デフォルト 1 |
| `m_IsRandomSelect` | `IsRandomSelect` | `bool` | — | |
| `m_CardGenDatas` | `CardGenDatas` | `List<RCG_CardGenData>` | — | `[Conditional("m_CreateType", false, Default)]` |
| `m_Deck` | `Deck` | `RCG_DeckGenData` | — | `[Conditional("m_CreateType", false, FromDeck)]` |

### A.3 主なメソッド
*   **`Infos`**：`Default` モードは hash で重複排除後に各カード info を追加；`FromDeck` モードは deck info を追加。
*   **`CardDatas` (private)**：CreateType に応じて候補 `RCG_CardData` リストを取得。
*   **`AddAction`**（ファイル 90 行外）：選択 + 追加位置（IsRandomSelect で分岐）。

### A.4 他システムとの連携
*   **`RCG_DeckGenData`**：牌山データソース。
*   **`CreateAction.AddSelectCardAction / CreateCard`**：UI 選択とカード挿入エントリ。
