---
title: Runtime 腳本 (RCG_RuntimeScriptData) 說明
description: 把一段邏輯腳本（RCG_RuntimeScript）打包成 Asset，可被多處引用
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-ja
---

> [!WARNING]
> 翻訳待機中 — このファイルは日本語翻訳が必要です。
参考用に zh-Hant 原文を以下に掲載しています。


# Runtime 腳本

> 程式類別名稱：`RCG_RuntimeScriptData`

## 用途

**把一段 `RCG_RuntimeScript` 邏輯腳本打包成 Asset**。讓多處可重用同一段邏輯而不必複製貼上（例如「死亡時觸發的通用流程」「某條件下執行的判斷腳本」）。

繼承自 `RCG_Asset<RCG_RuntimeScriptData>`。

## 編輯器中的樣貌

```
RCG_RuntimeScriptData: <ID>
    RuntimeScript    ← 實際的腳本內容（RCG_RuntimeScript）
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **RuntimeScript** | 是 | 實際的腳本（`RCG_RuntimeScript`，含節點、變數、執行順序等） |

## 行為說明

本檔本身只是個容器；所有邏輯都在 `RCG_RuntimeScript` 裡。

## 注意事項

*   **Preview 只顯示 ID**：要看實際內容必須開編輯按鈕進入腳本編輯器。
*   **`RCG_RuntimeScript` 的具體結構**請參考程式內定義（不在此資料的職責內）。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RuntimeData/RCG_RuntimeScriptData.cs`
*   **繼承自**：`RCG_Asset<RCG_RuntimeScriptData>`
*   **AssetGroup**：`Runtime`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_RuntimeScript` | RuntimeScript | `RCG_RuntimeScript` | |

### A.3 與其他系統的互動

*   **`RCG_RuntimeScript`** — 實際腳本內容。