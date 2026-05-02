---
title: 故事资料 (RCG_StoryData) 说明
description: 一段「故事」的完整定义：起始事件、子故事字典、条件、随机起点、Tag 筛选
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 故事资料

> 程式类别名称：`RCG_StoryData`

## 用途

**一段「故事」的完整剧本资料**。故事比 `RCG_MapEventData` 更复杂——包含多段子故事 (`SubStory`) 构成的对话树，玩家做选择后跳到不同子故事。例如「商人」可以是一个 Story：起始画面 → 选择 →「购买」/「离开」/「杀商人」 → 各自的子故事 → 结局。

继承自 `RCG_Asset<RCG_StoryData>`，实作介面：`UCLI_ShortName`。

## 编辑器中的样貌

```
RCG_StoryData: <ID>
    StoryData (m_StoryData)         ← 故事的元资料
        Tags / CanTriggerRepeatedly / Conditions
        IsRandomStart / RandomStartStories
    Story (StartEvent)              ← 起始 SubStory（key = "Start"）
    SubStory (m_EventDic)           ← 所有子故事 dictionary：{ id → RCG_OptionEventData }
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Tags** | 否 | 故事标签（用于 StoryDropPool 筛选） |
| **CanTriggerRepeatedly** | — | 是否可重复触发 |
| **Conditions** | 否 | 触发条件（OR；空 = 无条件） |
| **IsRandomStart** | — | 是否从多个起点随机选一个 |
| **RandomStartStories** | IsRandomStart=true | 随机起点清单 + 权重 |
| **Story** | 是 | 主起始事件（key 固定为 `"Start"`） |
| **SubStory** | 否 | 所有子故事字典（key = state name，value = `RCG_OptionEventData`） |

每个 `RCG_OptionEventData`（子故事）内含对话 / 选项 / 结果效果，可指定下一个 SubStory 的 ID 形成树状结构。

## 行为说明

### 起点选择 (`GetStartSubStory(iSubStory)`)
*   如果指定了 `iSubStory` → 用它。
*   否则若 `m_IsRandomStart` 且 `m_RandomStartStories` 非空 → 按权重随机抽。
*   否则 fallback 到 `StartStateName = "Start"`。

### 触发 (`StartStory(token, iSubStory, iIsRoot)`)
1. 用 `GetStartSubStory(iSubStory)` 决定起点。
2. 写入 `RCG_DataService.Ins.StartStory(ID, iSubStory, iIsRoot)`（纪录已触发故事）。
3. 设 `s_CurStoryData = this`（给其他系统查询当前故事）。
4. 取对应 SubStory 的 `RCG_OptionEventData.StartEvent(...)` 开始演绎。
5. 结束后清 `s_CurStoryData = null`。

### 条件判断 (`StoryData.CheckCondition`)
*   `m_Conditions` 为空 → 永远回 true。
*   非空 → `CheckConditions_OR`（OR 关系）。

## 注意事项

*   **`StartStateName = "Start"` 是 magic string**：起始事件 key 固定为这个；其他名称不会被当起始。
*   **重复触发旗标**：`CanTriggerRepeatedly` 预设 true（与 MapEventData 相反）；故事多半允许重复。
*   **`s_CurStoryData` 是全局静态**：故事执行期间外部能查到当前故事；但**多个故事不能并行**（会互相覆盖）。
*   **`m_EventDic` 自动补空项**：`GetSubStory(state)` 找不到 key 时会自动 add 一个空 `RCG_OptionEventData`，所以呼叫端不会 NRE，但会看到空白故事。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_StoryData.cs`
*   **继承自**：`RCG_Asset<RCG_StoryData>`
*   **实作介面**：`UCLI_ShortName`
*   **AssetGroup**：`EditQuestSetting`
*   **常数**：`StartStateName = "Start"`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_StoryData` | StoryData | `StoryData`（巢状） | 元资料：tags / conditions / random start |
| `m_EventDic` | SubStory | `Dictionary<string, RCG_OptionEventData>` | 子故事树 |

`StoryData` 巢状含 `m_Tags` / `m_CanTriggerRepeatedly` / `m_Conditions` / `m_IsRandomStart` / `m_RandomStartStories`。

### A.3 重要 Method

*   **`StartStory(token, iSubStory, iIsRoot)`** — 主触发；async；含起点选择 + DataService 纪录 + s_CurStoryData 切换。
*   **`StartEvent` (property)** — `GetSubStory(StartStateName)`，起始事件捷径。
*   **`GetSubStory(state)`** — 取子故事；缺则自动 add 空项。
*   **`StoryData.CheckCondition(data)`** — 触发条件 OR 检查。
*   **`StoryData.GetStartSubStory(iSubStory)`** — 起点选择逻辑。
*   **`DefaultStory` (static)** — 预设故事 fallback。

### A.4 与其他系统的互动

*   **`RCG_OptionEventData`** — 每个 SubStory 的内容（对话、选项、结果）。
*   **`RCG_StoryDropPool`** — 随机池来源。
*   **`RCG_DataService.Ins.StartStory`** — runtime 纪录已触发故事。
*   **`RCG_StoryGenData`** — Asset Entry 包装。
*   **`RCG_GameManager.Random`** — 随机起点抽取。

### A.5 已知议题

*   `s_CurStoryData` 不支援故事并行；极端情况需考虑中断处理。
*   `// public class StoryState` 与 `m_OptionEventData` 注解掉，标示旧版单一事件结构转成 dictionary 的历史。
*   `DeserializeFromJson` 内旧版迁移逻辑（`m_EventDic[StartStateName] = m_OptionEventData`）已注解。
