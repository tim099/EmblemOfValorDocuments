---
title: 对话框 说明
description: 显示角色对话泡泡（含字型大小、震动、音效），纯视觉呈现
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 对话框

> 程式类别名称：`RCG_DialogSetting`

## 用途
**显示角色对话泡泡**。纯视觉呈现，无游戏逻辑影响。常见用途：
*   敌人攻击前的台词
*   特殊技能的剧情演出
*   Boss 战中的对话桥段

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Dialog** | 是 | 对话内容（`RCG_LocalizeData`，支援多语系）。 |
| **Duration** | 是 | 对话显示时长（秒），预设 0.8 秒。 |
| **FontSize** | — | 字体大小，预设 40。 |
| **Shake** | — | 是否震动显示（强调用）。 |
| **PlayAudio** | — | 是否播放音效。 |
| **Audio** | PlayAudio 打勾时必填 | 要播放的音效（`RCG_SEGenData`）；`Conditional` 控制显示。 |
| **HideDescription** | — | 是否隐藏在卡片描述上 — 打勾 = 对话内容**不出现在卡片说明**（纯背景演出）。 |

## 行为说明
*   触发时建立 `VFX_Dialog` 并呼叫 `vfx.SetDialog(this, iData.User)` 显示在发话者头顶。
*   描述显示对话内容本身（除非 `HideDescription`）。
*   **精简描述永远为空**（不会出现在敌方意图列）。

## 注意事项
*   **HideDescription 适合「纯气氛对话」**：例如 Boss 战背景台词 — 卡片说明不该被对话文字塞满。
*   **Duration 过短**：低于 0.5 秒玩家可能读不完；对话越长越要拉长 Duration。
*   **音效未指定但 PlayAudio 打勾**：会尝试播放空 `RCG_SEGenData`，可能 Console 出 warning。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_DialogSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_DialogSetting` → 「对话框」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_Dialog` | `Dialog` | `RCG_LocalizeData` | — | |
| `m_Duration` | `Duration` | `float` | — | 预设 0.8f |
| `m_FontSize` | `FontSize` | `int` | — | 预设 40 |
| `m_Shake` | `Shake` | `bool` | — | |
| `m_PlayAudio` | `PlayAudio` | `bool` | — | |
| `m_Audio` | `Audio` | `RCG_SEGenData` | — | `[Conditional(nameof(m_PlayAudio), false, true)]` |
| `m_HideDescription` | `HideDescription` | `bool` | — | |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：`RCG_VFXManager.CreateVFX(VFX_Dialog) as RCG_VFX_Dialog` → `vfx.SetDialog(this, iData.User)`。
*   **`GetDescriptionFormat`**：`m_HideDescription ? string.Empty : m_Dialog.Name`。
*   **`GetDescriptionShort`** → 永远空字串。

### A.4 与其他系统的互动
*   **`CommonVFX.VFX_Dialog` / `RCG_VFX_Dialog.SetDialog`**：对话 UI 入口。
