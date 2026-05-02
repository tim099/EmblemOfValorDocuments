---
title: メインメニューデータ (RCG_MainMenuData)
description: メインメニュー画面の設定コンテナ（背景、ボタン、レイアウト）；現状の内容は主に RCG_MainMenuSetting で処理
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# メインメニューデータ

> クラス名：`RCG_MainMenuData`

## 用途

**メインメニュー画面の設定** — `RCG_MainMenuSetting` をラップ、異なる build / バージョンに異なるメインメニュー適用可（例：展示版で簡素化メニュー、Demo 版で部分ボタンロック）。

`RCG_Asset<RCG_MainMenuData>` を継承。

## エディタ上の見た目

```
RCG_MainMenuData: <ID = Default>
    MainMenuSetting   ← 実際のメインメニュー設定（背景、ボタン、レイアウト、過場…）
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **MainMenuSetting** | はい | メインメニュー設定本体（`RCG_MainMenuSetting`）、背景、ボタン、レイアウト詳細を含む |

## 動作説明

このクラス自体は薄いラッパー；実ロジックは `RCG_MainMenuSetting` に（RCG_Asset サブクラスではなく純粋データコンテナ）。

### Static エントリ
*   `RCG_MainMenuData.CreateInstance()` — Asset から `Default` インスタンス取得（強制再読込）。
*   `RCG_MainMenuData.Ins` はコメントアウト済；参照箇所は `RCG_GameSettingData.m_MainMenu` 経由に変更。

### 参照関係
`RCG_GameSettingData.m_MainMenu`（型 `RCG_MainMenuEntry`）がこのデータを参照；異なるゲームバージョンで GameSettingData 上に異なる MainMenuData を指定可能。

## 注意事項

*   **ID デフォルト `Default`**：このクラスは Asset 1つのみ想定；バージョン変種のために複数 ID 作成可能。
*   **`m_MainMenuSetting` は `[AlwaysExpendOnGUI]`**：Inspector でデフォルト展開、編集が便利。
*   **`Ins` static はコメントアウト済**：データ取得は GameSettingData 経由に統一、`RCG_MainMenuData.Util.GetData(...)` を直接呼ばない。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_MainMenuData.cs`
*   **継承**：`RCG_Asset<RCG_MainMenuData>`
*   **AssetGroup**：`EditGameSetting`
*   **定数**：`DefaultID = "Default"`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_MainMenuSetting` | MainMenuSetting | `RCG_MainMenuSetting` | `[AlwaysExpendOnGUI]` |

### A.3 主要メソッド

*   **`CreateInstance()` (static)** — `Util.GetData(DefaultID, false)`。
*   コンストラクタデフォルト `ID = DefaultID`。

### A.4 他システムとの連携

*   **`RCG_MainMenuSetting`** — 実コンテンツ。
*   **`RCG_MainMenuEntry`** — Asset Entry ラッパー；`RCG_GameSettingData.m_MainMenu` がこの型で参照。

### A.5 既知の問題

*   `Ins` static はコメントアウト済、「GameSettingData 経由で取得」という設計変更を示す。
