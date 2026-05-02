---
title: 重複觸發(迴圈) 說明
description: 把指定的組合效果重複執行 N 次的控制流戰鬥設定
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 重複觸發(迴圈)

> 程式類別名稱：`RCG_LoopSetting`

## 用途
把一段「**組合效果**」**重複執行 N 次**。常見用途：
*   「造成 1 點傷害，重複 5 次」（連擊類）
*   「每剩一張手牌就觸發一次」（變數綁定的連擊）
*   「給予 1 層敏捷，重複 3 次」（疊狀態層數）

## 編輯器中的樣貌
```
▼ ✓ [重複觸發(迴圈)(Loop)] (LoopContent) × N
    要重複的次數         [數值] 1
    要重複執行的效果     ▶ (展開後是「組合效果」容器)
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **要重複的次數** | 是 | 重複次數；支援變數綁定（例如「等於剩餘卡片數」「等於某狀態層數」）。 |
| **要重複執行的效果** | 是 | 一個「組合效果」容器，內含實際要重複的設定。 |

> [!NOTE]
> 為什麼內容欄位是「組合效果」而不是直接放一串設定？因為「組合效果」已經處理了多 Setting 的描述合成、Tag 去重、預覽合計等邏輯，包一層省事。**單一效果的迴圈也請放一個只有一項子設定的「組合效果」**。

## 行為說明

### 戰鬥中發生什麼
1. 系統解析「要重複的次數」拿到 N。
2. 連續執行 N 次「要重複執行的效果」中的內容。
3. 每圈動作都依序插入戰鬥佇列，與外層的設定串接執行。

### 描述會怎麼顯示
格式為「**重複 N 次：{內容描述}**」，會自動把內容首字大寫。

精簡描述（敵方意圖列）：直接顯示 `{內容} × {N}`。

### 預覽傷害不乘 N
卡片預覽傷害顯示的是「**單圈**」的傷害，不會自動乘以重複次數。例如「重複 3 次造成 4 點傷害」預覽顯示 4，**不顯示 12**。

> [!IMPORTANT]
> 這是設計選擇 — 玩家看「每次傷害」較直觀。若你要強調總傷害，請在卡片描述中明寫「**3 × 4 = 12 點**」。

## 注意事項

*   **重複次數 = 0 或負值**：技術上不會 crash，但等於「卡牌沒效果」，玩家會以為 bug。請在公式上夾 `Mathf.Max(1, ...)` 或設計上避開負值情境。
*   **巢狀迴圈**：可以「重複觸發」內再包「重複觸發」，但描述會變成「重複 M 次：重複 N 次：⋯」**對玩家極不友善**。請改用 `M*N` 攤平。
*   **多目標 + 迴圈**：「攻擊全敵 + 重複 3 次」≠「對每個敵人各 3 次」。前者是「3 次 AOE，每次重新解析目標」；後者請用「**迴圈每個目標(Foreach)**」設定。
*   **AOE 攻擊放在迴圈裡**：每圈會重新解析目標，所以中途死亡的敵人會在後續圈被跳過，符合直覺。

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_LoopSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_LoopSetting` → 「重複觸發(迴圈)」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_LoopTimes` | 要重複的次數 | `IntVariable` | `LoopTimes` | 支援變數型 |
| `m_LoopContent` | 要重複執行的效果 | `RCG_CombineSetting` | `LoopContent` | 包一層 Combine 為了重用其聚合邏輯 |

### A.3 重要 Method 摘要
*   **`AddAction`**：
    ```csharp
    int aLoopTimes = m_LoopTimes.GetValue(iData);
    for (int i = 0; i < aLoopTimes; i++)
        m_LoopContent.AddAction(iData, AddActionMode.InsertInOrder);
    ```
    **重點**：每圈用 `InsertInOrder`，保證動作順序與外層串接。
*   **`GetDescriptionFormat`**：
    1. 暫存 `iData.m_FullSentence`，設為 `false` 取乾淨內容描述。
    2. `LocalizedStringUtils.CapitalizeString` 把首字大寫（暫時 hack）。
    3. 復原 `m_FullSentence = true`，套 i18n key `LoopSettingDes`。
*   **`GetDescriptionShort`** → i18n key `LoopSettingDesShort`，格式為 `{Content} × {Times}`。
*   **透傳到 `m_LoopContent`**：`Infos` / `HasTerm` / `GetCollaborators` / `GetAtk` / `GetPreviewDamage` / `PreloadData`。
*   **`GetBattleSettings<T> / (Type)`** → 自身 + 遞迴 `m_LoopContent`。
*   **`GetFusionCandidateSettings`** → 直接代理 `m_LoopContent.GetFusionCandidateSettings()`（**Loop 自身不作為候選**）。
*   **`GetFusionBaseSetting`** → clone 自身結構，內容換為 placeholder 化的 Combine。

### A.4 與其他系統的互動
*   **`RCG_CombineSetting`** → `m_LoopContent` 的容器型態；Loop 透過 Combine 重用「多 Setting 描述合成」邏輯。
*   **`AddActionMode.InsertInOrder`** → 確保多圈動作維持插入順序，與 `PushBack` 不同。
*   **`IntVariable.GetValue`** → 解析重複次數；不夾值，所以 0 / 負數會直接 0 圈。

### A.5 已知議題
*   `LocalizedStringUtils.CapitalizeString` 是「暫時特殊處理」（程式註解 `//暫時特殊處理`），i18n 重構時應有正規方案。
*   `GetPreviewDamage` 不乘 N 是設計選擇而非 bug；若需求轉變請在此明文修改。
*   舊版 `CanEnhence` / `Enhence` 註解保留（強化系統舊邏輯）。
