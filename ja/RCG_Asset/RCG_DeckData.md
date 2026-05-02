---
title: デッキデータ (RCG_DeckData)
description: 1つの完全なデッキの定義（カード一覧と配置）；キャラ初期デッキ、加入デッキ、プレイヤーデッキの基盤
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# デッキデータ

> クラス名：`RCG_DeckData`

## 用途

**1つの完全なデッキの定義**。各カード + 数量 = デッキ。応用シナリオ：
*   キャラ `RCG_CharacterData.m_Deck`（初期デッキ）
*   キャラ `m_JoinDeck`（中途加入時に持参するデッキ）
*   `RCG_BattlePresetData.m_Deck`（テスト戦闘用デッキ）
*   特殊イベント付与のデッキ一式

`RCG_Asset<RCG_DeckData>` を継承。実装：`RCGI_Unloackable`（解放可）。

## エディタ上の見た目

```
RCG_DeckData: <ID>
    Name           ← デッキ表示名（多言語）
    Deck           ← カード一覧（SpawnDeckData、各カード + 数量含む）
    Unlock         ← 解放条件
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Name** | いいえ | 表示名（多言語）；空白時は ID にフォールバック |
| **Deck** | はい | カード一覧と数量（`SpawnDeckData`） |
| **Unlock** | いいえ | 解放条件 |

## 動作説明

### `SelectCard(setting)`
`SelectCardSetting` に従ってデッキから**順次**最初の適合カードを選別（**非ランダム**）；命中で `RCG_CardBattleData` に clone し返却。

### `SelectCards(count, setting)`
TODO：未実装（プログラム内 `// ToDo` で直接 return null）。

### `GetAllCards()`
デッキ全体の `RCG_CardGenData` リストを返却。

### Tooltip Infos
`Infos = m_Deck.Infos`：全カード上の状態効果説明を集約（例：「出血」「燃焼」状態解説）。

## 注意事項

*   **`SelectCards` は未実装**：バッチランダム抽選には現状この入口は使えない；自前実装または別の utility を経由する必要。
*   **デフォルト ID `Default` / `BackUp`**：`RCG_DeckGenData.DefaultID = "Default"`、`BackUpID = "BackUp"` — 「初期デッキ」と「予備デッキ」の慣習的命名。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_DeckData.cs`
*   **継承**：`RCG_Asset<RCG_DeckData>`
*   **実装**：`RCGI_Unloackable`
*   **AssetGroup**：`EditItems`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Deck` | Deck | `SpawnDeckData` | カード一覧 + 配置 |
| `m_Unlock` | Unlock | `RCG_UnlockEntry` | |

### A.3 主要メソッド

*   **`SelectCard(setting)`** — 条件で順次1枚選別、`RCG_CardBattleData` に clone。
*   **`SelectCards(count, setting)`** — **未実装**（return null + TODO）。
*   **`GetAllCards()`** — デッキ全体リスト。
*   **`AllCardsName / Infos / LocalizedName`** — 表示用プロパティ。

### A.4 他システムとの連携

*   **`SpawnDeckData`** — 実デッキコンテナ。
*   **`RCG_CardBattleData`** — 戦闘用カードインスタンス。
*   **`SelectCardSetting`** — カード選択ルール。
*   **`RCG_DeckGenData`** — Asset Entry；`Default` / `BackUp` の2つの定数 ID。

### A.5 既知の問題

*   `SelectCards` は未実装（`// ToDo`）。
