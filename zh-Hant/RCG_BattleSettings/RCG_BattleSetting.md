---
title: 戰鬥設定（基底）說明
description: 所有戰鬥設定（攻擊、治療、條件、組合等）的共通基底；說明所有子類共用的「啟用」開關、描述系統與下拉選單分類
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 戰鬥設定（基底）

> 程式類別名稱：`RCG_BattleSetting`

## 用途
這是「戰鬥中可發生的一個效果」的**基底類別**，並不是你會直接建立的東西，而是所有戰鬥設定（攻擊、治療、組合、條件…）的母體。當你在卡牌、敵人技能、狀態效果等資料上看到「**戰鬥設定**」欄位，下拉打開能選的選項，就是它的子類。

## 編輯器中的樣貌
任何 `RCG_BattleSetting` 子類在 Inspector 都會長這樣：
```
▼ ?  ✓  [類型(英文)] 摘要描述
```
*   **左側勾選框**：對應 `啟用` 欄位。打勾才會在戰鬥中觸發；不打勾的設定會被外層容器（如「組合效果」）整批跳過。
*   **`?` 圖示**：顯示對應的本份說明文件（前提是該子類有掛 `[HelpURL]`）。
*   **`[類型]` 標籤**：顯示子類的中文名稱（如「攻擊」「治療」「組合效果」），來源為 i18n key `RCG_XxxSetting`；後方括號是英文識別。
*   **後方文字**：呼叫該子類自身產生的「短描述」（Short Name），用於在折疊狀態快速辨識。

## 共通欄位

| 編輯器顯示 | 對應程式 | 說明 |
|---|---|---|
| **啟用** | `m_IsEnable` | 是否在戰鬥中觸發；外層容器會自動過濾掉未啟用的設定。 |

> [!NOTE]
> 不同子類額外的欄位，請查看各自的說明（例如「攻擊」會有「攻擊力」「攻擊次數」等）。

## 可選擇的子類型

下拉選單中可選的子類分為兩個族群（會自動切換）：

### 一般卡牌資料下的選項
編輯卡牌、狀態、強化等資料時提供，含 50 餘種，例如：
*   **效果類**：攻擊、防禦、治療、狀態 …
*   **資源類**：抽牌、棄牌、加卡費、消耗道具 …
*   **控制流**：組合效果、條件判斷、重複觸發、隨機觸發、迴圈每個目標 …
*   **進階**：召喚、單位變形、團體合擊、計數器調整 …

### 編輯怪物（敵方）資料時的額外選項
編輯 `RCG_UnitData` 或 `RCG_MonsterActionData` 時，會多出兩個怪物專用設定：
*   **怪物逃跑** (`RCG_MonsterFleeSetting`)
*   **怪物移動** (`RCG_MonsterMoveSetting`)

## 共通行為（你不必設定，但會看到）

*   **描述自動本地化**：每個子類會根據自己的欄位值產生「**詳細描述**」（卡牌資訊面板）和「**精簡描述**」（敵方意圖列）。
*   **預載資源**：戰鬥前系統會自動把該設定相關的圖檔、特效預先載入，避免戰鬥中讀取卡頓 — 你**不需要手動處理**。
*   **融合**：在卡牌融合機制下，每個子類定義自己的合併規則（例如兩張「治療」融合會把治療量加總）。

## 注意事項

*   **未啟用的設定不會被當作「不存在」**：它仍會佔資料空間、在 Inspector 顯示，只是執行時被跳過。要徹底移除請按右側 `−` 按鈕。
*   **新增的子類若 GUI 看不到選項**：那是程式端的 `AllTypes` 清單沒有把它加進去 — 屬於程式設定問題，請聯繫程式人員，不是資料端能解決的。
*   **`RCG_BattleSetting` 本身不能被選**：下拉清單只列出實質可用的子類；基底類別只是介面契約。

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_BattleSetting.cs`
*   **繼承自**：`UnityJsonSerializable`
*   **介面實作**：`UCLI_ShortName` / `UCLI_TypeList` / `UCLI_GetTypeName` / `UCLI_IsEnable` / `UCLI_NameOnGUI`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_IsEnable` | 啟用 | `bool` | `IsEnable` | `[UCL_HideOnGUI]`，由 `NameOnGUI` 自行繪製。 |
| `IsShowOnUI` | （無 GUI） | `static bool` | — | 全域 UI 開關，影響部分子類描述分支。 |
| `s_Types` / `s_MonsterDataTypes` | （無 GUI） | `static List<Type>` | — | 子類選單清單；`AllTypes` getter 依當前資料類型擇一回傳。 |

### A.3 重要 Method 摘要
*   **`virtual CheckPlayable(TriggerEffectData)`** → 預設 `true`；子類覆寫做資源 / 條件檢查。
*   **`virtual AddAction(TriggerEffectData, AddActionMode)`** → **核心入口**，把對應 Action 推入 `RCG_BattleManager` 佇列。基底為空實作。
*   **`virtual GetDescription / GetDescriptionFormat / GetDescriptionParams`** → 描述生成三件組；推薦覆寫 `Format + Params`，`GetDescription` 預設用兩者組合。
*   **`virtual GetDescriptionShort`** → 敵方意圖列精簡描述，預設空字串。
*   **`virtual GetShortName`** → Inspector 折疊狀態的縮寫名，預設取 `GetDescription` 截 25 字。
*   **`virtual Info / Infos`** → 卡片懸停資訊；`Infos` 預設聚合單個 `Info`。
*   **`virtual GetBattleTags / HasTerm / GetCollaborators`** → 標籤、辭條、協力者查詢；容器子類務必遞迴。
*   **`virtual GetAtk / GetPreviewDamage`** → 攻擊力 / 預覽傷害；後者回傳 `-1` 視為非攻擊牌。
*   **`virtual PreloadData(CancellationToken)`** → 戰鬥前資源預載；容器子類務必 `await` 子設定。
*   **`virtual Fusion / GetFusionCandidateSettings / GetFusionBaseSetting`** → 卡片融合三件組；預設不支援 / 自身為候選 / placeholder 化。
*   **`implicit operator string`** → 呼叫 `GetDescription()`；字串拼接方便但小心 null。

### A.4 與其他系統的互動
*   **`AllTypes` getter**：判斷 `UCLI_Asset.s_CurOnGUIAsset` 是否為 `RCG_UnitData` 或 `RCG_MonsterActionData`，是則回傳 `s_MonsterDataTypes`（多兩個 `RCG_Monster*Setting`），否則回傳一般 `s_Types`。
*   **JSON 序列化**：透過繼承 `UnityJsonSerializable`，預設由 `JsonConvert.SaveDataToJson(this, JsonConvert.SaveMode.Unity)` 處理（已被註解掉，走 base 預設）。
*   **GUI 繪製**：透過 `NameOnGUI` 在 Inspector 第一行繪製 `IsEnable` 勾選 + `[類型] 短描述`；類型名來自 `GetTypeName(GetType().Name)`，會嘗試以 i18n key（去掉 `RCG_` 前綴與 `Setting` 後綴）查表。

### A.5 已知議題
*   `CanEnhence` / `Enhence(RestSetting)` 已註解（舊強化系統殘留）。
*   `DeserializeFromJson / SerializeToJson` 也被註解（走 base 預設），若要客製須解開註解。
