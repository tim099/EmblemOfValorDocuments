---
title: 背景音乐 (RCG_BGMData) 说明
description: BGM 设定：音档、音量；支援 PlayBGM（取代）与 PushBGM（堆叠）两种播放模式
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 背景音乐

> 程式类别名称：`RCG_BGMData`

## 用途

**背景音乐的模板**。与一次性音效 (`RCG_SEData`) 不同，BGM 是持续播放的背景音乐；地图、战斗、选单各有自己的 BGM。本资料定义 BGM 的音档、音量、备注。

继承自 `RCG_Asset<RCG_BGMData>`。

## 编辑器中的样貌

```
RCG_BGMData: <ID>
    BGM        ← 音档（RCG_AudioData，预设 GroupBGM）
    Volume     ← 音量
    Note       ← 备注
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **BGM** | 是 | 音档，预设 `BGM_MapBGM` |
| **Volume** | 是 | 音量 0~1（slider），预设 1.0 |
| **Note** | 否 | 备注（编辑时自用） |

## 行为说明

### `PlayBGM()` vs `PushBGM()`
*   **PlayBGM**：取代当前 BGM（停掉旧的、播新的）。
*   **PushBGM**：堆叠到 BGM stack 上（常用于「战斗开始播战斗音乐、结束 pop 回原本的地图音乐」）。
两者都有对应的 async 版本（`PlayBGMAsync` / `PushBGMAsync`）。

### `m_BGM.IsEmpty` 时静默 return
不会 LogError；只有 clip 载入失败（IsEmpty=false 但 GetClip 回 null）才会 LogError。

### 预设 ID
`RCG_BGMData.DefaultBGM = "BGM_MapBGM"`：地图预设 BGM。

## 注意事项

*   **PushBGM 没有对应的 PopBGM 在这个档**：pop 逻辑在 `UCL_GameAudioService` 自己处理（外层呼叫 `PopBGM`）。
*   **BGM 与 SE 走不同混音器**：分别有独立音量控制；挑错路径会被当成 SE。
*   **无 `Preview` override**：用基底类预设绘制，编辑器内看不到音档波形预览。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_BGMData.cs`
*   **继承自**：`RCG_Asset<RCG_BGMData>`
*   **AssetGroup**：`EditGameSetting`
*   **常数**：`DefaultBGM = "BGM_MapBGM"`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_BGM` | BGM | `RCG_AudioData` | 预设 `ModResource + GroupBGM + DefaultBGM` |
| `m_Volume` | Volume | `float` | `[UCL_Slider(0, 1)]` |
| `m_Note` | Note | `string` | |

### A.3 重要 Method

*   **`PlayBGM() / PlayBGMAsync()`** — 取代当前 BGM。
*   **`PushBGM() / PushBGMAsync()`** — 堆叠播放。

### A.4 与其他系统的互动

*   **`UCL.Core.Game.UCL_GameAudioService`** — 实际播放服务（`PlayBGM` / `PushBGM`）。
*   **`RCG_AudioData`** — 音档资源封装。
*   **`RCG_BGMGenData`** — Asset Entry。
