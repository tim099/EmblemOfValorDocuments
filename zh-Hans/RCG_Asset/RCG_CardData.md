---
title: 卡牌资料 (RCG_CardData) 说明
description: 游戏中所有卡牌的资料模板：基本资讯、效果、强化分支、被动效果、解锁条件等完整定义
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 卡牌资料

> 程式类别名称：`RCG_CardData`

## 用途
**游戏中每一张卡牌的资料模板**。从基本资讯（名称、费用、稀有度、卡图）到实际效果、强化分支、被动标签、解锁条件，全部定义在这个 Asset 上。在 Developer Page 的 `EditItems` 群组下，所有可玩 / 可商店贩售 / 可掉落的卡牌都是这个类别的实例。

继承自 `RCG_Asset<RCG_CardData>`，实作介面：`RCGI_Item`（可加入背包）/ `RCGI_CardData`（卡牌资料协定）/ `RCGI_Unloackable`（可解锁）。

## 编辑器中的样貌
编辑一张卡牌时，画面分成三大区块：
```
卡牌设定 (CardData)        ← 基本栏位：名称 / 费用 / 类型 / 稀有度 / 标签 / 解锁等
效果 (N)                   ← m_Effects：实际的卡牌效果（OnPlay 等触发）
被动效果 (N)               ← m_PassiveEffects：条件式 BattleTag（影响 Cost 计算等）
预览                       ← 即时呈现卡牌外观与描述
```

最右侧「预览」会即时显示卡牌长相、Tooltip 标签、卡图、解锁条件，并提供「**强化分支预览**」与「**复制描述**」按钮。

## 主要栏位（卡牌设定 / CardData）

### 基本资讯
| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **名称(多国语言)** (`LocalizeName`) | 是 | 卡牌名称（`RCG_LocalizeData`，支援多语系）。 |
| **费用** (`Cost`) | 是 | 打出此牌消耗的能量；整数。 |
| **卡牌类型** (`CardType`) | 是 | `Default`（普通卡）或 `Chant`（**咏唱卡**：需累积回合才能打出）。 |
| **咏唱回合数** (`ChantTurn`) | CardType=Chant 时 | 咏唱要等几回合（变数可变）；条件显示。 |
| **稀有度** (`Rarity`) | 是 | `Bronze` / `Silver` / `Gold` / `Legend` / `Cursed` 等（`RCG_RarityTagGenData`）。 |
| **目标范围** (`TargetType`) | 是 | 17 种：`None`、`Friend`、`Allied(Front/Back)`、`Enemy(Front/Back)`、`All`、`AnyPos`、`AlliedNonLeader`、`AnySummoned` 等。 |
| **使用类型** (`UsedTypeTag`) | 是 | 例如「消耗」「可重复使用」「乙太」等使用后处理方式。 |
| **价格** (`Price`) | 是 | 商人贩售价格；整数。预设 100。 |
| **卡图** (`CardIcon`) | 是 | 卡牌图案（`RCG_SpriteData`）。 |

### 标签与专精
| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **专精** (`SkillTags`) | 否 | 角色需拥有的专精（战士 / 法师 / 牧师…）；空 = 任何角色都能用。多项视为「**任一符合即可**」。 |
| **卡牌标签** (`CardTags`) | 否 | 攻击 / 技能 / 咏唱 / 消耗… 等分类用标签。**重要**：判断「是否为咏唱卡」请用 `HasTag()`，因为 `m_CardTags` 不会自动包含咏唱 tag。 |

### 卡牌效果
| 区块 | 说明 |
|---|---|
| **效果** (`m_Effects`) | 主要的卡牌效果清单；每项是 `RCG_CommonEffect`，内含「触发时机（OnPlay / OnDraw / OnDiscard / ...）+ 组合效果」。打出卡或满足触发条件时会跑这些。 |
| **被动效果** (`m_PassiveEffects`) | 条件式 `RCG_ConditionalPassiveBattleTag` 清单；用于**状态查询**（例如「手牌数 ≥ 5 时 -1 费」）— **不参与 OnTriggerEffect 流程**，纯粹影响 Cost 等属性计算。 |

