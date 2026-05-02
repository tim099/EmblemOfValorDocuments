---
title: ゲーム設定 (RCG_GameSettingData)
description: ビルドごとの（Demo / 正式版）ゲームレベル設定：メインメニュー、最大解放レベル、チュートリアルリセット、ショップ更新コスト
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# ゲーム設定

> クラス名：`RCG_GameSettingData`

## 用途

**ビルドレベルのゲーム設定** — 異なる build（Demo / 正式版 / 展示版）に異なる設定を適用可能にする。システムは `Application.version` を ID として該当 Asset を取得；同 ID がない場合は `Default` にフォールバック。

`RCG_Asset<RCG_GameSettingData>` を継承。

## エディタ上の見た目

```
RCG_GameSettingData: <ID = Application.version>
    GameVersion          ← Default / Demo
    MainMenu             ← メインメニュー設定の参照
    MaxUnlockLevel       ← 最大解放レベル上限
    ResetTutorialOnNewGame ← 新ゲームごとにチュートリアルリセット
    RefreshBlessingShopPrice ← 祝福ショップ更新コスト基数
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **GameVersion** | はい | `Default`（普通版）または `Demo`（一部機能 / 解放レベルロック） |
| **MainMenu** | はい | メインメニュー設定の参照（`RCG_MainMenuEntry`） |
| **MaxUnlockLevel** | はい | 最大解放レベル；超過後はレベル成長停止（デフォルト 1000） |
| **ResetTutorialOnNewGame** | — | 新ゲームごとにチュートリアルをリセット、**展示機**用 |
| **RefreshBlessingShopPrice** | はい | 祝福ショップ更新コスト基数；総コスト = (残り購入可能数 - 4) × この値（デフォルト 10） |

## 動作説明

### Asset 取得 (`Ins`)
`RCG_GameSettingData.Ins` が `Application.version` を ID として Asset 庫から対応インスタンスを取得。**そのため Asset 命名は build バージョン番号に符合させる必要**（如 `0.1.0` / `0.2.0-demo`）；正確マッチがなければ、`Util.GetData(version, useDefaultIfMissing = true)` が `Default` にフォールバック。

### Demo 版制限
`GameVersion = Demo` 時にゲームロジックが一部機能を自動ロック（例：`MaxUnlockLevel` 強制低値、特定章無効化）。具体的ロックロジックは各 manager に散在；本ファイルは flag のみ。

## 注意事項

*   **ID は Application.version に対応必須**：build バージョン変更時に旧 Asset は自動適用されない（同名 Asset 新規作成または Default でカバーする必要）。
*   **`m_GameSetting` はコメントアウト済**：元は `RCG_GameInitData.m_GameSetting` がこの Asset を参照、現在は `Application.version` 自動検索に変更。
*   **`RefreshBlessingShopPrice` 公式**：負数 case 含む（残り ≤ 4 時にコストが 0 または負）、実価格ロジックでは ≥ 0 にクランプ必要。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_GameSettingData.cs`
*   **継承**：`RCG_Asset<RCG_GameSettingData>`
*   **AssetGroup**：`EditGameSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_GameVersion` | GameVersion | `GameVersion` enum | `Default` / `Demo` |
| `m_MainMenu` | MainMenu | `RCG_MainMenuEntry` | |
| `m_MaxUnlockLevel` | MaxUnlockLevel | `int` | デフォルト 1000 |
| `m_ResetTutorialOnNewGame` | ResetTutorialOnNewGame | `bool` | デフォルト false |
| `m_RefreshBlessingShopPrice` | RefreshBlessingShopPrice | `int` | デフォルト 10 |

### A.3 主要メソッド

*   **`Ins` (static)** — `Util.GetData(Application.version, useDefaultIfMissing = true)`。

### A.4 他システムとの連携

*   **`RCG_MainMenuData`** — `m_MainMenu` 参照のメインメニュー設定。
*   **`RCG_GameInitData`** — ゲーム初期データ；旧 `m_GameSetting` フィールドはコメントアウト、現在は `Application.version` で自動関連。
*   **`RCG_GameSettingGenData`** — Asset Entry；デフォルト ID = `"Default"`。

### A.5 既知の問題

*   `Ins` の `Debug.Log("Application.version: ...")` は "log!!" マーク済み — dev 用、release 前に削除必須。
