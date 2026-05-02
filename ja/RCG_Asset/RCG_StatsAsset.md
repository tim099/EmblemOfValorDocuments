---
title: 統計データ資産 (RCG_StatsAsset)
description: 「ゲーム内統計値」と Steam 統計のブリッジ：撃殺数、ダメージ総量等、累積可能な数値
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 統計データ資産

> クラス名：`RCG_StatsAsset`

## 用途

**ゲーム内統計値の定義**。例：「累計撃殺数」「累計使用カード数」「累計クリア回数」のような**累積可能な歴史的数値**を、`RCG_StatsAsset` で定義；オプションで Steam Stats と連携して自動報告可。

`RCG_Asset<RCG_StatsAsset>` を継承。

## エディタ上の見た目

```
RCG_StatsAsset: <ID>
    HasSteamStat         ← Steam 統計と連携するか
    SteamUserStat        (HasSteamStat=true 時) ← 対応する Steam 統計 ID
    GameStatsType        ← 対応するゲーム内統計列挙値（None / KillCount / ... 等）
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **HasSteamStat** | — | Steam 統計と連携するか |
| **SteamUserStat** | HasSteamStat=true 時 | 対応する Steam 統計 entry |
| **GameStatsType** | はい | ゲーム内統計タイプ（`GameStatsType` enum） |

## 動作説明

### Editor 内 `Create BuiltIn Stats` ツール
エディタページ上方にボタン：`GameStatsType` 列挙内の全非 `None` 値に対し、欠落 Asset を自動構築（ID は enum 文字列名）し `m_GameStatsType` を設定。**プログラム端 enum と Asset 端の同期に便利**。

## 注意事項

*   **`GameStatsType` enum はプログラム制御**：新統計タイプ追加には先に enum 改修、その後エディタツールで Asset 補完。
*   **デフォルト ID `None`**（`RCG_StatsEntry.DefaultID`）：「統計なし」を意味；実統計の ID として使わない。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_StatsAsset.cs`
*   **継承**：`RCG_Asset<RCG_StatsAsset>`
*   **AssetGroup**：`EditGameSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_HasSteamStat` | HasSteamStat | `bool` | |
| `m_SteamUserStat` | SteamUserStat | `UCL_SteamUserStatAssetEntry` | `Conditional(HasSteamStat)` |
| `m_GameStatsType` | GameStatsType | `GameStatsType` enum | |

### A.3 主要メソッド / ツール

*   **`CreateSelectAssetPage`** — `RCG_StatsAssetEditorPage.Create()`、エディタページ。
*   **`Create BuiltIn Stats`**（Editor only） — 欠落 stats Asset 自動補完。

### A.4 他システムとの連携

*   **`UCL_SteamUserStatAssetEntry`** — Steam SDK 連携。
*   **`GameStatsType` (enum)** — プログラム端統計タイプ列挙。
*   **`RCG_StatsEntry`** — Asset Entry；デフォルト `None`。
