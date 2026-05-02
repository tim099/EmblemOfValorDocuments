---
title: 获得道具 说明
description: 把指定的物品加入玩家背包；可选择是否为临时物品（战斗后自动移除）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 获得道具

> 程式类别名称：`RCG_AddItemSetting`

## 用途
把一个或多个指定物品加入玩家背包，常见于「战斗奖励卡」「拾取道具事件」「临时 buff 道具发放」。

## 编辑器中的样貌
```
▼ ✓ [获得道具(AddItem)] (物品缩图列表)
    Items         ▶ (物品清单；展开可加入多个)
    IsTmpItem     [✓]
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Items**（道具，未本地化于此） | 是 | 要加入的物品清单；每项是 `RCG_ItemGenData`（物品模板）。 |
| **IsTmpItem** (是否为临时物品)（未本地化于此） | — | 预设打勾。打勾 = 战斗结束后自动移除（不影响背包永久状态）；不打勾 = 永久持有。 |

## 行为说明
*   触发时依序把每个物品加入玩家背包，并弹出「获得道具」面板秀给玩家看。
*   若 IsTmpItem 打勾，物品会被标记为临时，战斗结束自动清掉。
*   描述格式为「**获得 {物品 1, 物品 2, ...}**」，临时物品下方会多一行 `[临时道具]` 标记。

## 注意事项
*   **临时物品的存档风险**：战斗中存档重新载入，临时物品的清除规则由背包系统处理；若物品本身有持续效果，务必测试载入流程。
*   **空清单**：技术上合法，但这张卡将什么也不会给予 — 属于资料错误。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_AddItemSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_AddItemSetting` → 「获得道具」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_Items` | `Items` (`Items` key 在通用对照中为「道具」) | `List<RCG_ItemGenData>` | `Items` | 物品模板清单 |
| `m_IsTmpItem` | `IsTmpItem` | `bool` | `IsTmpItem` (=「是否为临时物品?」) | 预设 `true` |

### A.3 重要 Method 摘要
*   **`Infos`**：聚合每个 `m_Items[i].GetData()` 的资讯；若 `m_IsTmpItem`，再追加 `CardInfoData.TmpItemInfo`。
*   **`GetDescriptionFormat`**：以 `, ` 串接物品名，后接 `[TmpItem]`（若临时），最终套 i18n key `AcquireItem`。
*   **`AddAction`**：包成 `AddAsyncActionTrigger`，逐个 `m_Items[i].GetData().AddItem()`，设置 `m_TmpItem` 标记，弹出 `RCG_AquireItemPanel`。

### A.4 与其他系统的互动
*   **`RCG_ItemGenData` / `RCG_Item`**：物品模板与实例。
*   **`UI.RCG_AquireItemPanel`**：获得物品面板 UI。
*   **`CardInfoData.TmpItemInfo`**：静态的「临时物品」资讯区块。
