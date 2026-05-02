---
title: 戦闘開始アクション (RCG_BattleStartActionData)
description: 戦闘開始時に発動する特殊行動：奇襲ダメージ、スタン、初期バフ等
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 戦闘開始アクション

> クラス名：`RCG_BattleStartActionData`

## 用途

**戦闘の最初に発動する特殊行動**。例：「奇襲」は敵の行動前に先に血を削り、「準備充分」は味方に初期バフを付与、「魔法陣」は turn 0 で AOE 解放。各戦闘は1つの `RCG_BattleStartActionData` を参照可能、内に `User`（誰が実行）+ 一連の `RCG_BattleSetting`（何をするか）。

`RCG_Asset<RCG_BattleStartActionData>` を継承。

## エディタ上の見た目

```
RCG_BattleStartActionData: <ID>
    User                  ← 実行者選択ルール
    BattleStartActions    ← 実行する BattleSetting 順序
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **User** | はい | 実行者選択ルール（`RCG_SelectTargetData`）。デフォルト Ally / All 範囲；空 → プレイヤー先頭にフォールバック |
| **BattleStartActions** | はい | 戦闘開始時に順次実行する `RCG_BattleSetting` 順序（ダメージ、バフ、抽選） |

## 動作説明

### `AddAction(TriggerEffectData)`
1. `m_User` でターゲット一覧取得。
2. 最初のターゲットを `iData.User` にする（無ければ → プレイヤー先頭）。
3. `m_BattleStartActions` を全て順次アクションキューに追加（`PushBack` モード）。

### デフォルト ID
`RCG_BattleStartActionGenData.DefaultID = "None"`：「開始アクションなし」を表す。

## 注意事項

*   **`User` は「実行者」、「ターゲット」ではない**：各 BattleSetting 内部に独自のターゲット選択あり；`m_User` は**動作が誰の視点から発動するか**のみ決定。
*   **ターゲット一覧空時の fallback**：`RCG_BattleManager.PlayerBattleUnits[0]`、User ルール命中失敗で開始アクションが失効しないことを保証。
*   **`OnGUI` は丸ごとコメントアウト**：現状は `Preview` でデータ表示のみ、編集は基底 `OnGUI` フローで描画。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_BattleStartActionData.cs`
*   **継承**：`RCG_Asset<RCG_BattleStartActionData>`
*   **AssetGroup**：`EditBattleSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_User` | User | `RCG_SelectTargetData` | デフォルト `SelectTargetType.Range`、範囲 `Ally + All` |
| `m_BattleStartActions` | BattleStartActions | `List<RCG_BattleSetting>` | 開始アクション順序 |

### A.3 主要メソッド

*   **`AddAction(TriggerEffectData)`** — 主入口；User 計算 → iData.User 設定 → 全 actions をキューに追加。
*   **`DefaultAction`** (static) — `Util.GetData(RCG_BattleStartActionGenData.DefaultID)`、デフォルト「None」。

### A.4 他システムとの連携

*   **`RCG_SelectTargetData`** — 実行者選択ルール。
*   **`RCG_BattleSetting`** — 各アクションノード。
*   **`RCG_BattleManager.PlayerBattleUnits`** — ターゲット fallback ソース。
*   **`AddActionMode.PushBack`** — アクションキュー追加方式。

### A.5 既知の問題

*   `OnGUI` の完全な実装はコメントアウト済；基底クラスのデフォルト描画使用。
