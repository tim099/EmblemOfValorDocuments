---
title: RCG_CombineSetting 說明
description: 組合戰鬥設定 (CombineSetting) 的職責、欄位語意與行為合併規則
last_updated: 2026-05-02
target_audience: [AI_Agent, Gameplay_Programmer, Designer]
---

# RCG_CombineSetting

## 1. 職責 (Responsibility)
將多個 `RCG_BattleSetting` 合併為**單一**設定來依序或同時執行，是組合複合行為（例如「造成傷害 + 抽牌 + 加狀態」）的最常用容器。

繼承自：`RCG_BattleSetting`

## 2. 主要欄位

| 欄位 | 型別 | 物理意義 |
|---|---|---|
| `m_OverrideDescription` | `bool` | 是否覆寫子設定串接出來的描述。`true` 時改用 `m_Description`，避免子設定描述疊加過長。 |
| `m_Description` | `RCG_LocalizeData` | 覆寫描述本體；僅在 `m_OverrideDescription = true` 時生效（由 `Conditional` 屬性管控可視性）。 |
| `m_CombineSettings` | `List<RCG_BattleSetting>` | 要組合執行的子戰鬥設定清單。`AlwaysExpendOnGUI` 讓它在 Inspector 中永遠展開。 |

## 3. 行為合併規則

### 3.1 可玩判定 `CheckPlayable`
**全部** 子設定皆 `CheckPlayable == true` 才視為可玩；任一失敗即整體不可玩（AND 邏輯）。

### 3.2 預覽傷害 `GetPreviewDamage`
取所有子設定 `GetPreviewDamage` 的**最大值**作為代表（**MAX 邏輯**）。意即：UI 顯示的攻擊力 = 子設定中最強那一刀。

### 3.3 戰鬥標籤 / 卡片資訊
`Infos` / `GetBattleTags()` 採**去重串接**，避免同類型 Buff/Tag 在多個子設定間重複堆疊顯示。

### 3.4 描述生成 `GetDescription`
*   **未覆寫**：將每個子設定的 `GetDescription` 換行串接（跳過空字串）。
*   **覆寫**：直接回傳 `m_Description.Name`，子設定不再參與描述合成。

## 4. 設計建議
*   **拆分粒度**：保持每個子設定只負責單一行為（damage、draw、buff 等），由 CombineSetting 負責組裝，方便重用與單元測試。
*   **避免巢狀過深**：CombineSetting 內再放 CombineSetting 雖可行，但會讓 `GetPreviewDamage` 的 MAX 邏輯變得反直覺，建議最多兩層。
