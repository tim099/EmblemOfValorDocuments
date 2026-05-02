---
title: Runtime スクリプト (RCG_RuntimeScriptData)
description: 1段のロジックスクリプト（RCG_RuntimeScript）を Asset にパッケージ、複数箇所から参照可能
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Runtime スクリプト

> クラス名：`RCG_RuntimeScriptData`

## 用途

**1段の `RCG_RuntimeScript` ロジックスクリプトを Asset にパッケージ**。複数箇所が同じロジックを再利用してコピー&ペーストする必要をなくす（例：「死亡時に発動する汎用フロー」「特定条件下で実行する判定スクリプト」）。

`RCG_Asset<RCG_RuntimeScriptData>` を継承。

## エディタ上の見た目

```
RCG_RuntimeScriptData: <ID>
    RuntimeScript    ← 実スクリプト内容（RCG_RuntimeScript）
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **RuntimeScript** | はい | 実スクリプト（`RCG_RuntimeScript`、ノード、変数、実行順序等を含む） |

## 動作説明

このファイル自体はコンテナのみ；全ロジックは `RCG_RuntimeScript` 内。

## 注意事項

*   **Preview は ID のみ表示**：実コンテンツを見るには編集ボタンでスクリプトエディタに突入必須。
*   **`RCG_RuntimeScript` の具体的構造**はプログラム内定義参照（このデータの責務外）。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RuntimeData/RCG_RuntimeScriptData.cs`
*   **継承**：`RCG_Asset<RCG_RuntimeScriptData>`
*   **AssetGroup**：`Runtime`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_RuntimeScript` | RuntimeScript | `RCG_RuntimeScript` | |

### A.3 他システムとの連携

*   **`RCG_RuntimeScript`** — 実スクリプト内容。
