---
title: 本地化檔資料 (RCG_LocalizeFileData) 說明
description: 把多語系文字檔（每語一份）打包成 Asset，依當前語系自動取對應檔
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-ja
---

> [!WARNING]
> 翻訳待機中 — このファイルは日本語翻訳が必要です。
参考用に zh-Hant 原文を以下に掲載しています。


# 本地化檔資料

> 程式類別名稱：`RCG_LocalizeFileData`

## 用途

**多語系文字檔的打包 Asset**。當需要長段落的本地化文字（例如劇本、教學說明、功勳簿條目）時，每語系一份 `.txt` / `.json`，本 Asset 把它們綁在一起，依當前語系自動取對應檔。

繼承自 `RCG_Asset<RCG_LocalizeFileData>`。

## 編輯器中的樣貌

```
RCG_LocalizeFileData: <ID>
    DefaultLang        ← 預設語系（fallback 用）
    LocalizeFile       ← 語系 → TextAsset 字典
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **DefaultLang** | 是 | 預設語系（當前語系沒對應檔時 fallback） |
| **LocalizeFile** | 是 | `Dictionary<UCL_LanguageCodeEntry, UCL_TextAssetEntry>`：每個語系對一份文字檔 |

## 行為說明

### `GetTextData()` 取檔順序
1. `m_LocalizeFile` 有當前語系 → 用它。
2. 否則有 `DefaultLang` → fallback 到預設。
3. 否則 → 取 `LocalizeFile.Values.FirstElementInCollection()`（第一個 entry）。
4. 完全空 → null。

### `GetText()`
取文字內容；無資料回空字串。

## 注意事項

*   **`UCL_LanguageCodeEntry` 是字典 key**：不同語系用不同 entry，編輯時注意 key 不要重複。
*   **缺漏語系時 fallback 順序**：當前 → 預設 → 第一個。即使全缺也不會 NRE，會回空字串。
*   **與 i18n 短字串系統的差異**：短字串走 `UCL_LocalizeManager`（key-value），本檔適用於**長段落 / 整段內容**。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_LocalizeFileData.cs`
*   **繼承自**：`RCG_Asset<RCG_LocalizeFileData>`
*   **AssetGroup**：`EditLocalizeSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_DefaultLang` | DefaultLang | `UCL_LanguageCodeEntry` | |
| `m_LocalizeFile` | LocalizeFile | `Dictionary<UCL_LanguageCodeEntry, UCL_TextAssetEntry>` | |

### A.3 重要 Method

*   **`GetTextData()`** — 含當前語 → DefaultLang → 第一個 entry 的 fallback 鏈。
*   **`GetText()`** — 取字串內容；無資料回空字串。

### A.4 與其他系統的互動

*   **`UCL_LanguageCodeEntry`** — 語系系統。
*   **`UCL_TextAssetEntry`** — 文字檔資源包裝。
*   **`RCG_GameManager.CurLanguageCode`** — 當前語系來源。
*   **`RCG_LocalizeFileGenData`** — Asset Entry 包裝。