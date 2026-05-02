---
title: 解锁资料 (RCG_UnlockData) 说明
description: 通用解锁条件设定（卡牌 / 装备 / 道具 / 技能 / 系统功能）；含商店解锁与 Steam 成就串接
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 解锁资料

> 程式类别名称：`RCG_UnlockData`

## 用途

**通用的解锁条件设定**。多个资料（卡牌 / 装备 / 道具 / 技能 / 角色 / 大地图 / 牌组 / 主动能力）可以引用同一个 `RCG_UnlockData` 共享解锁条件——例如「达到 5 级时一次解锁一组初级卡牌与装备」只需建一个 UnlockData，相关资产 `m_Unlock.ID = "Lv5_Unlock"` 即可。

也支援「祝福商店解锁」（解锁后仍要花祝福购买）与「系统功能解锁」（解锁后启用最终关 / 禁卡 / 篝火强化等）。

继承自 `RCG_Asset<RCG_UnlockData>`。

## 编辑器中的样貌

```
RCG_UnlockData: <ID>
    UnlockSetting       ← 解锁条件本体（UnlockType: Level / SkillTagLevel / None / Tutorial / Achievement / Never）
    IsBlessingShop      ← 是否透过祝福商店解锁
    IsHiddenInCodex     ← 图鉴中隐藏
    SystemUnlockItems   ← 解锁此 Asset 时顺便启用的系统功能（FinalLevel / BanCard / Inheritance...）
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **UnlockSetting** | 是 | 解锁条件主设定 |
| **IsBlessingShop** | — | true：解锁条件达成后仍要在祝福商店购买才生效 |
| **IsHiddenInCodex** | — | 图鉴隐藏（不暴露未解锁的物件名） |
| **SystemUnlockItems** | 否 | 一同启用的系统功能清单（`SystemUnlockItem` enum） |

`UnlockSetting` 内含：

| 子栏位 | 说明 |
|---|---|
| **UnlockType** | `Level`（玩家等级）/ `SkillTagLevel`（职业等级）/ `None`（直接解锁）/ `Tutorial`（通教学）/ `Achievement`（特定成就）/ `Never`（永不解锁） |
| **SkillTag** | UnlockType=SkillTagLevel 时的职业 |
| **UnlockLevel** | UnlockType=Level/SkillTagLevel 时的等级门槛 |
| **Achievement** | UnlockType=Achievement 时的成就 ID |

## 行为说明

### `CheckUnlock()` (static)
扫所有 UnlockData：
1. 已在 `RCG_GameRecord.UnlockRecords` 内 → 跳过。
2. `UnlockType.None` → 直接记录解锁 + 套用 `SystemUnlockItems`，但不弹窗显示。
3. 其他类型：`!IsLocked()` → 记录 + 套用 SystemUnlockItems；非祝福商店物品则加入回传 set（让上层弹解锁通知）。
回传「本次新解锁且非祝福商店的 ID 集合」，供 UI 显示「新解锁物品」。

### `GetUnloackables<T>(id)` (generic static)
查所有实作 `RCGI_Unloackable` 的 Asset 中 `UnlockEntry.ID == id` 的；用于本 UnlockData 对应到哪些卡牌 / 装备 / 道具 / 技能 / 角色 / 大地图 / 牌组 / 主动能力。

### `IsLocked()` (UnlockSetting)
*   `Level` → 玩家等级 < `m_UnlockLevel`。
*   `SkillTagLevel` → 对应职业等级 < `m_UnlockLevel`。
*   `None` → false（已解锁）。
*   `Tutorial` → true（这层永远回 true，实际解锁判断在 GameRecord 的 Tutorial 集合）。
*   `Achievement` → 该成就未解锁。
*   `Never` → true。

### Editor 预览
分四类显示对应的 unlockable：CardData / ItemData / EquipmentData / UnitSkillData，可分页切换。BigMap / Character 略过。

## 注意事项

*   **`Tutorial` UnlockType 在 IsLocked 永远回 true**：实际解锁路径走 `RCG_GameRecord.Ins.UnlockedCharacters` 等记录，不靠 `IsLocked`。
*   **`IsBlessingShop = true` 的解锁**有两阶段：条件达成 → 解锁商店上架；玩家购买 → 真正可用（记录在 `RCG_GameRecord.UnlockedCards / UnlockedItems / UnlockedEquipments`）。
*   **`SystemUnlockItems` 是 enum 写死**：FinalLevel / BanCard / Inheritance LV1/2 / RestPointEnhancement LV1/2 / Blessing_Shop LV1/2/3 / Secret_Base，新增功能要改 enum + 程式对应点。
*   **Disabled 解锁**：`UnlockEditor` 提供把已解锁的记录「禁用」的开关，加入 `DisabledUnlockRecord` 集合（debug / 测试用）。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_UnlockData.cs`
*   **继承自**：`RCG_Asset<RCG_UnlockData>`
*   **AssetGroup**：`EditGameSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_UnlockSetting` | UnlockSetting | `UnlockSetting` | 条件主设定 |
| `m_IsBlessingShop` | IsBlessingShop | `bool` | |
| `m_IsHiddenInCodex` | IsHiddenInCodex | `bool` | |
| `m_SystemUnlockItems` | SystemUnlockItems | `List<SystemUnlockItem>` | enum：10 种系统功能 |

### A.3 重要 Method

*   **`CheckUnlock()` (static)** — 扫描全 UnlockData 并更新解锁纪录。
*   **`GetUnloackables<T>(id)` (generic static)** — 对应引用此 UnlockData 的所有 Asset。
*   **`GetLockedCards / GetLockedEquipments / ... ` (instance)** — 各类型的快捷查询。
*   **`UnlockSetting.IsLocked()`** — 条件判断。
*   **`UnlockSetting.GetShortName()`** — 自动生成解锁描述字串。
*   **`RCG_UnlockEntry.Unlocked / IsShopItem / Name / UnlockType`** — 引用此资料的 entry 的 helper。

### A.4 与其他系统的互动

*   **`RCG_GameRecord.Ins.UnlockRecords`** — 已解锁纪录集。
*   **`RCG_GameRecord.UnlockedCards / UnlockedItems / UnlockedEquipments / UnlockedCharacters`** — 商店购买纪录。
*   **`RCG_GameAchievement.Ins.IsAchievementUnlocked`** — Achievement 类型解锁检查。
*   **`RCG_GameRecord.DisabledUnlockRecord`** — debug 禁用解锁。
*   **`RCG_DataService.Ins.m_GameData.m_UnlockedSystemUnlocks`** — 系统功能解锁纪录。
*   **`UCLI_Asset.GetUtilByType`** — 反射取对应型别的 Asset Util。

### A.5 已知议题

*   `// TODO: Add Deck QWQ234!!` — 预览 UI 漏了 Deck / Character 两类，列举不完整。
*   `RCG_ShopUnlockData` / `CommonItemGenData` 是同档的辅助结构，与 RCG_UnlockData 配合使用。
