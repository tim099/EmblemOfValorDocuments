---
title: TMP 图示资料 (RCG_IconSprite) 说明
description: 给 TextMeshPro 用的 inline 图示资源；UI 描述中的小图示（HP / 护甲 / 卡牌等）来源
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# TMP 图示资料

> 程式类别名称：`RCG_IconSprite`

## 用途

**给 TextMeshPro 用的 inline 图示资源**。卡牌描述、状态说明、战斗日志中常出现「+5 HP」「-3 护甲」之类的文字，HP / 护甲等小图示就是用 TMP 的 `<sprite name=...>` 标签插入；本 Asset 就是这些图示的来源。系统自动把所有 `RCG_IconSprite` 打包成一张 `TMP_SpriteAsset` 给文字渲染。

继承自 `RCG_Asset<RCG_IconSprite>`。

## 编辑器中的样貌

```
RCG_IconSprite: <ID>
    Icon       ← 图示 sprite
    Scale      ← 缩放（TMP 内显示用）
    BearingX   ← 水平基线偏移
    BearingY   ← 垂直基线偏移（预设 0.85）
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Icon** | 是 | 图示 sprite（`RCG_SpriteData`），预设 `Status_BloodDrain` |
| **Scale** | 是 | 在 TMP 内的缩放（预设 1） |
| **BearingX** | — | 水平基线偏移 |
| **BearingY** | — | 垂直基线偏移（预设 0.85，对应 TMP 文字基线） |

## 行为说明

### TMPKey
每个 IconSprite 自动产生 TMP 标签：`<sprite name={ID}>`。在文字里用这个就能 inline 显示对应图示。

### 自动打包成 TMP_SpriteAsset
`InitSpriteAsset(token)` 在启动时扫所有 `RCG_IconSprite` 并打包成单一 `TMP_SpriteAsset`；模组重新载入时 (`UCL_ModuleService.OnLoadedModule`) 会自动 refresh。

### `Save()` 触发 RefreshSpriteAsset
编辑器内存档本 Asset 会自动触发 `RefreshSpriteAsset()` 重建打包，立即看到变更。

### 内建 static 引用
*   `EffectIcon_Target` / `EffectIcon_CountDown` / `EffectIcon_Armor` / `EffectIcon_Health` / `EffectIcon_Card` / `EffectIcon_Die` / `EffectIcon_Energy` / `EffectIcon_Magnifier`
这些是程式内常用的图示，直接用 static property 取得。

### Editor 工具
*   **`Output sprite sheet`**（编辑器专用）：汇出整套 sprite sheet 图（合图档）。
*   **`Refresh SpriteAsset`**：手动重新打包。

## 注意事项

*   **打包是 async**：`InitSpriteAsset` 用 `s_IsRefreshingSpriteAsset` 旗标互斥，避免并行打包；refresh 中再呼叫会等到上次结束。
*   **`BearingY = 0.85` 预设**：与 TMP 字型基线对齐，调整可能会让图示飘上飘下。
*   **预设 ID `AbsorbShield`**（`RCG_IconSpriteGenData.DefaultID`）。
*   **`m_Disable` 已被注解**：曾规划跳过 disabled 图示；目前所有图示都会被打包。
*   **`LocalizeName` 行为**：UI 模式回 TMPKey（图示），非 UI 模式回 i18n 翻译（纯文字）；卡牌描述系统依 `RCG_BattleSetting.IsShowOnUI` 切换。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_IconSprite.cs`
*   **继承自**：`RCG_Asset<RCG_IconSprite>`
*   **AssetGroup**：`EditTags`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_Icon` | Icon | `RCG_SpriteData` | 预设 `Status_BloodDrain` |
| `m_Scale` | Scale | `float` | 预设 1 |
| `m_BearingX` | BearingX | `float` | 预设 0 |
| `m_BearingY` | BearingY | `float` | 预设 0.85 |

### A.3 重要 Method（含 static）

*   **`InitSpriteAsset(token)` (static async)** — 启动时打包；含 OnLoadedModule 订阅。
*   **`GenerateSpriteAsset(token)` (static)** — 载入所有 icons 并建立 `TMP_SpriteAsset`。
*   **`RefreshSpriteAsset / RefreshSpriteAssetAsync`** — 重新打包（重复呼叫互斥）。
*   **`LoadIconSprites(token)` (private static)** — 读所有 textures + names + icons 的 helper。
*   **`Save()` (override)** — 触发 RefreshSpriteAsset。
*   **`TMPKey` (property)** — `<sprite name={ID}>`。
*   多个 static `EffectIcon_*` 快速引用。

### A.4 与其他系统的互动

*   **`TMP_SpriteAsset`** — TextMeshPro 图示资源。
*   **`RCG_TMPTools`** — 打包与工具（`CreateSpriteAsset / RefreshSpriteAsset / CreateIconSpriteSheetEditor`）。
*   **`RCG_SpriteData`** — 来源 sprite。
*   **`UCL_ModuleService.OnLoadedModule`** — 模组重载时自动 refresh。
*   **`RCG_BattleSetting.IsShowOnUI`** — 文字描述模式切换。
*   **`RCG_IconSpriteGenData`** — Asset Entry；预设 `AbsorbShield`。

### A.5 已知议题

*   `m_Disable` / `CheckSpriteAsset / GenerateSpriteAsset (sync ver)` 等多处注解，标示旧版同步打包流程已改 async。
*   `s_IsRefreshingSpriteAsset` 是全局 static flag，极端情况下若 task 抛例外未复位可能卡住未来 refresh。
