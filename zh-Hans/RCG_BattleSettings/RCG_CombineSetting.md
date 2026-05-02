---
title: 组合效果说明
description: 把多个战斗设定（攻击、治疗、抽牌…）打包成单一效果一起执行；最常用的容器型设定
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 组合效果

> 程式类别名称：`RCG_CombineSetting`

## 用途
把**多个**战斗设定（攻击、治疗、抽牌、给状态 …）**打包**成一个单位，让它们在战斗中**一起**执行。这是制作「复合效果卡」最常用的容器，例如：
*   「造成 3 点伤害 + 抽 1 张牌」
*   「给予 2 点护甲 + 加 1 层敏捷」
*   「攻击敌人前排 + 自损 1 点 HP」

## 编辑器中的样貌
```
▼ ✓ [组合效果(Combine)] [缩图描述]
    OverrideDescription  □
    描述                 [被遮蔽 / 显示]
    组合效果             ▶ (展开后是一份子设定清单)
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **OverrideDescription** | 否 | 是否覆写子设定串接出来的描述。打勾后改用「**描述**」栏位，避免子设定描述叠加过长。 |
| **描述** | OverrideDescription 打勾时必填 | 自订描述本体（支援多语系 `RCG_LocalizeData`）；OverrideDescription 没打勾时这栏会自动隐藏。 |
| **组合效果** | 是 | 要组合执行的子战斗设定清单。每个子项都是一个完整的战斗设定（攻击 / 治疗 / 抽牌等）。 |

## 行为说明

### 描述如何显示
*   **OverrideDescription 没打勾**：把每个子设定的描述用换行串起来。空描述会跳过。
*   **OverrideDescription 打勾**：直接显示「描述」栏位，子设定描述完全不参与。

> [!TIP]
> 子设定描述串太长变成裹脚布时，**先试 OverrideDescription**，自己写一段干净的整体描述。但记得手动同步维护！

### 卡片可不可以打出来
**全部** 子设定都判定可打出，整个组合才打得出来。任一子设定失败（例如资源不足），整张卡都不能用。

### 预览伤害显示
取所有子设定中**最大**的那一个伤害值作为预览（例如 3 个攻击子设定分别预览 5 / 8 / 2，UI 显示 8）。

### 攻击标签 / 卡片资讯
所有子设定的标签、Buff 图示等会**去除重复**后合并显示，避免同一个 Buff 图示在不同子设定间连续出现多次。

## 注意事项

*   **保持子设定粒度单一**：每个子设定只负责一件事（造成伤害 / 抽牌 / 加状态），由「组合效果」负责组装。这样方便重用、改动单一行为时不会牵动整体。
*   **避免巢状过深**：组合效果里再放组合效果虽然可以，但会让描述系统与预览伤害显示变得反直觉。**建议最多两层**。
*   **空清单也合法**：「组合效果」可以为空（不设定任何子项），但这样这张卡就什么都不会发生 — 属于资料错误。

---

## 附录：程式人员参考 (Programmer Reference)

> 此段以下使用程式内部术语，受众转为程式人员与 AI agent。前半段内容请优先采信。

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CombineSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_CombineSetting` → 「组合效果」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_OverrideDescription` | `OverrideDescription`（未本地化） | `bool` | — | 控制 `m_Description` 的 GUI 可视性 |
| `m_Description` | 描述 | `RCG_LocalizeData` | `Description` | `[Conditional(nameof(m_OverrideDescription), false, true)]` 条件显示 |
| `m_CombineSettings` | 组合效果 | `List<RCG_BattleSetting>` | `CombineSettings` | `[AlwaysExpendOnGUI]` 永远展开 |

### A.3 重要 Method 摘要
*   **`CombineSettings` (property)** → `m_CombineSettings.GetEnableBattleSettings()`，过滤掉 `m_IsEnable = false` 的项目。
*   **`AddAction` (未列于本档，继承自父类)**：实际透过子设定的 `AddAction` 串接，顺序由清单决定。
*   **`CheckPlayable`** → 对所有子设定取 AND；任一 false 即整体 false。
*   **`GetPreviewDamage`** → 对子设定的 `GetPreviewDamage` 取 `Mathf.Max`，初始值 -1。
*   **`Infos / GetBattleTags`** → 对子设定使用 `AppendIfNotRepeat` 去重串接。
*   **`GetDescription`**：
    *   `m_OverrideDescription = true`：直接回传 `m_Description.Name`。
    *   否则：以换行串接所有非空子描述。
*   **`GetBattleSettings<T> / (Type)`** → 自身若符合则加入结果，再递回 `CombineSettings`。

### A.4 与其他系统的互动
*   **`RCG_LoopSetting`** 内含一个 `RCG_CombineSetting` 作为 `m_LoopContent`；Combine 是 Loop 的「执行单元」。
*   **`RCG_BattleTagCombineSetting` 继承 `RCG_CombineSetting`**，特化处理 BattleTag 逻辑。

### A.5 已知议题
*   `m_OverrideDescription` 与 `m_Description` 为两个耦合栏位，旧资料中 `m_OverridingDescriptionKey` 已淘汰（程式中以 `// ` 注解保留）。
