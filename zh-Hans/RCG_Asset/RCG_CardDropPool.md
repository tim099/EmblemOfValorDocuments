---
title: 卡牌掉落池 (RCG_CardDropPool) 说明
description: 定义一组「会掉哪些卡、各自的权重」的资料；奖励画面、商店、强化分支底层都靠它抽卡
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 卡牌掉落池

> 程式类别名称：`RCG_CardDropPool`

## 用途

定义「**这个池子会掉哪些卡牌、各张的权重各是多少**」。战斗奖励、商店补货、商人特卖、活动掉落都从某个 `RCG_CardDropPool` 随机抽卡。同样的池子可被多个来源引用，改一次到处生效。

继承自 `RCG_Asset<RCG_CardDropPool>`，实作介面：`UCL.Core.UCLI_ShortName`。

## 编辑器中的样貌

```
RCG_CardDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ 显示名称
        Name(多国语言)
    ▼ DropPool / MixDropPools / FilterDropData  ← 视 DropType 显示对应区块
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **DropType** | 是 | `DropPool`（直接列卡 + 权重）/ `MixPool`（混合别的池子）/ `FilterDrop`（用标签条件筛） |
| **Name** | 否 | 显示名称（多语系）。空白时 `GetShortName()` fallback 到第一个 drop 的名称、再 fallback 到 ID |
| **DropPool** | DropType=DropPool | 卡片清单 + 各自权重（`RCG_CommonDropSetting<RCG_CardGenData>`） |
| **MixDropPools** | DropType=MixPool | 引用其他掉落池并各自指定权重；最终结果是合并后的加权平均 |
| **FilterDropData** | DropType=FilterDrop | 用 `CardDropFilter` 条件式（标签、稀有度、等等）动态筛出符合的卡，无条件 = 全部卡 |

## 行为说明

### 三种模式
*   **DropPool（直接列）**：手动列出每张卡 + 权重；最直观，适合精选池。
*   **MixPool（混合）**：把多个既有池子按权重合并，避免重复维护同一份名单。权重会层层加权（外层 weight × 内层 weight）。最多递回 10 层，避免循环。
*   **FilterDrop（标签筛）**：用条件式从**全卡库**筛出符合的卡，所有命中卡权重均等。条件支援 AND / OR / NOT 巢状。

### 隐性筛选（runtime 战斗中自动套用）
即使你写了权重 0.5 的卡，runtime 也可能把它从池子里剔除：
*   **未解锁** (`UnlockData.CheckCardLocked`)：跳过。
*   **队伍无人能用** (`CheckRequireSkill`)：卡有指定专精且队伍中无任何角色拥有对应专精，跳过。
*   **主选单与非战斗环境**：不做技能检查（避免 UI 预览空转）。
被剔除后剩余卡片会**重新标准化权重**（总和回到 1）。

### 预览
编辑器右侧按 `ShowDetail` 可即时看到当前条件下的最终掉落率清单。

## 注意事项

*   **DropPool 模式下** 反序列化时会**自动移除不存在的卡 ID**（例如 ID 改名 / 被删）；MixPool 模式类似（移除不存在的子池）。看 console 有 `Remove Invalid DropPool` 的 LogError 表示这个池子曾经引用了坏 ID。
*   **FilterDrop 条件全空** = 全卡库均等掉落。除非你真的想要这个效果，否则记得加条件。
*   **MixPool 的循环**会被 `iLayer > 10` 截断成空清单；发现某个池子掉空时先检查混合链是否成环。
*   **RCG_CardFilter cache**：FilterDrop 的查询走 `RCG_CardFilter.GetCards`，有快取；新增卡牌后若预览不对，可从 `RCG_CardData.OnLoadModule` 触发 cache 清除。

---

## 附录：程式人员参考 (Programmer Reference)

> 此段以下使用程式内部术语，受众转为程式人员与 AI agent。前半段内容请优先采信。

### A.1 类别资讯

*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_CardDropPool.cs`
*   **继承自**：`RCG_Asset<RCG_CardDropPool>`
*   **实作介面**：`UCL.Core.UCLI_ShortName`
*   **AssetGroup**：`EditDropSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_Name` | 显示名称 | `RCG_LocalizeData` | `Name` | `[SerializeField] protected` |
| `m_DropPool` | 掉落池 | `RCG_CommonDropSetting<RCG_CardGenData>` | — | `Conditional(m_DropType == DropPool)` |
| `m_MixDropPools` | 混合池清单 | `List<MixDropPoolData>` | — | `Conditional(m_DropType == MixPool)` |
| `m_FilterDropData` | 条件式 | `FilterDropData`（内部巢状） | — | `Conditional(m_DropType == FilterDrop)` |
| `m_DropType` | DropType | `EDropType` enum | `DropType` | 预设 `DropPool` |

### A.3 重要 Method 摘要

*   **`GetDropCards(int, bool)`** → 主要外部入口，回传 `iDropCount` 张随机卡。
*   **`GetDropCardsWithFilterFunc(int, Func)`** → 外部自订筛选版（例如指定稀有度）。
*   **`GetDropRate(bool, CheckDropConditionData, int)`** → 取最终权重表（总和=1）；`iIsFilterSkill = true` 时自动套用 `CheckRequireSkill`。
*   **`CheckRequireSkill(RCG_CardGenData)`** (static) → 三层检查：runtime 中 → 主选单跳过 → 解锁检查 → 队伍专精符合性。
*   **`DeserializeFromJson`** → 载入时清理失效 ID（DropPool / MixPool 两种模式各自处理）。
*   **`Preview`** → 编辑器内预览 UI，可展开看完整掉落率表。

### A.4 与其他系统的互动

*   **`RCG_CommonDropSetting<T>`** — 通用掉落容器；`m_DropPool` 直接用它。
*   **`RCG_CardGenData`** — 卡片 ID 包装；掉落结果是 `List<RCG_CardGenData>`。
*   **`RCG_CardDropPoolGenData`** — Asset Entry 包装；其他 Asset 引用此池子时用这个型别栏位。
*   **`RCG_CardFilter`** — `FilterDrop` 模式下查询卡库的入口。
*   **`RCG_DataService.Ins.m_UnlockData`** — runtime 锁定查询。
*   **`RCG_CharacterDataService.Ins.GetAllSkillTags()`** — 队伍专精查询。

### A.5 已知议题

*   `MixPool` 的 weight 计算对权重总和未做标准化（直接累加 `aWeight * aDrop.m_Weight`）；最终透过 `GetDropRate()` 标准化为总和=1。
*   递回上限 10 层（`iLayer > 10` 直接回空），这个阈值是 magic number。
