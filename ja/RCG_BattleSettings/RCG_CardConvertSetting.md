---
title: カード変換
description: 指定カードを別の種類に変換（手札・選択中・このカードの 3 ソース）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# カード変換

> クラス名：`RCG_CardConvertSetting`

## 用途
指定したカードを**まとめて別のカードに変換**。よくある用途：
*   「手札を全部死者カードに変換」
*   「このカードを強化版に変換」
*   「選択カードを廃カードに変換」

システム内部でも使用 — キャラ死亡時、二度とプレイ不可となるカードを「死者カード」に自動変換。

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **ConvertType** | はい | 変換ソース：<br>• **SelectedCards** — プレイヤーが選んだ手札<br>• **AllHandCards** — 全手札<br>• **ThisCard** — この効果をトリガーしたカード自身 |
| **ConvertTarget** | はい | 変換先のカード（`RCG_CardGenData`）。 |

## 挙動
*   説明形式は `ConvertType` に応じて i18n key を切替（`CardConvert_SelectedCards` / `CardConvert_AllHandCards` / `CardConvert_ThisCard`）。
*   実行時：
    1. ConvertType に応じて変換対象カードリストを収集。
    2. 各カードに対し `aDeck.Convert(旧, 新)` で置換。
    3. **手札のカード**は変換アニメを再生（各 0.2 秒間隔）；他位置（牌山）は無アニメで直接置換。

## 注意点
*   **ThisCard はカードトリガー時のみ有効**：状態 / バトルイベントトリガーの場合「このカード」概念がなく対象が見つからない。
*   **SelectedCards は事前に選択済みである必要**：通常前段に「**手札選択**」設定を置いて `iData.SelectedHandCards` を埋めておきます。
*   **変換先 ID が存在しない場合 NRE**：`ConvertTarget` の指す ID が登録済みか確認を。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardConvertSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **`[System.Serializable]`** 標記
*   **i18n クラス名 key**：`RCG_CardConvertSetting` → 「カード変換」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_ConvertType` | `ConvertType` | `ConvertType` (enum) | — | `SelectedCards` / `AllHandCards` / `ThisCard` |
| `m_ConvertTarget` | `ConvertTarget` | `RCG_CardGenData` | — | 対象カードテンプレート |

### A.3 主なメソッド
*   **`AddAction`** (async)：
    1. `m_ConvertType` に応じて `aCards` を収集（手札 / 選択 / このカード）。
    2. 各カードに対し新 `RCG_CardBattleData` を clone し、`aDeck.Convert(旧, 新)` を呼ぶ。
    3. 手札位置のカードは `CardConvertAnim` アニメを再生（0.2 秒ずらし）；他はサイレント置換。
*   **`RemoveDeadUnitCards(triggerEffectData)` (static)**：キャラ死亡 hook — `!HaveRequireSkills(PlayerSkillTags)` のカードを見つけ、すべて `RCG_CardData.DeadCardID` に変換。
*   **`LocalizeKey`** → `"CardConvert_" + m_ConvertType.ToString()`、説明文は対応する i18n を引く。

### A.4 他システムとの連携
*   **`RCG_Player.Ins.Deck.Convert`**：実際のカード置換実行。
*   **`RCG_CardBattleData.CreateCard(target)`**：clone して新カードインスタンスを生成。
*   **`RCG_Card.CardConvertAnim`**：手札変換アニメ。
*   **`RCG_CardData.DeadCardID`**：死亡時自動変換のターゲット。
