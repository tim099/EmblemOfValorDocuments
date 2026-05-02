---
title: コピー
description: 選択した手札のコピーを生成；手札 / 牌山 / 捨て札への追加位置を指定可能
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# コピー

> クラス名：`RCG_CopySetting`

## 用途
**選択した手札をコピー**して**新しい複製**を生成し、設定された場所（手札・牌山・捨て札）に追加。よくある用途：
*   「このカードをコピーして手札に」
*   「選択カードを牌山頂点にコピー」

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **SelectHandCardSetting** | はい | コピー対象を選ぶ設定（`SkipSelect` / `ThisCard` / 通常選択などのモードに対応）。 |
| **CreateCardType** | はい | 複製の追加位置：手札 / 牌山 / 捨て札 / 牌山頂点。 |

## 挙動
*   `SelectHandCardSetting` を先にトリガーし、選択された各カードに対し `CopyCard()` で複製を生成、`CreateAction.CreateCard(複製, CreateCardType)` で指定位置に挿入。
*   説明は `SelectType` に応じて 3 種類変化：
    *   `SkipSelect` → 直接「コピー」と表示
    *   `ThisCard` → 「このカードをコピー」と表示
    *   他 → 「{選択描述} とコピー」
*   `CreateCardType ≠ AddToHandCard` のとき位置説明を追加。

## 注意点
*   **複製はチャント状態を継承しない**：コピー後は新カード；元カードの詠唱進度は引き継がれない（特別処理が無い限り）。
*   **強化されたカードのコピー**：`CopyCard()` は強化効果も含む；複製も強化版（**ベース版に降格しない**）。
*   **CreateCardType = AddToDeckTop**：プレイヤーはアニメを見れない；無効果と誤解されるかも。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CopySetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_CopySetting` 文字列で description 用に使用（「コピー」）；`AllTypes` ドロップダウンには stripped name `Copy` で表示

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_SelectHandCardSetting` | `SelectHandCardSetting` | `RCG_SelectHandCardSetting` | — | `[SerializeField] protected` |
| `m_CreateCardType` | `CreateCardType` | `CreateCardType` (enum) | — | `RCG_CardCreateSetting.CreateCardType` を共有 |

### A.3 主なメソッド
*   **`AddAction`**：選択 → 各カードに対し `aCard.CopyCard()` で複製 → `CreateAction.CreateCard(複製, m_CreateCardType, InsertInOrder)`。
*   **`Infos`**：`RCG_CopySetting` + `CopySettingInfo` の i18n。
*   **`GetDescriptionFormat`**：`m_SelectHandCardSetting.m_SelectType` で 3 分岐処理。

### A.4 他システムとの連携
*   **`RCG_CardBattleData.CopyCard()`**：実際の複製生成メソッド。
*   **`CreateAction.CreateCard`**：複製を挿入する Action。
*   **`RCG_SelectHandCardSetting`**：選択トリガー。