### 强化系统
| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **禁用的强化分支** (`BannedEnhence`) | 否 | 不允许在此卡上掉落的强化分支 ID。 |
| **强化池** (`EnhencePool`) | 是 | 强化时随机抽取的 `RCG_CardEnhenceDropPoolGenData`。 |
| **ApplyEnhenceOnAllTiming** | — | 是否把强化效果套用到此卡所有触发时机（不只 OnPlay）。 |
| **强化等级** (`UpgradeLv`) | — | 显示用：≥1 会在卡名后加 `+1` / `+2`；通常由系统自动设定，不建议手动改。 |
| **EnhenceLocalize** | 否 | 强化版的命名替代（例如 α、β、γ）；空时用 `+` 前缀。 |

> [!IMPORTANT]
> 目前**强化次数限制为 1**（`m_UpgradeLv < 1` 才允许再强化），且**诅咒卡 (Cursed) 无法强化**。`CanEnhence` 判断会直接挡下。

### 描述
| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **卡牌描述方式** (`CardDescriptionType`) | 是 | `Auto`（自动由效果合成）或 `Manually`（手动撰写）。 |
| **卡牌描述** (`CardDescription`) | Manually 时 | 手动撰写的描述本体（多语系）；自动模式下隐藏。 |
| **描述范本** (`DescriptionTemplate`) | Manually 时 | 含 `{(OnPlay.0)}` 等占位符的范本字串；按 `Generate Description Template` 按钮可从当前效果自动生成。 |

> [!TIP]
> 自动模式下系统会把每个 effect 的描述、目标范围、battle tags 自动串接。多数情况用自动就好；只有需要**精准控制句型** / **隐藏部分效果** / **加风味文字**时才切手动。

### 解锁 / 杂项
| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **解锁** (`Unlock`) | 否 | 解锁条件（`RCG_UnlockEntry`）；未解锁前不会出现在奖励 / 商店中。 |
| **HideInCodex** | — | 在图鉴中隐藏（测试用 / 废卡）。 |
| **备注** (`Note`) | — | 纯文字备注（无实际作用，给设计师自己看）。 |
| **作者** (`Author`) | — | 卡牌作者署名。 |

## 行为说明

### 卡牌显示名称
取自 `LocalizedName`：
*   普通卡：直接是 `m_LocalizeName.Name`
*   强化版（`UpgradeLv ≥ 1`）：
    *   `EnhenceLocalize` 非空 → `{Name}{α/β/γ}`
    *   `UpgradeLv == 1` → `{Name}+`
    *   `UpgradeLv ≥ 2` → `{Name}+{N}`
*   咏唱卡：在游戏中以「**咏唱**」前缀 + 咏唱色显示；Editor 内显示 `咏唱[Name]`

### 咏唱卡（CardType = Chant）
*   需要先「咏唱」`m_ChantTurn` 个回合才能真正打出。
*   描述会在最上方加一行「**咏唱[N]**」（绿色咏唱色）。
*   `HasTag(TagChant)` 会回传 true（即使 `CardTags` 没明列）。

### 描述生成
*   **自动模式 (Auto)**：依「目标范围 → 各 OnTriggerOn 群组 → 触发效果 → battle tags → passive 效果」的顺序自动串接。
*   **手动模式 (Manually)**：以 `m_CardDescription` 为本体，套用 `GetDescriptionParams()` 拿到的 `(OnPlay.0)` 等占位符替换。

### 卡牌资讯面板（Tooltip 右侧）
显示顺序：
1. 必要的专精（如果有）
2. 各效果的 `Infos`（去重）
3. 战斗标签的解说
4. 被动效果的条件 + 标签资讯
5. 使用类型（如果 `m_ShowTag = true`）
6. 咏唱卡额外加上咏唱说明于最顶

### 强化分支
*   `GetEnhenceBranchs(N)` 从 `EnhencePool` 抽出 N 个分支。
*   过滤条件：`BannedEnhence` 列出的不抽、`RCG_CardEnhenceCondition` 不通过的不抽。
*   每个分支会 clone 一份卡片并提升 `UpgradeLv`，再套用对应的强化逻辑。

### 卡牌融合
本类别**不直接定义融合规则**；融合时的「叶效果合并」交给各 `RCG_BattleSetting` 子类自己处理（见 `RCG_BattleSetting.GetFusionCandidateSettings / GetFusionBaseSetting`）。

## 注意事项

