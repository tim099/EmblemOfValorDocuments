---
title: 組合效果說明
description: 把多個戰鬥設定（攻擊、治療、抽牌…）打包成單一效果一起執行；最常用的容器型設定
last_updated: 2026-05-05
target_audience: [Designer, Modder, AI_Agent]
---

# 組合效果

> 程式類別名稱：`RCG_CombineSetting`

## 用途
把**多個**戰鬥設定（攻擊、治療、抽牌、給狀態 …）**打包**成一個單位，讓它們在戰鬥中**一起**執行。這是製作「複合效果卡」最常用的容器，例如：
*   「造成 3 點傷害 + 抽 1 張牌」
*   「給予 2 點護甲 + 加 1 層敏捷」
*   「攻擊敵人前排 + 自損 1 點 HP」

## 編輯器中的樣貌
```
▼ ✓ [組合效果(Combine)] [縮圖描述]
    OverrideDescription  □
    描述                 [被遮蔽 / 顯示]
    組合效果             ▶ (展開後是一份子設定清單)
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **OverrideDescription** | 否 | 是否覆寫子設定串接出來的描述。打勾後改用「**描述**」欄位，避免子設定描述疊加過長。 |
| **描述** | OverrideDescription 打勾時必填 | 自訂描述本體（支援多語系 `RCG_LocalizeData`）；OverrideDescription 沒打勾時這欄會自動隱藏。 |
| **組合效果** | 是 | 要組合執行的子戰鬥設定清單。每個子項都是一個完整的戰鬥設定（攻擊 / 治療 / 抽牌等）。 |

## 行為說明

### 描述如何顯示
*   **OverrideDescription 沒打勾**：把每個子設定的描述用換行串起來。空描述會跳過。
*   **OverrideDescription 打勾**：直接顯示「描述」欄位，子設定描述完全不參與。
*   **`GetDescriptionShort`（敵方意圖列）**：分隔符改為 `", "`（逗號），避免敵人頭頂顯示破版。

> [!TIP]
> 子設定描述串太長變成裹腳布時，**先試 OverrideDescription**，自己寫一段乾淨的整體描述。但記得手動同步維護！

### 卡片可不可以打出來
**全部** 子設定都判定可打出，整個組合才打得出來（AND 邏輯）。任一子設定失敗（例如資源不足），整張卡都不能用。

### 預覽傷害顯示 / 攻擊力聚合
取所有子設定中**最大**的那一個傷害值作為預覽（`GetPreviewDamage`，初始 -1）。`GetAtk` 同理取最大值（初始 0），用於攻擊力 buff 閾值預估。

### 攻擊標籤 / 卡片資訊
*   **`Infos`**：所有子設定的 `Infos` 會 `AppendIfNotRepeat` **去重**後合併顯示，避免同一個 Buff 圖示在不同子設定間連續出現多次。
*   **`GetBattleTags`**：直接 `Append` 串接，**不去重** —— 因 BattleTag 系統允許同 Tag 累計層數。
*   **`HasTerm`**：對子設定取 OR；任一帶該 Term 即整體 true（與 CheckPlayable 的 AND 相反）。
*   **`GetCollaborators`**：遞迴聚合並以 `Contains` 手動去重。

### 執行順序（AddAction）
整個 Combine 透過 `iData.AddActionTrigger` 包成一個 Trigger 入隊，外層採用呼叫端指定的 `iAddActionMode`（預設 `PushBack`）。
**內層子設定強制以 `InsertInOrder` 模式**入隊，確保 N 個子動作維持原清單順序、不被其他併發效果插斷。

## 注意事項

*   **保持子設定粒度單一**：每個子設定只負責一件事（造成傷害 / 抽牌 / 加狀態），由「組合效果」負責組裝。這樣方便重用、改動單一行為時不會牽動整體。
*   **避免巢狀過深**：組合效果裡再放組合效果雖然可以，但會讓描述系統與預覽傷害顯示變得反直覺。**建議最多兩層**。
*   **空清單也合法**：「組合效果」可以為空（不設定任何子項），但這樣這張卡就什麼都不會發生 — 屬於資料錯誤。
*   **被停用（`m_IsEnable = false`）的子設定**：所有 virtual override 都走 `CombineSettings` property（已過濾），但 `GetFusionCandidateSettings` / `GetFusionBaseSetting` 直接走 `m_CombineSettings` 並手動檢查 `IsEnable` —— 行為一致但實作分歧，新增 method 時要對齊。

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊
*   **檔案路徑**：[`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CombineSetting.cs`](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CombineSetting.cs)
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_CombineSetting` → 「組合效果」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_OverrideDescription` | `OverrideDescription`（未本地化） | `bool` | — | 控制 `m_Description` 的 GUI 可視性 |
| `m_Description` | 描述 | `RCG_LocalizeData` | `Description` | `[Conditional(nameof(m_OverrideDescription), false, true)]` 條件顯示 |
| `m_CombineSettings` | 組合效果 | `List<RCG_BattleSetting>` | `CombineSettings` | `[AlwaysExpendOnGUI]` 永遠展開 |

### A.3 重要 Method 摘要

| Method | 邏輯 | 備註 |
|---|---|---|
| `CombineSettings` (property) | `m_CombineSettings.GetEnableBattleSettings()` | 過濾掉 `m_IsEnable = false` 的項目；所有聚合 method 都應走此 property |
| `CheckPlayable` | 對所有子設定取 AND；任一 false 即整體 false | 短路求值 |
| `GetPreviewDamage` | 對子設定的 `GetPreviewDamage` 取 `Mathf.Max`，初始 -1 | -1 表示非攻擊牌 |
| `GetAtk` | 對子設定的 `GetAtk` 取 `System.Math.Max`，初始 0 | 0 表示無攻擊 |
| `Infos` (property) | 子設定 `Infos` 用 `AppendIfNotRepeat` 去重串接 | 純 UI 聚合 |
| `GetBattleTags` | 子設定 Tag 用 `Append` 串接，**不去重** | 容許同 Tag 累計層數 |
| `HasTerm` | 子設定取 OR | 與 CheckPlayable 相反 |
| `GetCollaborators` | 遞迴聚合 + 手動 `Contains` 去重 | |
| `GetDescription` | `m_OverrideDescription = true`：直接回 `m_Description.Name`<br>否則：以換行串接所有非空子描述 | |
| `GetDescriptionShort` | 子簡短描述用 `", "` 分隔串成單行 | 敵方意圖列防破版 |
| `GetDescriptionFormat` / `GetDescriptionParams` | 子描述以 `{ID}_{index}` 為命名空間前綴遞迴 | 避免不同子設定參數鍵衝突 |
| `GetBattleSettings<T> / (Type)` | 自身若符合則加入結果，再遞迴 `CombineSettings` | |
| `PreloadData` | 依序 `await` 每個子設定的 `PreloadData` | 取消 token 在下一個子設定前生效 |
| `AddAction` | 外層 `AddActionTrigger(..., iAddActionMode, this)` 包一次<br>內層子設定強制 `InsertInOrder` | 維持子動作順序不被插斷 |
| `GetFusionCandidateSettings` | 走 `m_CombineSettings` 直接過濾 `IsEnable`；遞迴攤平所有葉節點 | 給 CardFusionSetting 收集融合候選 |
| `GetFusionBaseSetting` | `CloneObject` 自身 → 清空子清單 → 子設定遞迴取 Placeholder 重建 | 保留結構不保留效果 |

### A.4 與其他系統的互動
*   **`RCG_LoopSetting`** 內含一個 `RCG_CombineSetting` 作為 `m_LoopContent`；Combine 是 Loop 的「執行單元」。
*   **`RCG_ForeachTargetSetting`** 同理，把 `RCG_CombineSetting` 當作 `m_Content`。
*   **`RCG_BattleTagCombineSetting`** 繼承 `RCG_CombineSetting`，特化處理 BattleTag 邏輯。
*   **`RCG_CardFusionSetting`** 透過 `GetFusionCandidateSettings` / `GetFusionBaseSetting` 兩個 virtual hook 與 Combine 互動。

### A.5 已知議題
*   `m_OverrideDescription` 與 `m_Description` 為兩個耦合欄位，舊資料中 `m_OverridingDescriptionKey` 已淘汰（程式中以 `// ` 註解保留）。
*   **實作分歧**：聚合 method 走 `CombineSettings`（property，已過濾 IsEnable），但 `GetFusionCandidateSettings` / `GetFusionBaseSetting` 走 `m_CombineSettings` 並手動檢查 IsEnable —— 行為一致但兩種寫法並存。新增聚合 method 請統一走 property。
