---
title: カード生成
description: 指定カードを生成して手札・牌山・捨て札に追加；生成枚数を指定可能
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# カード生成

> クラス名：`RCG_CardCreateSetting`

## 用途
**新カードを生成**して指定位置に追加。よくある用途：
*   「燃焼カード 2 枚を捨て札に追加」
*   「強化版カードを手札の頂点に置く」
*   「挑発カード 1 枚を牌山に生成」

プレイヤーに**多選択肢から選ばせたい**場合は「**カード生成(複数選択)**」(`RCG_CreateSelectedCardSetting`) を使ってください。

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **CardData** | はい | 生成するカードテンプレート（`RCG_CardGenData`）。 |
| **IsChanted** | チャント中のカードのみ表示 | 詠唱済み状態か（チャントカード限定）。 |
| **CreateCardType** | はい | 追加位置：<br>• **AddToHandCard** — 手札に追加<br>• **AddToDeck** — 牌山ランダム位置（デフォルト）<br>• **AddToDiscardPile** — 捨て札に追加<br>• **AddToDeckTop** — 牌山の頂点 |
| **CreateCount** | はい | 生成枚数；変数バインド可能。 |

## 挙動
*   説明形式：「**{枚数} 枚の {カード名} を生成**」（i18n key `CardCreateEffectDes_{CreateCardType}`）。
*   実行時に各 1 枚ずつ `RCG_CardBattleData.CreateCard(テンプレート, IsChanted)` を呼んで指定位置に挿入。
*   生成カードは `iData.m_TriggerData.m_CreatedCards` に記録され、後続の設定が参照可能。

### カードフュージョン
2 枚の「カード生成」をフュージョンすると、**生成枚数が加算**（例：「1 枚生成」+「2 枚生成」→「3 枚生成」）。

## 注意点
*   **CardData が存在しない ID を指す場合**：実行時 `CardData.GetData()` が null となり、カード名が空文字に。ID が登録済みか確認を。
*   **チャントカード（IsChant）の IsChanted フィールド**：CardData がチャントカードのときのみ表示（リフレクション `IsChantCard()` で判定）。
*   **AddToDeckTop の視覚的フィードバック**：プレイヤーはアニメを見れず、次のドロー時に出てくるだけ；強く演出したいなら AddToHandCard を。
*   **「カード生成(複数選択)」との違い**：本設定は**プレイヤーに選ばせる段階がない**ため直接生成；複数選択版は選択 UI が出る。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardCreateSetting.cs`
*   **継承元**：`RCG_BattleSetting`、実装 `UCLI_FieldOnGUI`
*   **`[System.Serializable]`** 標記
*   **i18n クラス名 key**：`RCG_CardCreateSetting` → 「カード生成」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_CardData` | `CardData` | `RCG_CardGenData` | — | テンプレート |
| `m_RuntimeCardData` | （非表示） | `RCG_CardBattleDataPointer` | — | `[UCL_HideOnGUI]`；アイテム化用参照 |
| `m_IsChanted` | `IsChanted` | `bool` | — | `[Conditional(nameof(IsChantCard))]` チャントカードのみ表示 |
| `m_CreateCardType` | `CreateCardType` | `CreateCardType` (enum) | — | 4 種追加位置 |
| `m_CreateCount` | `CreateCount` | `IntVariable` | — | `[UCL_FieldOnGUI]` |

### A.3 主なメソッド
*   **`CardData` (property)**：`m_RuntimeCardData` を優先、なければ `m_CardData.GetData()` にフォールバック。
*   **`OnGUI`**：自前のフィールド描画、`UCL_GUILayout.DrawField(this, ..., RCG_StaticFunctions.LocalizeFieldName)` を呼ぶ。
*   **`AddAction`**：`m_CreateCount` 分ループし `RCG_CardBattleData.CreateCard(aCardData, m_IsChanted)` で生成、`iData.m_TriggerData.m_CreatedCards` に記録、`CreateAction.CreateCard(aCard, m_CreateCardType)` でキュー追加。
*   **`Fusion(other)`**：clone + `IntVariable.FuseAdd` で `m_CreateCount` を統合；`m_CardData` の同一性は**チェックしない**（**異種カードがフュージョンする可能性**あり）。
*   **`IsChantCard()` (private)** → リフレクション用；`Conditional` で `IsChanted` 表示判定に使用。
*   **`SerializeToJson / DeserializeFromJson`**：`m_RuntimeCardData` のシリアライズロジック保持（コメントアウト中、現在は base 既定使用）。

### A.4 他システムとの連携
*   **`RCG_CardBattleData.CreateCard(...)`**：カードインスタンス生成。
*   **`CreateAction.CreateCard`**：実際の Action class。
*   **`RCG_CardBattleDataPointer`**：runtime カード参照器；アイテム化（アイテム内でカードを生成する仕組）に使用。
*   **`RCG_StaticFunctions.LocalizeFieldName`**：フィールド名 i18n コールバック。

### A.5 既知の課題
*   `m_RuntimeCardData` のシリアライズロジックがコメントアウト（base 既定使用）；このフィールドの永続化が必要か要確認。
*   `Fusion` が `m_CardData` の一致をチェックしないため、「X を生成」+「Y を生成」が「X を 2 枚生成」（前者の CardData を使う）にフュージョンされる理論的可能性あり — 設計意図の検証が必要。