*   **咏唱卡的 CardTags 不需手动加入咏唱 Tag**：系统会在 `HasTag` 判断时补上。但要确认 `CardType = Chant`。
*   **价格自动化**：编辑器页面有「Auto Price」按钮，会以「稀有度价值 × 5」覆盖所有卡的 Price。**手动改过的价格会被覆盖**，要保留请避免按。
*   **解锁条件忘填**：`Unlock` 为空 = 预设解锁；要做隐藏成就卡时记得填。
*   **m_Note / m_Author 对玩家不可见**：纯粹开发者记事 / 署名用。
*   **Description 的多语系一致性**：手动描述模式要记得在所有语言都填；缺漏会 fallback 到预设语系。
*   **强化卡的 Enhance Pool 同源**：强化分支抽自卡片自身的 `EnhencePool`；同名卡的不同实例若要有不同强化路线需各自设定。

---

## 附录：程式人员参考 (Programmer Reference)

> 此段以下使用程式内部术语，受众转为程式人员与 AI agent。前半段内容请优先采信。

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_CardData.cs`
*   **继承自**：`RCG_Asset<RCG_CardData>`
*   **实作介面**：`RCGI_Item` / `RCGI_CardData` / `RCGI_Unloackable`
*   **AssetGroup**：`AssetGroup.EditItems`，sort = `RCG_CardData`
*   **i18n 类别名 key**：未直接定义；`AssetGroup` 名称会用于 EditItems 分类显示。

### A.2 栏位对照（外层 `RCG_CardData`）

| 程式栏位 | 编辑器显示 | 型别 | 说明 |
|---|---|---|---|
| `m_Data` | 卡牌设定 | `CardData` (内部巢状类) | 主要资料；A.3 列详细栏位 |
| `m_Effects` | 效果 | `List<RCG_CommonEffect>` | 触发效果清单 |
| `m_PassiveEffects` | 被动效果 | `List<RCG_ConditionalPassiveBattleTag>` | 条件被动标签；用于属性查询，不走 OnTriggerEffect |

### A.3 栏位对照（巢状 `CardData`）

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_LocalizeName` | 名称(多国语言) | `RCG_LocalizeData` | `LocalizeName` | |
| `m_Cost` | 费用 | `int` | `Cost` | 预设 0 |
| `m_CardType` | 卡牌类型 | `CardType` (档内 enum) | `CardType` | `Default` / `Chant` |
| `m_ChantTurn` | 咏唱回合数 | `IntVariable` | `ChantTurn` | `[Conditional(nameof(m_CardType), false, CardType.Chant)]` |
| `m_Rarity` | 稀有度 | `RCG_RarityTagGenData` | `Rarity` | |
| `m_TargetType` | 目标范围 | `TargetType` (档内 enum) | `TargetType` | 17 种值 |
| `m_UsedTypeTag` | 使用类型 | `RCG_UsedTypeTagGenData` | `UsedTypeTag` | |
| `m_Price` | 价格 | `int` | `Price` | 预设 100 |
| `m_SkillTags` | 专精 | `List<RCG_SkillTagGenData>` | `SkillTags` | |
| `m_CardTags` | 卡牌标签 | `List<RCG_CardTagGenData>` | `CardTags` | |
| `m_BannedEnhence` | 禁用的强化分支 | `List<RCG_CardEnhenceGenData>` | `BannedEnhence` | |
| `m_EnhencePool` | 强化池 | `RCG_CardEnhenceDropPoolGenData` | — | |
| `m_ApplyEnhenceOnAllTiming` | `ApplyEnhenceOnAllTiming` | `bool` | — | |
| `m_UpgradeLv` | 强化等级 | `int` | `UpgradeLv` | 预设 0 |
| `m_EnhenceLocalize` | `EnhenceLocalize` | `RCG_LocalizeData` | — | 替代 `+1`/`+2` 显示 |
| `m_CardIcon` | 卡图 | `RCG_SpriteData` | `CardIcon` | 预设指向 `Card_AngelsWings` |
| `m_CardDescriptionType` | 卡牌描述方式 | `CardDescriptionType` (档内 enum) | `CardDescriptionType` | `Auto` / `Manually` |
| `m_CardDescription` | 卡牌描述 | `RCG_LocalizeData` | `CardDescription` | `[Conditional(... Manually)]` |
| `m_DescriptionTemplate` | 描述范本 | `string` | `DescriptionTemplate` | `[Conditional(... Manually)]` |
| `m_Unlock` | 解锁 | `RCG_UnlockEntry` | `Unlock` | |
| `m_HideInCodex` | 在图鉴中隐藏 | `bool` | `HideInCodex` | |
| `m_Note` | 备注 | `string` | `Note` | |
| `m_Author` | `Author` | `string` | — | |
| `m_DataType` | （隐藏） | `DataType` | — | `[UCL_HideOnGUI]`，`BuiltIn` / `InGameRuntime` 等 |

