---
title: Localize File Data (RCG_LocalizeFileData)
description: Bundles per-language text files into an Asset; auto-fetches the corresponding file based on current language
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Localize File Data

> Class name: `RCG_LocalizeFileData`

## Purpose

**Bundling Asset for multi-language text files**. When you need long-form localized text (e.g., scripts, tutorial explanations, codex entries), one `.txt` / `.json` per language; this Asset bundles them together and auto-fetches the matching one for the current language.

Inherits from `RCG_Asset<RCG_LocalizeFileData>`.

## Editor Layout

```
RCG_LocalizeFileData: <ID>
    DefaultLang        ← default language (fallback)
    LocalizeFile       ← language → TextAsset dictionary
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **DefaultLang** | yes | Default language (used when no match for current language) |
| **LocalizeFile** | yes | `Dictionary<UCL_LanguageCodeEntry, UCL_TextAssetEntry>`: each language → text file |

## Behavior

### `GetTextData()` Lookup Order
1. `m_LocalizeFile` has current language → use it.
2. Else if `DefaultLang` exists → fallback to default.
3. Else → returns `LocalizeFile.Values.FirstElementInCollection()` (first entry).
4. All empty → null.

### `GetText()`
Returns text content; empty string when no data.

## Caveats

*   **`UCL_LanguageCodeEntry` is the dict key**: different languages use different entries; avoid duplicates when editing.
*   **Fallback order on missing**: current → default → first. Even if all are missing, no NRE — returns empty string.
*   **Difference from short-string i18n**: short strings use `UCL_LocalizeManager` (key-value); this Asset suits **long passages / full content**.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_LocalizeFileData.cs`
*   **Inherits**: `RCG_Asset<RCG_LocalizeFileData>`
*   **AssetGroup**: `EditLocalizeSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_DefaultLang` | DefaultLang | `UCL_LanguageCodeEntry` | |
| `m_LocalizeFile` | LocalizeFile | `Dictionary<UCL_LanguageCodeEntry, UCL_TextAssetEntry>` | |

### A.3 Key Methods

*   **`GetTextData()`** — current language → DefaultLang → first entry fallback chain.
*   **`GetText()`** — string content; empty on missing.

### A.4 System Interactions

*   **`UCL_LanguageCodeEntry`** — language system.
*   **`UCL_TextAssetEntry`** — text file resource wrapper.
*   **`RCG_GameManager.CurLanguageCode`** — current language source.
*   **`RCG_LocalizeFileGenData`** — Asset Entry wrapper.
