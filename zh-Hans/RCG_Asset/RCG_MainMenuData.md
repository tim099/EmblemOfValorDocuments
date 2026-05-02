---
title: 主选单资料 (RCG_MainMenuData) 说明
description: 主选单画面的设定容器（背景、按钮、版面）；目前内容主要由 RCG_MainMenuSetting 处理
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 主选单资料

> 程式类别名称：`RCG_MainMenuData`

## 用途

**主选单画面的设定**——包装一个 `RCG_MainMenuSetting`，让不同 build / 版本能套不同主选单（例如展览版用简化选单、Demo 版锁部分按钮）。

继承自 `RCG_Asset<RCG_MainMenuData>`。

## 编辑器中的样貌

```
RCG_MainMenuData: <ID = Default>
    MainMenuSetting   ← 实际的主选单设定（背景、按钮、版面、过场⋯）
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **MainMenuSetting** | 是 | 主选单设定本体（`RCG_MainMenuSetting`），含背景、按钮、版面细节等 |

## 行为说明

本类别本身只是个薄薄的包装；实际逻辑都在 `RCG_MainMenuSetting`（不是 RCG_Asset 子类，是纯资料容器）。

### Static 入口
*   `RCG_MainMenuData.CreateInstance()` — 从 Asset 取 `Default` 实例（强制重读）。
*   `RCG_MainMenuData.Ins` 已被注解；引用点改走 `RCG_GameSettingData.m_MainMenu`。

### 引用关系
`RCG_GameSettingData.m_MainMenu`（型别 `RCG_MainMenuEntry`）引用此资料；不同游戏版本可在 GameSettingData 上指定不同 MainMenuData。

## 注意事项

*   **ID 预设为 `Default`**：本类别预期单一 Asset；要做版本变体就建多个 ID。
*   **`m_MainMenuSetting` 标 `[AlwaysExpendOnGUI]`**：Inspector 内预设展开，方便编辑。
*   **`Ins` static 已被注解**：取资料统一从 GameSettingData 走，不要直接 `RCG_MainMenuData.Util.GetData(...)`。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_MainMenuData.cs`
*   **继承自**：`RCG_Asset<RCG_MainMenuData>`
*   **AssetGroup**：`EditGameSetting`
*   **常数**：`DefaultID = "Default"`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_MainMenuSetting` | MainMenuSetting | `RCG_MainMenuSetting` | `[AlwaysExpendOnGUI]` |

### A.3 重要 Method

*   **`CreateInstance()` (static)** — `Util.GetData(DefaultID, false)`。
*   建构式预设 `ID = DefaultID`。

### A.4 与其他系统的互动

*   **`RCG_MainMenuSetting`** — 实际内容。
*   **`RCG_MainMenuEntry`** — Asset Entry 包装；`RCG_GameSettingData.m_MainMenu` 用此型别引用。

### A.5 已知议题

*   `Ins` static 已被注解，标示「请走 GameSettingData 取」的设计转变。