### A.4 重要 Method 摘要

#### 介面实作
*   **`AddItem()`** → `RCG_DataService.Ins.m_DeckData.AddCard(ID)`，回传 self；玩家获得卡牌即执行此入口。
*   **`ToCardData`** → `this`；介面要求实作。
*   **`ToCardBattleData`** → `null`；**已弃用**，改用 `RCG_CardBattleData.CreateCard()` 统一生成。
*   **`Icon` / `IconSpriteData` / `IconTexture`** → `m_Data.m_CardIcon` 对应属性。
*   **`ItemName` / `LocalizedName`** → `m_Data.LocalizeName`（含强化等级后缀逻辑）。
*   **`ShowInfo()`** → 开启 `RCG_CardInfoPanel`。
*   **`GetPreviewDamage(target)`** → 预设 -1（非攻击牌；CardData 层级不算伤害）。
*   **`UnlockEntry`** → `m_Data.m_Unlock`。

#### 描述系统
*   **`Description` (property)** → `DisplayDescription()`。
*   **`DisplayDescription(iData)`** → 咏唱卡先加咏唱标题、后接 `GetDescription`，最末附使用类型（若 `m_ShowTag`）。
*   **`GetDescription(iData, iShowBattleTags)`** → 主逻辑：
    *   `Manually` 模式 + `m_CardDescription` 非空 → 套参数后直接返回。
    *   `Auto` 模式 → 「目标描述 + 各 effect 描述 + battle tags」串接。
*   **`GetDescriptionTemplate(iData)`** → 产生 `(OnPlay.0)` 形式的范本字串（给 Manually 模式的 `Generate` 按钮用）。
*   **`GetDescriptionParams(iData)`** → 拿出所有 effect 的参数列表（给 Manually 模板替换用）。
*   **`CardDisplayName`** → 卡名（咏唱卡会带咏唱前缀 / 颜色）。
*   **`CostStr`** → `Cost.ToString()`。
*   **`ChantTitle(iData)`** → 「咏唱[X]」字串（绿色）。

#### 标签 / 词条 / 协力
*   **`CardTags`** → `m_Data.m_CardTags`（非咏唱 Tag）。
*   **`HasTag(tag)` / `HasTag(tags)`** → 内含「咏唱卡时自动视为含 `TagChant`」的特例逻辑。
*   **`HasTerm(term)`** → 对所有 `CardEffects` 递回查询。
*   **`GetBattleTags()`** → 对所有 `CardEffects` 聚合。
*   **`GetPassiveTags(iData)`** → 回传 `m_PassiveEffects`；用于 Cost 计算等属性查询。
*   **`GetCollaborators()`** → 对所有 `CardEffects` 聚合协力者，去重。
*   **`GetTagsDescription(showHiddenTags)`** → 标签描述字串（隐藏标签包 `[]`）。

#### 战斗与融合相关
*   **`CheckPlayable(iData)`** → 使用类型含 `Unplayable` 直接 false；其他对所有 `CardEffects.CheckPlayable` 取 AND。
*   **`CheckRequireSkill(skill)` / `CheckRequireSkill(skills)`** → 全条件 AND。
*   **`CanPlayThisCard(skills)`** → 「**任一专精符合即可**」（与 `CheckRequireSkill` 不同）。
*   **`GetBattleSettings<T>() / (Type)`** → 对所有 `m_Effects[i].m_CombineSetting` 递回。
*   **`OnTriggerEffect(triggerOn, iData)`** → 取 `m_Effects.GetEffects(triggerOn)`、若 iData 为 null 则建立 child，呼叫 `aEffects.TriggerEffects(iData)` 并 log 统计。

