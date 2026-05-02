---
title: RCG_UnitLevelData 說明
description: <!-- TODO: 一句話功能摘要 -->
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# RCG_UnitLevelData

> 程式類別名稱：`RCG_UnitLevelData`

## 用途

<!-- TODO: 描述這個 Asset 在遊戲裡負責什麼、什麼情境會用、舉 1-2 個範例。 -->

繼承自 `RCG_Asset<RCG_UnitLevelData>`。

## 編輯器中的樣貌

```
<!-- TODO: 描繪此 Asset 在編輯器內的版面 -->
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **CurveType** | — | <!-- TODO: 說明欄位用途 --> |
| **MinVal** | — | <!-- TODO: 說明欄位用途 --> |
| **MaxVal** | — | <!-- TODO: 說明欄位用途 --> |
| **PowerSegment** | — | <!-- TODO: 說明欄位用途 --> |
| **PowerBase** | — | <!-- TODO: 說明欄位用途 --> |
| **LevelCurve** | — | <!-- TODO: 說明欄位用途 --> |
| **HPCurve** | — | <!-- TODO: 說明欄位用途 --> |
| **AtkCurve** | — | <!-- TODO: 說明欄位用途 --> |

## 行為說明

<!-- TODO: 戰鬥 / 載入 / 解鎖時的觸發時機與順序。 -->

## 注意事項

<!-- TODO: 常見的設計反模式 / 容易踩到的坑。 -->

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊

*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_UnitLevelData.cs`
*   **繼承自**：`RCG_Asset<RCG_UnitLevelData>`
*   **實作介面**：（無）

### A.2 欄位對照（自動產生，需人工複核）

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_CurveType` | CurveType | `CurveType` | `CurveType` | |
| `m_MinVal` | MinVal | `float` | `MinVal` | UCL.Core.PA.Conditional("m_CurveType",false, CurveType.Linear) |
| `m_MaxVal` | MaxVal | `float` | `MaxVal` | UCL.Core.PA.Conditional("m_CurveType", false, CurveType.Linear) |
| `m_PowerSegment` | PowerSegment | `int` | `PowerSegment` | UCL.Core.PA.Conditional("m_CurveType", false, CurveType.Power) |
| `m_PowerBase` | PowerBase | `float` | `PowerBase` | UCL.Core.PA.Conditional("m_CurveType", false, CurveType.Power) |
| `m_LevelCurve` | LevelCurve | `CurveData` | `LevelCurve` | |
| `m_HPCurve` | HPCurve | `CurveData` | `HPCurve` | |
| `m_AtkCurve` | AtkCurve | `CurveData` | `AtkCurve` | |

### A.3 重要 Method 摘要

<!-- TODO: 補上影響行為的關鍵 method（OnGUI / Preview / 序列化覆寫等）。 -->

### A.4 與其他系統的互動

<!-- TODO: 列出依賴 / 被依賴的類別與系統。 -->

### A.5 已知議題（選填）

<!-- TODO: TODO/FIXME 摘錄、待重構點。 -->
