---
title: Runtime 構造定義 (RCG_RuntimeStructData)
description: RCG_RuntimeData が使う「型表」：Int/Float/String/Bool/Json/Enum/Struct/List/Dic 構造を定義
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Runtime 構造定義

> クラス名：`RCG_RuntimeStructData`

## 用途

**`RCG_RuntimeData` が使用する「型定義表」**。システムが動的にデータ構造を定義する必要がある時（C# class でなくデータ駆動）、ここに Struct を作成して記述：どのフィールドがあるか、各々何の型か。例：「プレイヤーの本局状態」Struct に `Score: Int / IsBoss: Bool / Inventory: List<Item>` を含めることが可能。

`RCG_Asset<RCG_RuntimeStructData>` を継承。

## エディタ上の見た目

```
RCG_RuntimeStructData: <ID>
    StructType  ▾ Int / Float / String / Bool / Json / Enum / Struct / Dic / List
    Name(多言語)
    Fields       (StructType=Struct)  ← 各フィールド定義 dic：name → struct type ref
    ElementType  (StructType=List/Dic) ← 要素型 ref
    Enums        (StructType=Enum)     ← enum 値リスト
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **StructType** | はい | 9 種型：5 種プリミティブ型（Int/Float/String/Bool/Json）+ Enum + Struct + List + Dic |
| **Name** | いいえ | 表示名（多言語） |
| **Fields** | StructType=Struct 時 | 構造フィールド dic：フィールド名 → 別の RuntimeStructData への参照 |
| **ElementType** | StructType=List/Dic 時 | 要素型参照 |
| **Enums** | StructType=Enum 時 | 列挙値の文字列リスト |

## 動作説明

### デフォルトプリミティブ型自動生成 (`PrimitiveDic`)
5 種プリミティブ型は初回アクセス時に lazy 構築：`{ "Int": ..., "Float": ..., "String": ..., "Bool": ..., "Json": ... }`。**手動で Asset 作成不要**、使用時に直接 ID `"Int"` / `"Float"` 等を参照。

### ジェネリック文字列解析 (`CreateData`)
ID に角括弧含む（如 `"List<Int>"` または `"Dic<String,Float>"`）時に自動解析：
*   StructType 文字列を切出（List / Dic）。
*   要素型文字列を切出（最後のカンマ後を取得）。
*   構築し `GenericDic` に cache。

### プレビュー (`PreviewData`)
StructType に応じて対応 UI でデータ描画；Struct / List / Dic を再帰展開。

### 編集 (`DataOnGUI`)
StructType に応じて対応編集介面描画（IntField / NumField / TextArea / BoolField / Popup / 入れ子データ展開等）。

### Json シリアライズ (`SerializeDataToJson` / `DeserializeDataFromJson`)
全構造が JSON 往復変換をサポート；Struct/List/Dic を再帰処理。

### データ変動検出
`DataOnGUI` が `iDataDic[PrevDataKey]` で現データ比較；不一致で GUI sub dic クリア（IntField cache に誤入力残留回避）。

## 注意事項

*   **`Event` 型コメントアウト済**（プログラム内 `// Event,`）：以前計画されたが未実装。
*   **プリミティブ型 Asset を手動構築しない**：`PrimitiveDic` の自動 lazy 構築使用、手動構築は自動生成と衝突。
*   **List / Dic のジェネリック文法**：`"List<要素型ID>"` / `"Dic<KeyID,ValueID>"`（Key は現状 String のみサポート、最初の値は無視され最後のみ読込）。
*   **再帰上限 10 層**：`CreateData(layer)` で無限再帰防止；深すぎる入れ子 struct は null 返却。
*   **Struct の Fields 修正**：元データは自動移行されない；schema 変更後の旧データ dictionary には余分な key が残ったり、key 不足になる可能性。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RuntimeData/RCG_RuntimeStructData.cs`
*   **継承**：`RCG_Asset<RCG_RuntimeStructData>`
*   **AssetGroup**：`Runtime`
*   **定数**：`s_PrimitiveTypes` = 5 種；`GenericTypeDic` = List/Dic（dic の要素型数）

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_StructType` | StructType | `StructType` enum | 9 種 |
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Fields` | Fields | `Dictionary<string, RCG_RuntimeStructGenData>` | `Conditional(Struct)` |
| `m_ElementType` | ElementType | `RCG_RuntimeStructGenData` | `Conditional(List/Dic)` |
| `m_Enums` | Enums | `List<string>` | `Conditional(Enum)` |

### A.3 主要メソッド

*   **`PrimitiveDic` / `GenericDic` (static)** — lazy 構築 cache。
*   **`GetAllIDs(useCache)`** — primitives + generics + base IDs を含む。
*   **`CreateData(string)` (override)** — ジェネリック文字列解析含む。
*   **`CreateData(int layer = 0)` (instance)** — 初期値構築（再帰上限含む）。
*   **`DataOnGUI(data, fieldName, dataDic)`** — 主編集介面、StructType で分岐。
*   **`PreviewData(data, fieldName)`** — 主プレビュー。
*   **`SerializeDataToJson / DeserializeDataFromJson`** — JSON 往復。
*   **`GetRuntimeObject(obj, key/int)` / `SetData(obj, key/int, val)` / `GetList / GetDictionary / GetFields / GetFieldInfo`** — 外部データアクセス用 helper。

### A.4 他システムとの連携

*   **`RCG_RuntimeData`** — この型を参照する Asset。
*   **`RCG_RuntimeObject`** — 動的値コンテナ。
*   **`RCG_RuntimeStructGenData`** — Asset Entry ラッパー；`GenericTypeLeft = '<'` 等の定数含む。

### A.5 既知の問題

*   `Event` 型は enum 内コメントアウト、存在するが使用されない。
*   `IsNone` は Struct 型の Fields 空のみ判定；他型は None とみなされない。
*   `CreateData` のジェネリック解析は**最後の** type パラメータのみ取得（Dic に適用、将来拡張時に変更の可能性）。