#### 强化分支
*   **`CanEnhence`** → `m_Data.m_UpgradeLv < 1 && !CardRarity.Equals(Cursed)`。
*   **`CreateEnhenceCard(ID)` (private)** → clone 自身，设新 ID，`m_UpgradeLv += 1`，标 `InGameRuntime`。
*   **`GetEnhenceBranchs(branchCount)` (yield)** → 从 `EnhencePool` 抽，过滤 `BannedEnhence` 与 `CheckCondition` 不通过的，逐个 clone + 套用强化资料。

#### 序列化 / 资料流
*   **`SaveToJson(iData)` (static)** → 写入 `ID` + `IsRuntimeCardData` 两栏位。
*   **`LoadFromJson(iData)` (static)** → `IsRuntimeCardData = true` → `RCG_DataService.Ins.m_GameRuntimeAsset.GetRuntimeData<RCG_CardData>(aID)`；否则 `Util.GetData(aID)`。
*   **`Save()`** → base + 清除 `RCG_CardFilter` cache。
*   **`CloneCard()`** → 透过 `SerializeToJson + DeserializeFromJson` 深拷贝。
*   **`CreateSelectAssetPage()`** → `RCG_CardDataEditorPage.Create()`。
*   **`OnLoadModule()` (static)** → `RCG_CardFilter.Clear()` + `Util.PrewarmAllAssets()`。
*   **`PreloadData(token)`** → 标记 `Preloaded` 后预载卡图、稀有度图示、各 effect 的 `PreloadData`。

#### 编辑 / 预览
*   **`OnGUI(iDataDic)`** → 三段式绘制：CardData → Effects → PassiveEffects → Preview。Manually 模式下多显示「Generate Description Template」按钮与描述参数编辑器。
*   **`Preview(iDic, iIsShowEditButton)`** → 卡图 + 名称 + 标签 + 描述 + 强化分支按钮 + 解锁条件 + DisplayCardInfos。
*   **`DisplayCardInfos(iShowOnUI)`** → 切换 `RCG_BattleSetting.IsShowOnUI` 后组装完整 InfoPanel 内容（含咏唱资讯）。

### A.5 与其他系统的互动

*   **`RCG_CardBattleData`** — 战场上实际运作的卡片实例；`RCG_CardData` 是模板，`RCG_CardBattleData.CreateCard(cardData, isChanted)` 产生实例。
*   **`RCG_CommonEffect`** — 卡牌效果单位；`m_Effects` 是它的 `List`，包含 `m_EffectTriggerOn` + `m_CombineSetting`。
*   **`RCG_ConditionalPassiveBattleTag`** — 被动条件式标签；含 `m_Conditions` + `m_Tags`，影响属性查询但不参与 OnTrigger。
*   **`RCG_CardEnhenceData / RCG_CardEnhenceDropPoolGenData`** — 强化分支系统；`GetEnhenceBranchs` 的核心。
*   **`RCG_CardFilter`** — 卡片筛选 cache；`Save()` 与 `OnLoadModule()` 会清。
*   **`RCG_DataService.Ins.m_DeckData`** — 玩家牌组储存；`AddItem` 写入此处。
*   **`RCG_DataService.Ins.m_GameRuntimeAsset`** — runtime 卡片（如 CardToItem 产生的）储存；`LoadFromJson` 在 `IsRuntimeCardData = true` 时读取。
*   **`UI.RCG_CardInfoPanel` / `RCG_CardDataEditorPage` / `RCG_PreviewEnhenceCardPage`** — 主要 UI 入口。
*   **`RCG_CardDataComparer`** (档内) — 排序 helper：先比专精类别 → 稀有度 → 费用降序。

### A.6 已知议题

*   `ToCardBattleData` 已弃用但保留 (`return null`)；注解明示「之后禁止用这个转换为 CardBattleData」。
*   `CanEnhence` 暂时硬编码「强化次数 ≤ 1」，未来解除限制需改此 property。
*   `m_PassiveEffects` 的 `GetPassiveTags` 中注解 `// QWQ23`、`// Append does not work QWQ?` 标示了实作疑虑（用 `Add` 取代 `Append`）。
*   `InitData()` 为 virtual 但目前是空实作；子类 `RCG_CardBattleData` 才真正用到。
*   旧版 `m_UpgradeCards` 栏位已注解掉（曾打算用清单方式列强化目标）。
