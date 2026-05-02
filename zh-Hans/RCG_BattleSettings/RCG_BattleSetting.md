---
title: 战斗设定（基底）说明
description: 所有战斗设定（攻击、治疗、条件、组合等）的共通基底；说明所有子类共用的「启用」开关、描述系统与下拉选单分类
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 战斗设定（基底）

> 程式类别名称：`RCG_BattleSetting`

## 用途
这是「战斗中可发生的一个效果」的**基底类别**，并不是你会直接建立的东西，而是所有战斗设定（攻击、治疗、组合、条件…）的母体。当你在卡牌、敌人技能、状态效果等资料上看到「**战斗设定**」栏位，下拉打开能选的选项，就是它的子类。

## 编辑器中的样貌
任何 `RCG_BattleSetting` 子类在 Inspector 都会长这样：
```
▼ ?  ✓  [类型(英文)] 摘要描述
```
*   **左侧勾选框**：对应 `启用` 栏位。打勾才会在战斗中触发；不打勾的设定会被外层容器（如「组合效果」）整批跳过。
*   **`?` 图示**：显示对应的本份说明文件（前提是该子类有挂 `[HelpURL]`）。
*   **`[类型]` 标签**：显示子类的中文名称（如「攻击」「治疗」「组合效果」），来源为 i18n key `RCG_XxxSetting`；后方括号是英文识别。
*   **后方文字**：呼叫该子类自身产生的「短描述」（Short Name），用于在折叠状态快速辨识。

## 共通栏位

| 编辑器显示 | 对应程式 | 说明 |
|---|---|---|
| **启用** | `m_IsEnable` | 是否在战斗中触发；外层容器会自动过滤掉未启用的设定。 |

> [!NOTE]
> 不同子类额外的栏位，请查看各自的说明（例如「攻击」会有「攻击力」「攻击次数」等）。

## 可选择的子类型

下拉选单中可选的子类分为两个族群（会自动切换）：

### 一般卡牌资料下的选项
编辑卡牌、状态、强化等资料时提供，含 50 余种，例如：
*   **效果类**：攻击、防御、治疗、状态 …
*   **资源类**：抽牌、弃牌、加卡费、消耗道具 …
*   **控制流**：组合效果、条件判断、重复触发、随机触发、回圈每个目标 …
*   **进阶**：召唤、单位变形、团体合击、计数器调整 …

### 编辑怪物（敌方）资料时的额外选项
编辑 `RCG_UnitData` 或 `RCG_MonsterActionData` 时，会多出两个怪物专用设定：
*   **怪物逃跑** (`RCG_MonsterFleeSetting`)
*   **怪物移动** (`RCG_MonsterMoveSetting`)

## 共通行为（你不必设定，但会看到）

*   **描述自动本地化**：每个子类会根据自己的栏位值产生「**详细描述**」（卡牌资讯面板）和「**精简描述**」（敌方意图列）。
*   **预载资源**：战斗前系统会自动把该设定相关的图档、特效预先载入，避免战斗中读取卡顿 — 你**不需要手动处理**。
*   **融合**：在卡牌融合机制下，每个子类定义自己的合并规则（例如两张「治疗」融合会把治疗量加总）。

## 注意事项

*   **未启用的设定不会被当作「不存在」**：它仍会占资料空间、在 Inspector 显示，只是执行时被跳过。要彻底移除请按右侧 `−` 按钮。
*   **新增的子类若 GUI 看不到选项**：那是程式端的 `AllTypes` 清单没有把它加进去 — 属于程式设定问题，请联系程式人员，不是资料端能解决的。
*   **`RCG_BattleSetting` 本身不能被选**：下拉清单只列出实质可用的子类；基底类别只是介面契约。

---

## 附录：程式人员参考 (Programmer Reference)

> 此段以下使用程式内部术语，受众转为程式人员与 AI agent。前半段内容请优先采信。

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_BattleSetting.cs`
*   **继承自**：`UnityJsonSerializable`
*   **介面实作**：`UCLI_ShortName` / `UCLI_TypeList` / `UCLI_GetTypeName` / `UCLI_IsEnable` / `UCLI_NameOnGUI`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_IsEnable` | 启用 | `bool` | `IsEnable` | `[UCL_HideOnGUI]`，由 `NameOnGUI` 自行绘制。 |
| `IsShowOnUI` | （无 GUI） | `static bool` | — | 全域 UI 开关，影响部分子类描述分支。 |
| `s_Types` / `s_MonsterDataTypes` | （无 GUI） | `static List<Type>` | — | 子类选单清单；`AllTypes` getter 依当前资料类型择一回传。 |

### A.3 重要 Method 摘要
*   **`virtual CheckPlayable(TriggerEffectData)`** → 预设 `true`；子类覆写做资源 / 条件检查。
*   **`virtual AddAction(TriggerEffectData, AddActionMode)`** → **核心入口**，把对应 Action 推入 `RCG_BattleManager` 伫列。基底为空实作。
*   **`virtual GetDescription / GetDescriptionFormat / GetDescriptionParams`** → 描述生成三件组；推荐覆写 `Format + Params`，`GetDescription` 预设用两者组合。
*   **`virtual GetDescriptionShort`** → 敌方意图列精简描述，预设空字串。
*   **`virtual GetShortName`** → Inspector 折叠状态的缩写名，预设取 `GetDescription` 截 25 字。
*   **`virtual Info / Infos`** → 卡片悬停资讯；`Infos` 预设聚合单个 `Info`。
*   **`virtual GetBattleTags / HasTerm / GetCollaborators`** → 标签、辞条、协力者查询；容器子类务必递回。
*   **`virtual GetAtk / GetPreviewDamage`** → 攻击力 / 预览伤害；后者回传 `-1` 视为非攻击牌。
*   **`virtual PreloadData(CancellationToken)`** → 战斗前资源预载；容器子类务必 `await` 子设定。
*   **`virtual Fusion / GetFusionCandidateSettings / GetFusionBaseSetting`** → 卡片融合三件组；预设不支援 / 自身为候选 / placeholder 化。
*   **`implicit operator string`** → 呼叫 `GetDescription()`；字串拼接方便但小心 null。

### A.4 与其他系统的互动
*   **`AllTypes` getter**：判断 `UCLI_Asset.s_CurOnGUIAsset` 是否为 `RCG_UnitData` 或 `RCG_MonsterActionData`，是则回传 `s_MonsterDataTypes`（多两个 `RCG_Monster*Setting`），否则回传一般 `s_Types`。
*   **JSON 序列化**：透过继承 `UnityJsonSerializable`，预设由 `JsonConvert.SaveDataToJson(this, JsonConvert.SaveMode.Unity)` 处理（已被注解掉，走 base 预设）。
*   **GUI 绘制**：透过 `NameOnGUI` 在 Inspector 第一行绘制 `IsEnable` 勾选 + `[类型] 短描述`；类型名来自 `GetTypeName(GetType().Name)`，会尝试以 i18n key（去掉 `RCG_` 前缀与 `Setting` 后缀）查表。

### A.5 已知议题
*   `CanEnhence` / `Enhence(RestSetting)` 已注解（旧强化系统残留）。
*   `DeserializeFromJson / SerializeToJson` 也被注解（走 base 预设），若要客制须解开注解。
