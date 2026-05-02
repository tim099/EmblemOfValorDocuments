---
title: 実行時データ (RCG_RuntimeData)
description: 汎用 runtime 変数コンテナ：RuntimeStructData で定義された動的構造（Int/Bool/Struct/List/Dic/Enum 等）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 実行時データ

> クラス名：`RCG_RuntimeData`

## 用途

**汎用 runtime 変数コンテナ**。システム内各所で「動的構造の保存スロット」が必要な場合に使用 — 例：戦闘段階状態、カード発動タイミングカウンター、各種一時 flag。実構造は `RCG_RuntimeStructData`（型表）で定義；`RCG_RuntimeData` は**この構造の1インスタンス**で、初期値を含む。

`RCG_Asset<RCG_RuntimeData>` を継承。実装：`UCLI_FieldOnGUI`。

## エディタ上の見た目

```
RCG_RuntimeData: <ID>
    StructType   ← 参照する RCG_RuntimeStructData ID（型定義）
    DefaultValue ← このインスタンスの初期値（StructType に応じて動的描画）
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **StructType** | はい | 参照する `RCG_RuntimeStructData` ID（このインスタンスのデータ形を決定） |
| **DefaultValue** | はい | このインスタンスの初期値；型変動時に自動再構築 |

## 動作説明

### 型変更時の自動再構築
`DefaultValue` の getter が `m_DefaultValue.ID` と `m_StructType.ID` を比較、不一致 → null し再構築。**旧型データが残らないことを保証**。

### シリアライズ
`SerializeToJson` が base + `DefaultValue` を一緒に出力（key = `DefaultValueKey = "DefaultValue"`）；デシリアライズ時に該 key があれば DefaultValue を上書き。

### デフォルト取得 (`InGameData`)
`RCG_RuntimeData.InGameData` が `RCG_RuntimeGenData.InGameDataID` 対応の Asset を取得、「グローバル In-Game 変数保存」として使用。

### `RCG_RuntimeDataConst`（同ファイルサブクラス）
m_StructType 編集をスキップ、直接 DefaultValue 表示。`[UCL_IgnoreAsset]` マーク済で独立 Asset として出現しない。

## 注意事項

*   **`StructType` は先に存在必須**：対応 `RCG_RuntimeStructData` を先に作成してからここで参照。
*   **StructType 変更で既存 DefaultValue クリア**：型不一致で直接再構築、旧データ消失。
*   **`DefaultType` enum がよく使う ID を列挙**：`BattleState` / `CardTriggerTiming` は内蔵共通 struct type のショートカット方式。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RuntimeData/RCG_RuntimeData.cs`
*   **継承**：`RCG_Asset<RCG_RuntimeData>`
*   **実装**：`UCLI_FieldOnGUI`
*   **AssetGroup**：`Runtime`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_StructType` | StructType | `RCG_RuntimeStructGenData` | 型表参照 |
| `m_DefaultValue` (private) | DefaultValue | `RCG_RuntimeObject` | property 経由で動的構築 |

### A.3 主要メソッド

*   **`InGameData` (static)** — `Util.GetData(InGameDataID)`、グローバル in-game 変数保存。
*   **`DefaultValue` (property)** — 型不一致自動再構築ロジック含む。
*   **`SerializeToJson / DeserializeFromJson`** — DefaultValue サブオブジェクトシリアライズ含む。
*   **`GetEnumIDs()`** — Enum 型の全 enum 文字列値を取得。
*   **`OnGUIDefaultValue(field, dataDic)`** — DefaultValue 編集欄描画。
*   **`OnGUI(field, dataDic, params)`** — `UCLI_FieldOnGUI` 介面実装。

### A.4 他システムとの連携

*   **`RCG_RuntimeStructData`** — 型定義ソース。
*   **`RCG_RuntimeObject`** — 動的値のコンテナクラス。
*   **`RCG_RuntimeGenData`** — Asset Entry。
*   **`RCG_RuntimeDataConst`**（同ファイル）— 独立存在不可な定数バリアント。

### A.5 既知の問題

*   `RCG_RuntimeDataConst` は `[UCL_IgnoreAsset]` だが参照可能、誤って Asset 作成しないよう注意。
