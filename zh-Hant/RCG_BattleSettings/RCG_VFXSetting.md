---
title: 特效設定 說明
description: 播放純視覺特效（無遊戲邏輯）；支援 VFXGenData 或 Timeline 兩種模式
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 特效設定

> 程式類別名稱：`RCG_VFXSetting`

## 用途
**播放純視覺特效**。無遊戲邏輯影響，純粹做演出。常見用途：
*   攻擊前的閃光 / 蓄力光環
*   特殊技能的場景特效
*   背景時序動畫（Timeline 模式）

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **VFXType** | 是 | 特效類型：<br>• **VFXGenData** — 使用 `RCG_CommonVFXGenData` 生成單一特效（預設）<br>• **Timeline** — 依時序播放多個音效 / 特效 |
| **VFXGenData** | VFXType=VFXGenData 時 | 特效模板。 |
| **Settings** | VFXType=Timeline 時 | 時序設定清單（每項一個 `VFXSetting`，可指定時間與內容）。 |

## 行為說明
*   **VFXGenData 模式**：直接 `m_VFXGenData.GetData().PlayVFX(iData, token)`。
*   **Timeline 模式**：每個 `VFXSetting` 並行 `Play(token)`，全部 `await` 完成後結束。
*   **不參與融合**：`GetFusionBaseSetting()` 回傳 null。

## 注意事項
*   **VFX 為空**：`VFXGenData = null` 時不會 crash 但不播放任何效果。
*   **Timeline 並行與順序**：所有設定**並行播放**而非依序；想要嚴格序列請改用「組合效果」+ 多個 VFXSetting。
*   **戰鬥節奏**：本設定會 await 直到特效播完才繼續 — 過長的特效會拖慢戰鬥節奏。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_VFXSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_VFXSetting` → 「特效設定」
*   **同檔頂層 enum**：`VFXType` (`VFXGenData`, `Timeline`)

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_VFXType` | `VFXType` | `VFXType` (檔內 enum) | — | 預設 `VFXGenData` |
| `m_VFXGenData` | `VFXGenData` | `RCG_CommonVFXGenData` | — | `[Conditional(... VFXGenData)]` |
| `m_Settings` | `Settings` | `List<VFXSetting>` | — | `[Conditional(... Timeline)]` |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：依 `m_VFXType` 分流：
    *   `VFXGenData` → `await m_VFXGenData.GetData().PlayVFX(iData, iToken)`。
    *   `Timeline` → 收集每個 `setting.Play(iToken)` 為 `UniTask`，`UniTask.WhenAll(tasks)` 並行等待。
*   **`GetFusionCandidateSettings`** → 空清單；**`GetFusionBaseSetting`** → null。

### A.4 與其他系統的互動
*   **`RCG_CommonVFXGenData / VFXSetting`**：特效模板與時序設定容器。
*   **`PlayVFX(iData, token)`**：實際播放入口。

### A.5 已知議題
*   舊版 `m_VFX` (`RCG_VFXResData`) 與 `m_VFXTime` 已被新架構（VFXGenData）取代，反序列化邏輯保留註解供參考。
