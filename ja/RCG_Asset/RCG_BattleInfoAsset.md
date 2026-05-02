---
title: 戦闘情報資産 (RCG_BattleInfoAsset)
description: 戦闘ログ左側に表示される「リアルタイム統計情報」設定（このターン使用カード数、ダメージ総量等）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 戦闘情報資産

> クラス名：`RCG_BattleInfoAsset`

## 用途

**戦闘ログ左側に表示されるリアルタイム統計情報**。例：「このターン使用カード：3」「累積ダメージ：120」「残り抽選枠：2」。各統計項目が1つの `RCG_BattleInfoAsset` インスタンス；戦闘 UI が `m_Order` でソートし順次表示。

`RCG_Asset<RCG_BattleInfoAsset>` を継承。

## エディタ上の見た目

```
RCG_BattleInfoAsset: <ID>
    Enable
    InfoType   ▾ IntVariable
    Variable   ← InfoType=IntVariable 時に表示
    Order
    HideIfZero
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Enable** | はい | この情報を有効にするか（false で表示から除外） |
| **InfoType** | はい | 情報タイプ、現状は `IntVariable` のみ |
| **Variable** | InfoType=IntVariable 時 | 表示する整数変数（`IntVariable`）、戦闘カウンターにバインド可 |
| **Order** | はい | 表示順序、小さいほど前 |
| **HideIfZero** | — | 0 時に非表示（0 だらけの氾濫回避） |

## 動作説明

### `ShowInfo(data)`
*   `m_HideIfZero = true` かつ `Variable.GetValue() == 0` → 非表示。
*   それ以外 → 表示。

### `GetInfo(data)`
`InfoType` に応じて文字列取得：
*   `IntVariable` → `m_Variable.GetDescription(iData)`（フォーマット済、タグプレフィックス含む）。

## 注意事項

*   **`InfoType` は現状 `IntVariable` のみサポート**：将来 String / Float 型を追加する可能性があるが、現在 enum は1値のみ。
*   **`m_Order` 衝突時**：同 Order 値の Asset の表示順序は保証されない；値を手動で分けることを推奨。
*   **HideIfZero は非 IntVariable には無効**：ロジックは現状 IntVariable case のみ対応。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_BattleInfoAsset.cs`
*   **継承**：`RCG_Asset<RCG_BattleInfoAsset>`
*   **AssetGroup**：`EditBattleSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_Enable` | Enable | `bool` | デフォルト `true` |
| `m_InfoType` | InfoType | `InfoType` enum | 現状 `IntVariable` のみ |
| `m_Variable` | Variable | `IntVariable` | `Conditional(IntVariable)` |
| `m_Order` | Order | `int` | デフォルト 0；小さいほど前 |
| `m_HideIfZero` | HideIfZero | `bool` | デフォルト `false` |

### A.3 主要メソッド

*   **`ShowInfo(TriggerEffectData)`** — 表示要否；`m_HideIfZero` と Variable 値で判定。
*   **`GetInfo(TriggerEffectData)`** — フォーマット後の文字列取得。

### A.4 他システムとの連携

*   **`IntVariable`** — 変数バインドソース。
*   **戦闘ログ左側 UI** — これらの Asset の消費先。

### A.5 既知の問題

*   `InfoType` enum が1値のみ、将来拡張計画を示唆するが未実装。
