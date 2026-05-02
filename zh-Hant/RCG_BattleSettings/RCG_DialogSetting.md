---
title: 對話框 說明
description: 顯示角色對話泡泡（含字型大小、震動、音效），純視覺呈現
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 對話框

> 程式類別名稱：`RCG_DialogSetting`

## 用途
**顯示角色對話泡泡**。純視覺呈現，無遊戲邏輯影響。常見用途：
*   敵人攻擊前的台詞
*   特殊技能的劇情演出
*   Boss 戰中的對話橋段

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Dialog** | 是 | 對話內容（`RCG_LocalizeData`，支援多語系）。 |
| **Duration** | 是 | 對話顯示時長（秒），預設 0.8 秒。 |
| **FontSize** | — | 字體大小，預設 40。 |
| **Shake** | — | 是否震動顯示（強調用）。 |
| **PlayAudio** | — | 是否播放音效。 |
| **Audio** | PlayAudio 打勾時必填 | 要播放的音效（`RCG_SEGenData`）；`Conditional` 控制顯示。 |
| **HideDescription** | — | 是否隱藏在卡片描述上 — 打勾 = 對話內容**不出現在卡片說明**（純背景演出）。 |

## 行為說明
*   觸發時建立 `VFX_Dialog` 並呼叫 `vfx.SetDialog(this, iData.User)` 顯示在發話者頭頂。
*   描述顯示對話內容本身（除非 `HideDescription`）。
*   **精簡描述永遠為空**（不會出現在敵方意圖列）。

## 注意事項
*   **HideDescription 適合「純氣氛對話」**：例如 Boss 戰背景台詞 — 卡片說明不該被對話文字塞滿。
*   **Duration 過短**：低於 0.5 秒玩家可能讀不完；對話越長越要拉長 Duration。
*   **音效未指定但 PlayAudio 打勾**：會嘗試播放空 `RCG_SEGenData`，可能 Console 出 warning。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_DialogSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_DialogSetting` → 「對話框」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_Dialog` | `Dialog` | `RCG_LocalizeData` | — | |
| `m_Duration` | `Duration` | `float` | — | 預設 0.8f |
| `m_FontSize` | `FontSize` | `int` | — | 預設 40 |
| `m_Shake` | `Shake` | `bool` | — | |
| `m_PlayAudio` | `PlayAudio` | `bool` | — | |
| `m_Audio` | `Audio` | `RCG_SEGenData` | — | `[Conditional(nameof(m_PlayAudio), false, true)]` |
| `m_HideDescription` | `HideDescription` | `bool` | — | |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：`RCG_VFXManager.CreateVFX(VFX_Dialog) as RCG_VFX_Dialog` → `vfx.SetDialog(this, iData.User)`。
*   **`GetDescriptionFormat`**：`m_HideDescription ? string.Empty : m_Dialog.Name`。
*   **`GetDescriptionShort`** → 永遠空字串。

### A.4 與其他系統的互動
*   **`CommonVFX.VFX_Dialog` / `RCG_VFX_Dialog.SetDialog`**：對話 UI 入口。
