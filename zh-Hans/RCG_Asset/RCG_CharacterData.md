---
title: 玩家角色资料 (RCG_CharacterData) 说明
description: 玩家可选择 / 加入队伍的角色模板：HP、初始牌组、初始技能、解锁条件、加入特典
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 玩家角色资料

> 程式类别名称：`RCG_CharacterData`

## 用途

**玩家可选择或在剧情中加入队伍的角色模板**。每个英雄、伙伴、可招募 NPC 都是一个 `RCG_CharacterData`。内含：基本资讯、初始 HP、初始牌组、初始技能/专精、解锁条件、加入队伍时的「加入牌组 (JoinDeck)」等。

继承自 `RCG_Asset<RCG_CharacterData>`，实作介面：`UCLI_ShortName` / `RCGI_Unloackable`（解锁系统）。

## 编辑器中的样貌

```
RCG_CharacterData: <ID>
    Data (UnitData)            ← 编辑时真正修改的设定（HP / Deck / Skills / Order / Unlock）
    RuntimeData                ← 游戏执行时才变动的资料（装备、当前技能等）
    Preview (右侧)              ← 头像 / 名称 / 最大 HP / 技能 / 牌组
```

## 主要栏位（Data 内部）

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Name** | 是 | 角色显示名（多语系） |
| **MaxHp** | 是 | 最大生命值 |
| **Avatar** | 是 | 头像 / 立绘 sprite |
| **Intro** | 否 | 角色介绍文字（选角画面用） |
| **Order** | 是 | 选角画面排序索引 |
| **Deck** | 是 | 起始牌组（`RCG_DeckData`） |
| **JoinDeck** | 否 | 中途加入队伍时带来的牌组（与 Deck 不同的进度） |
| **UnitSkillDatas** | 否 | 起始已学会的单位技能 |
| **SkillTags** | 是 | 角色拥有的专精（战士 / 法师 / 牧师…），用于卡牌专精检查 |
| **Unlock** | 否 | 解锁条件（`RCG_UnlockEntry`）。`UnlockType.Tutorial` 表示教学解锁；商店解锁则需另在 GameRecord 纪录购买 |

## 行为说明

### 选角分类
程式提供四个 static 入口取得不同角色清单（皆按 `m_Order` 排序）：
*   `GetAllTutorialCharacters()` — 教学解锁的角色
*   `GetAllUnlockedCharacters()` — 已解锁
*   `GetAllLockedCharacters()` — 未解锁
*   `GetAllJoinCharacters()` — 教学 + 已解锁（可加入队伍的）

### 解锁判断 (`Unlocked`)
*   `UnlockEntry.Unlocked == true`：已过解锁条件 → 商店类还要看 `RCG_GameRecord.UnlockedCharacters` 是否含 ID。
*   `UnlockType.None`：永远解锁。
*   `UnlockType.Tutorial`：要 GameRecord 内登记过完成教学。

### 编辑器分页
*   **OnGUI** 显示左侧完整 Data 编辑栏 + 右侧预览。
*   **TestDropSkills** 按钮（内建测试工具）可预览掉落 / 抽卡组合。

## 注意事项

*   **`Order` 影响选角画面排序**：未填会堆在最前。
*   **`JoinDeck` 与 `Deck` 是两份**：开局选的角色用 `Deck`；剧情途中招募的角色用 `JoinDeck`（通常较精简，避免破坏牌组平衡）。
*   **解锁条件的两层判断**：`UnlockEntry` + 商店购买记录；改动解锁逻辑时两边都要检查。
*   **`m_RuntimeData`** 是执行时才变的东西（装备、当前技能），编辑器不要动它。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_CharacterData.cs`
*   **继承自**：`RCG_Asset<RCG_CharacterData>`
*   **实作介面**：`UCLI_ShortName` / `RCGI_Unloackable`
*   **AssetGroup**：`EditCharacter`

### A.2 栏位对照（外层）

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_Data` | Data | `UnitData`（巢状） | 编辑期资料 |
| `m_RuntimeData` | RuntimeData | `UnitRuntimData` | 执行期资料（装备等） |

`UnitData` 内含 `m_LocalizeName` / `m_Avatar` / `m_Intro` / `m_MaxHp` / `m_Deck` / `m_JoinDeck` / `m_UnitSkillDatas` / `m_SkillTags` / `m_Unlock` / `m_Order` 等。

### A.3 重要 Method 摘要

*   **`Unlocked` (property)** — 两阶段解锁判断（UnlockEntry → GameRecord）。
*   **`GetAllTutorialCharacters` / `GetAllUnlockedCharacters` / `GetAllLockedCharacters` / `GetAllJoinCharacters`** (static) — 选角画面用。
*   **`Preview` / `OnGUI`** — 编辑器绘制。
*   **`Data.TestDropSkills(...)`** — 内建掉落测试工具。
*   **`Init(string ID)` / `Init(CharacterID)`** — 多型建构入口。

### A.4 与其他系统的互动

*   **`RCG_DeckData`** — `m_Deck` / `m_JoinDeck` 引用的牌组。
*   **`RCG_UnitSkillData`** — `m_UnitSkillDatas` 引用的初始技能。
*   **`RCG_SkillTagGenData`** — 专精系统。
*   **`RCG_GameRecord.UnlockedCharacters`** — 商店 / 教学解锁记录。
*   **`RCG_UnlockEntry`** — 解锁条件。
*   **`UnitRuntimData`** — 执行期角色状态（HP、装备）。

### A.5 已知议题

*   `Unlocked` 的 `UnlockType.Tutorial` 路径有 `// QWQ23!!` 注解，标示旧版逻辑异动。
*   `m_ActivePowers` 已被注解，表示曾规划过「主动能力」栏位，目前由 `UnitSkillData` 系统取代。
