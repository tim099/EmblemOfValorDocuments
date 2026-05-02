---
title: 手札コスト変化
description: 指定範囲の手札に対しコスト増加 / 減少 / ゼロ化
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 手札コスト変化

> クラス名：`RCG_CardCostAlterSetting`

## 用途
**手札コストを変更**。よくある用途：
*   「全手札のコスト -1」
*   「ランダムな手札 1 枚を 0 コストに」
*   「コスト最高の手札をゼロに」

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **CardAlterRange** | はい | 影響範囲：<br>• **AllHandCards** — 全手札<br>• **SelectedCards** — プレイヤーが事前に選んだ手札<br>• **OneRandomHandCard** — ランダム 1 枚<br>• **ThisCard** — このカード自身（カードトリガー時のみ）<br>• **MostCostCard** — コスト最高のカード<br>• **NewSelectedCards** — この効果内で選択（選択設定が自動展開） |
| **CostAlterType** | はい | 変化モード：<br>• **AddHandCardCost** — コスト増加<br>• **SubHandCardCost** — コスト減少<br>• **HandCardCostToZero** — 0 コスト化（数値不要） |
| **SelectHandCardSetting** | NewSelectedCards 必須 | ネストした選択設定；他モードでは自動非表示。 |
| **コスト** (`Cost`) | AddHandCardCost / SubHandCardCost 必須 | 変化数値（0 コスト化では無視）。 |

## 挙動
*   **NewSelectedCards** モードはネスト選択設定をトリガーし、プレイヤーの選択完了後に実行。
*   範囲でカードを抽出後、CostAlterType を適用：
    *   増加 → `AlterCost(+Cost)`
    *   減少 → `AlterCost(-Cost)`
    *   ゼロ化 → `AlterCost(-現コスト)`（直接相殺）

### カードフュージョン
同 `CostAlterType` の設定をフュージョンすると `Cost` を加算。**異なる CostAlterType 間ではフュージョン不可**（「+1 とゼロ化」のセマンティクス衝突回避）。

## 注意点
*   **OneRandomHandCard** は即時ランダム抽選；事前抽出ではないため、複数回トリガーで毎回違うカードが当たる可能性。
*   **ThisCard は非カードトリガー場面で無効**：状態効果トリガー時など `iData.Card == null` のとき。
*   **HandCardCostToZero は「コスト -∞」ではない**：現コストから現値を引いて 0 にする；**+1 コスト後にゼロ化すると 0 コストに**、逆方向の積み増しではない。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardCostAlterSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_CardCostAlterSetting` → 「手札コスト変化」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_CardAlterRange` | `CardAlterRange` | `CardAlterRange` (enum) | — | 6 種ソース |
| `m_CostAlterType` | `CostAlterType` | `CostAlterType` (enum) | — | 3 種モード |
| `m_SelectHandCardSetting` | `SelectHandCardSetting` | `RCG_SelectHandCardSetting` | — | `[Conditional(m_CardAlterRange, false, NewSelectedCards)]` |
| `m_Cost` | コスト | `IntVariable` | `Cost` | |

### A.3 主なメソッド
*   **`AddAction`**：`NewSelectedCards` モードはまず `m_SelectHandCardSetting.AddAction` を呼び、後続の ActionTrigger 内で範囲に応じてカードを取得し `AlterCost` を実行。
*   **`Fusion(other)`**：両者の `m_CostAlterType` が一致を要求；clone 後 `IntVariable.FuseAdd` で `m_Cost` を統合。
*   **`GetDescriptionFormat`**：CostAlterType に応じて i18n key を切替（`AddHandCardCost_Des` / `SubHandCardCost_Des` / `HandCardCostToZero_Des`）。

### A.4 他システムとの連携
*   **`RCG_Card.AlterCost(int)`**：実際にコスト修正するエントリ。
*   **`UCL_Random.Instance.Next`**：OneRandomHandCard のランダムソース。
