---
title: ローカライズファイルデータ (RCG_LocalizeFileData)
description: 多言語テキストファイル（言語ごとに1ファイル）を Asset にパッケージ、現言語に応じて対応ファイルを自動取得
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# ローカライズファイルデータ

> クラス名：`RCG_LocalizeFileData`

## 用途

**多言語テキストファイルのパッケージ Asset**。長段落のローカライズテキストが必要な時（如劇本、チュートリアル説明、図鑑エントリ）、言語ごとに1つの `.txt` / `.json` を用意、本 Asset がそれらをまとめ、現言語に応じて対応ファイルを自動取得。

`RCG_Asset<RCG_LocalizeFileData>` を継承。

## エディタ上の見た目

```
RCG_LocalizeFileData: <ID>
    DefaultLang        ← デフォルト言語（fallback 用）
    LocalizeFile       ← 言語 → TextAsset 辞書
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **DefaultLang** | はい | デフォルト言語（現言語に対応ファイルがない時の fallback） |
| **LocalizeFile** | はい | `Dictionary<UCL_LanguageCodeEntry, UCL_TextAssetEntry>`：各言語に対し1つのテキストファイル |

## 動作説明

### `GetTextData()` 取得順序
1. `m_LocalizeFile` に現言語あり → 使用。
2. それ以外で `DefaultLang` あり → デフォルトに fallback。
3. それ以外 → `LocalizeFile.Values.FirstElementInCollection()`（最初の entry） 取得。
4. 完全に空 → null。

### `GetText()`
テキスト内容取得；データなしで空文字列返却。

## 注意事項

*   **`UCL_LanguageCodeEntry` は辞書 key**：異なる言語に異なる entry 使用、編集時に key 重複に注意。
*   **欠落言語時の fallback 順序**：現在 → デフォルト → 最初。全欠落でも NRE せず、空文字列返却。
*   **i18n 短文字列システムとの違い**：短文字列は `UCL_LocalizeManager` 経由（key-value）、本ファイルは**長段落 / 内容全体**向け。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_LocalizeFileData.cs`
*   **継承**：`RCG_Asset<RCG_LocalizeFileData>`
*   **AssetGroup**：`EditLocalizeSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_DefaultLang` | DefaultLang | `UCL_LanguageCodeEntry` | |
| `m_LocalizeFile` | LocalizeFile | `Dictionary<UCL_LanguageCodeEntry, UCL_TextAssetEntry>` | |

### A.3 主要メソッド

*   **`GetTextData()`** — 現言語 → DefaultLang → 最初の entry の fallback chain。
*   **`GetText()`** — 文字列内容；データなしで空文字列。

### A.4 他システムとの連携

*   **`UCL_LanguageCodeEntry`** — 言語システム。
*   **`UCL_TextAssetEntry`** — テキストファイルリソースラッパー。
*   **`RCG_GameManager.CurLanguageCode`** — 現言語ソース。
*   **`RCG_LocalizeFileGenData`** — Asset Entry ラッパー。
