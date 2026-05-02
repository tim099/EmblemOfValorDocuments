---
title: 本地化档资料 (RCG_LocalizeFileData) 说明
description: 把多语系文字档（每语一份）打包成 Asset，依当前语系自动取对应档
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 本地化档资料

> 程式类别名称：`RCG_LocalizeFileData`

## 用途

**多语系文字档的打包 Asset**。当需要长段落的本地化文字（例如剧本、教学说明、功勋簿条目）时，每语系一份 `.txt` / `.json`，本 Asset 把它们绑在一起，依当前语系自动取对应档。

继承自 `RCG_Asset<RCG_LocalizeFileData>`。

## 编辑器中的样貌

```
RCG_LocalizeFileData: <ID>
    DefaultLang        ← 预设语系（fallback 用）
    LocalizeFile       ← 语系 → TextAsset 字典
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **DefaultLang** | 是 | 预设语系（当前语系没对应档时 fallback） |
| **LocalizeFile** | 是 | `Dictionary<UCL_LanguageCodeEntry, UCL_TextAssetEntry>`：每个语系对一份文字档 |

## 行为说明

### `GetTextData()` 取档顺序
1. `m_LocalizeFile` 有当前语系 → 用它。
2. 否则有 `DefaultLang` → fallback 到预设。
3. 否则 → 取 `LocalizeFile.Values.FirstElementInCollection()`（第一个 entry）。
4. 完全空 → null。

### `GetText()`
取文字内容；无资料回空字串。

## 注意事项

*   **`UCL_LanguageCodeEntry` 是字典 key**：不同语系用不同 entry，编辑时注意 key 不要重复。
*   **缺漏语系时 fallback 顺序**：当前 → 预设 → 第一个。即使全缺也不会 NRE，会回空字串。
*   **与 i18n 短字串系统的差异**：短字串走 `UCL_LocalizeManager`（key-value），本档适用于**长段落 / 整段内容**。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_LocalizeFileData.cs`
*   **继承自**：`RCG_Asset<RCG_LocalizeFileData>`
*   **AssetGroup**：`EditLocalizeSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_DefaultLang` | DefaultLang | `UCL_LanguageCodeEntry` | |
| `m_LocalizeFile` | LocalizeFile | `Dictionary<UCL_LanguageCodeEntry, UCL_TextAssetEntry>` | |

### A.3 重要 Method

*   **`GetTextData()`** — 含当前语 → DefaultLang → 第一个 entry 的 fallback 链。
*   **`GetText()`** — 取字串内容；无资料回空字串。

### A.4 与其他系统的互动

*   **`UCL_LanguageCodeEntry`** — 语系系统。
*   **`UCL_TextAssetEntry`** — 文字档资源包装。
*   **`RCG_GameManager.CurLanguageCode`** — 当前语系来源。
*   **`RCG_LocalizeFileGenData`** — Asset Entry 包装。
