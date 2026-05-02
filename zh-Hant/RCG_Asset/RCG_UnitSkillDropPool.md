---
title: RCG_UnitSkillDropPool 說明
description: <!-- TODO: 一句話功能摘要 -->
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# RCG_UnitSkillDropPool

> 程式類別名稱：`RCG_UnitSkillDropPool`

## 用途

<!-- TODO: 描述這個 Asset 在遊戲裡負責什麼、什麼情境會用、舉 1-2 個範例。 -->

繼承自 `RCG_Asset<RCG_UnitSkillDropPool>`，實作介面：`UCL.Core.UCLI_ShortName`

## 編輯器中的樣貌

```
<!-- TODO: 描繪此 Asset 在編輯器內的版面 -->
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **DropPool** | — | <!-- TODO: 說明欄位用途 --> |
| **FilterType** | — | <!-- TODO: 說明欄位用途 --> |
| **SkillTag** | — | <!-- TODO: 說明欄位用途 --> |
| **Tag** | — | <!-- TODO: 說明欄位用途 --> |
| **OperatorType** | — | <!-- TODO: 說明欄位用途 --> |
| **Filters** | — | <!-- TODO: 說明欄位用途 --> |
| **IsInvert** | — | <!-- TODO: 說明欄位用途 --> |
| **Name** | — | <!-- TODO: 說明欄位用途 --> |
| **DropPool** | — | <!-- TODO: 說明欄位用途 --> |
| **MixDropPools** | — | <!-- TODO: 說明欄位用途 --> |
| **FilterDropData** | — | <!-- TODO: 說明欄位用途 --> |
| **DropType** | — | <!-- TODO: 說明欄位用途 --> |

## 行為說明

<!-- TODO: 戰鬥 / 載入 / 解鎖時的觸發時機與順序。 -->

## 注意事項

<!-- TODO: 常見的設計反模式 / 容易踩到的坑。 -->

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊

*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_UnitSkillDropPool.cs`
*   **繼承自**：`RCG_Asset<RCG_UnitSkillDropPool>, UCL.Core.UCLI_ShortName`
*   **實作介面**：`UCL.Core.UCLI_ShortName`

### A.2 欄位對照（自動產生，需人工複核）

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_DropPool` | DropPool | `RCG_UnitSkillDropPoolGenData` | `DropPool` | |
| `m_FilterType` | FilterType | `FilterType` | `FilterType` | |
| `m_SkillTag` | SkillTag | `RCG_SkillTagGenData` | `SkillTag` | UCL.Core.PA.Conditional(nameof(m_FilterType), false, FilterType.SkillTag) |
| `m_Tag` | Tag | `RCG_ItemTagGenData` | `Tag` | UCL.Core.PA.Conditional(nameof(m_FilterType), false, FilterType.Tag) |
| `m_OperatorType` | OperatorType | `OperatorType` | `OperatorType` | UCL.Core.PA.Conditional(nameof(m_FilterType), false, FilterType.Operator) |
| `m_Filters` | Filters | `List<DropFilter>` | `Filters` | UCL.Core.PA.Conditional(nameof(m_FilterType), false, FilterType.Operator) |
| `m_IsInvert` | IsInvert | `bool` | `IsInvert` | |
| `m_Name` | Name | `RCG_LocalizeData` | `Name` | |
| `m_DropPool` | DropPool | `RCG_CommonDropSetting<RCG_UnitSkillGenData>` | `DropPool` | UCL.Core.PA.Conditional(nameof(m_DropType), false, UnitSkillDropType.DropPool) |
| `m_MixDropPools` | MixDropPools | `List<MixDropPoolData>` | `MixDropPools` | UCL.Core.PA.Conditional(nameof(m_DropType), false, UnitSkillDropType.MixPool) |
| `m_FilterDropData` | FilterDropData | `FilterDropData` | `FilterDropData` | UCL.Core.PA.Conditional(nameof(m_DropType), false, UnitSkillDropType.FilterDrop) |
| `m_DropType` | DropType | `UnitSkillDropType` | `DropType` | |

### A.3 重要 Method 摘要

<!-- TODO: 補上影響行為的關鍵 method（OnGUI / Preview / 序列化覆寫等）。 -->

### A.4 與其他系統的互動

<!-- TODO: 列出依賴 / 被依賴的類別與系統。 -->

### A.5 已知議題（選填）

<!-- TODO: TODO/FIXME 摘錄、待重構點。 -->
