---
title: 暗霧設定 (RCG_DarkMistSettingsData)
description: 暗霧メカニズムのレベルデータ：異なる暗霧レベル下でのプレイヤー弱化効果と敵強化効果
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 暗霧設定

> クラス名：`RCG_DarkMistSettingsData`

## 用途

**暗霧メカニズムのレベルデータ**。暗霧はゲーム進行と共に深まる環境圧力、レベルが高いほどプレイヤーは弱く、敵は強くなる。本データは「異なる暗霧レベル下で双方が何の効果を得るか」を定義 — 通常は2つの共有 `CustomStatus`（`DarkMistBuff` / `DarkMistDebuff`）をキャリアとし、戦闘開始時に自動適用。

`RCG_Asset<RCG_DarkMistSettingsData>` を継承。

## エディタ上の見た目

```
RCG_DarkMistSettingsData: <ID>
    DarkMistBuffGenData      ← 敵に与える強化状態 ID
    DarkMistDebuffGenData    ← プレイヤーに与える弱化状態 ID
    DarkMistDatas            ← 各レベルデータ一覧（DarkMistLevelData）
        TriggerLevel
        DarkMistBuffStatusData    ← このレベル下の強化効果
        DarkMistDebuffStatusData  ← このレベル下の弱化効果
        Effects                   ← 戦闘開始時に発動する BattleSetting
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **DarkMistBuffGenData** | はい | 共有「敵強化状態」ID（デフォルト `DarkMistBuff`） |
| **DarkMistDebuffGenData** | はい | 共有「プレイヤー弱化状態」ID（デフォルト `DarkMistDebuff`） |
| **DarkMistDatas** | はい | 各レベルデータ一覧（`TriggerLevel` でソート、**最後の要素を優先**） |

各 `DarkMistLevelData` の内訳：

| サブフィールド | 説明 |
|---|---|
| **TriggerLevel** | このレベルの発動閾値（暗霧レベル ≥ この値で本データ使用） |
| **DarkMistBuffStatusData** | このレベル下の敵強化効果（`DarkMistBuff` Status に上書き） |
| **DarkMistDebuffStatusData** | このレベル下のプレイヤー弱化効果（`DarkMistDebuff` Status に上書き） |
| **Effects** | 戦闘開始時に発動する `RCG_BattleSetting` 順序 |

## 動作説明

### 現在のレベルデータ取得 (`GetCurrentDarkMistLevelData`)
リストの**末尾から前へ**最初の `TriggerLevel ≤ iLevel` のデータを検索；そのためリストは**TriggerLevel 昇順ソート**されている必要（例：`[0, 5, 10, 20]`）でないと正しくマッチしない。

### 発動 (`OnTriggerEffect`)
対応レベルデータを見つけたら、当該データの `m_Effects` を全部アクションキューに追加。

### 状態上書き (`UpdateDarkMistEffect`)
現在のレベルの `DarkMistBuffStatusData` を共有 `DarkMistBuff` Status の `m_Effects` に書込 — これは**グローバル Status Asset を直接修正**、毎戦闘開始前にリセット。

## 注意事項

*   **DarkMistDatas は昇順ソート必須**（`TriggerLevel` 依拠）：データ取得ロジックがこの前提に依存、逆順 / 乱順で誤レベル取得。
*   **共有 Status は動的上書き**：`DarkMistBuff` / `DarkMistDebuff` の2つの `RCG_CustomStatusData` の effects は runtime に上書きされる；この2つの Asset を手動修正しない。
*   **複数の使用元**：`RCG_GameInitData.m_DarkMistSettings` はリスト、複数セット可能（例：難度ごとに異なる暗霧）。
*   **Preview はコメントアウト済**：エディタ内プレビューは基底クラスデフォルト描画使用。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_DarkMistSettingsData.cs`
*   **継承**：`RCG_Asset<RCG_DarkMistSettingsData>`
*   **AssetGroup**：`EditQuestSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_DarkMistBuffGenData` | DarkMistBuffGenData | `RCG_CustomStatusGenData` | デフォルト `"DarkMistBuff"` |
| `m_DarkMistDebuffGenData` | DarkMistDebuffGenData | `RCG_CustomStatusGenData` | デフォルト `"DarkMistDebuff"` |
| `m_DarkMistDatas` | DarkMistDatas | `List<DarkMistLevelData>` | 各レベルデータ |

### A.3 主要メソッド

*   **`OnTriggerEffect(data, level)`** — 対応レベルデータ検索 → effects 適用。
*   **`GetCurrentDarkMistLevelData(level)`** — 末尾から前へ `TriggerLevel ≤ level` の要素を検索。
*   **`UpdateDarkMistEffect(level)`** — 共有 Buff Status の effects を上書き。
*   **`SetDarkMistBuff(status)`** — JSON シリアライズで Buff Asset 全体を上書き。

### A.4 他システムとの連携

*   **`RCG_CustomStatusData`** — `DarkMistBuff` / `DarkMistDebuff` の2つのグローバル状態。
*   **`RCG_GameInitData.m_DarkMistSettings`** — このデータを参照するリスト。
*   **`RCG_BattleSetting`** — `m_Effects` の要素型。

### A.5 既知の問題

*   `Preview` 完全実装はコメントアウト；基底クラスデフォルト描画使用。
*   リストは手動ソート必要、自動ソート保証なし；データ順序が乱れると誤レベル取得。
