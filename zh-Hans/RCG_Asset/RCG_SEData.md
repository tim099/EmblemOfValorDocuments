---
title: 音效资料 (RCG_SEData) 说明
description: 单一音效的设定：音档、音量、AudioType、可同步等待结束
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 音效资料

> 程式类别名称：`RCG_SEData`

## 用途

**单一音效（SE）的设定**。例如「卡牌打出音」「命中音」「按钮点击音」「胜利音乐」等。每个 `RCG_SEData` 是一份音档 + 音量 + 类型，可以同步播放或等待播完。

继承自 `RCG_Asset<RCG_SEData>`。

## 编辑器中的样貌

```
RCG_SEData: <ID>
    SE          ← 音档（RCG_AudioData）
    Volume      ← 音量（0~1，slider）
    AudioType   ← 音效类型（SE / UI 等）
    Note        ← 备注
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **SE** | 是 | 音档（`RCG_AudioData`），预设走 `ModResource` 载入 |
| **Volume** | 是 | 音量 0~1（slider），预设 1.0 |
| **AudioType** | 是 | 音效类型（`UCL.Core.Game.AudioType`），预设 `SE` |
| **Note** | 否 | 备注（编辑时自用） |

## 行为说明

### `PlaySE()`
非阻塞播放——`PlaySEAsync().Forget()` 立刻返回，不等待音档结束。

### `PlaySEAsync()` (UniTask)
async 取 clip → `UCL_GameAudioService.Ins.Play(clip, audioType, volume)`，回传 `AudioPlayer`。

### `PlaySEUntilEnd(token)`
取 player 后设定 `EndAct` 旗标 → `WaitUntil` 结束；可用在「过场 SE 结束才能继续」的情境。

### 预设 ID
`RCG_SEGenData.DefaultID = "Null"`：表示「无音效」；`s_Victory = "SoundEffect_Victory"` 是内建的胜利音效。

## 注意事项

*   **`m_SE.IsEmpty` 时不会 LogError**：静默 return null；要确认音效有播要主动检查。
*   **`PlaySE()` 是非阻塞**：要等音效播完用 `PlaySEUntilEnd(token)`。
*   **AudioType 影响混音器分组**：UI / SE / BGM 各有独立音量控制，挑错类型会被误调。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_SEData.cs`
*   **继承自**：`RCG_Asset<RCG_SEData>`
*   **AssetGroup**：`EditGameSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_SE` | SE | `RCG_AudioData` | 预设 `ModResource + GroupSoundEffect` |
| `m_Volume` | Volume | `float` | `[UCL_Slider(0, 1)]`，预设 1.0 |
| `m_AudioType` | AudioType | `UCL.Core.Game.AudioType` | 预设 `SE` |
| `m_Note` | Note | `string` | |

### A.3 重要 Method

*   **`PlaySE()`** — fire-and-forget。
*   **`PlaySEAsync()`** — 取 clip 并播放，回 `AudioPlayer`。
*   **`PlaySEUntilEnd(token)`** — 等播完再回。

### A.4 与其他系统的互动

*   **`UCL.Core.Game.UCL_GameAudioService`** — 实际播放服务。
*   **`RCG_AudioData`** — 音档资源封装。
*   **`RCG_SEGenData`** — Asset Entry；预设 ID `Null`，`s_Victory` 内建。
