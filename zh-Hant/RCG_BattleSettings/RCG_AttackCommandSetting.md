---
title: 攻擊指令 說明
description: 與「攻擊」類似但可指定攻擊者；用於「指揮另一個單位攻擊」的情境
last_updated: 2026-05-05
target_audience: [Designer, Modder, AI_Agent]
---

# 攻擊指令

> 程式類別名稱：`RCG_AttackCommandSetting`

## 用途
和「**攻擊**」設定幾乎一致，但**多了一個欄位指定攻擊者** — 用於「**指揮另一個單位攻擊**」的情境，例如：
*   召喚物代替主角發動攻擊
*   隊友協同攻擊
*   多人連擊技

> [!NOTE]
> 一般攻擊請優先用「**攻擊**」設定（`RCG_AttackSetting`），此設定僅在需要明確指定攻擊者時才使用。

## 主要欄位

與「**攻擊**」幾乎相同，僅多一個 **Attacker（攻擊者）** 欄位：

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Attacker** | 是 | 攻擊發動者選擇器（取代戰鬥中當前的 `User`）。 |
| **攻擊目標** | 是 | 同「攻擊」設定。 |
| **攻擊力** | 是 | 同「攻擊」設定。 |
| **攻擊次數** | 是 | 同「攻擊」設定。 |
| **攻擊類型** | 是 | 同「攻擊」設定（`Counter` 反擊類型會自動繼承被攻擊的傷害類型）。 |
| **攻擊標籤** | 否 | 同「攻擊」設定。 |
| **攻擊特效** | 是 | 同「攻擊」設定。 |
| **詳細設定** | 否 | 同「攻擊」設定（特攻、AttackAtOnce 等）。 |

## 行為說明
*   解析 `Attacker` 拿到實際攻擊者；若拿不到（例如選擇器條件無人符合）會用「全部活著的單位」第一個作為防呆。
*   `IgnoreBuff = false` 時，攻擊次數會經過攻擊者的 `m_UnitStatus.GetAtkTimes(...)` 修飾。
*   描述格式為「**{攻擊者} 對 {目標} 造成 {攻擊力} 傷害**」（i18n key `AttackDommand_Des` 套版）。
*   其他行為（預覽傷害、特攻顯示、AttackAtOnce 等）與「攻擊」一致。

## 注意事項
*   **Attacker 防呆會踩雷**：若選擇器解析不到攻擊者，**會強制找一個活著的單位**頂上去 — 這雖避免 NRE，但可能讓「敵人指令」誤觸發成「我方攻擊」。請仔細設計 Attacker 選擇器。
*   **與「攻擊」的差別**：兩者欄位幾乎重複，請避免亂用 — 不需要指定攻擊者的場合一律用「攻擊」。
*   **Counter 類型**：未由「受到攻擊」觸發時會降級為 `Normal`（物傷）。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：[`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_AttackCommandSetting.cs`](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_AttackCommandSetting.cs)
*   **繼承自**：`RCG_BattleSetting`
*   **`[System.Serializable]`** 標記
*   **i18n 類別名 key**：`RCG_AttackCommandSetting` → 「攻擊指令」
*   **TODO 註解**：`// TODO: 測試 QWQ 能可以跟 RCG_AttackSetting 共用邏輯?` — 兩類有大量重複碼，未來可能合併。

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_Attacker` | `Attacker` | `RCG_SelectTargetData` | — | 攻擊發動者 |
| `m_AttackTarget` | 攻擊目標 | `RCG_SelectTargetData` | `AttackTarget` | |
| `m_Atk` | 攻擊力 | `IntVariable` | `Atk` | `[UCL_FieldOnGUI]` |
| `m_AtkTimes` | 攻擊次數 | `IntVariable` | `AtkTimes` | `[UCL_FieldOnGUI]` |
| `m_AttackType` | 攻擊類型 | `AttackType` | `AttackType` | |
| `m_AttackTagGenDatas` | 攻擊標籤 | `List<RCG_AttackTagGenData>` | `AttackTagGenDatas` | |
| `m_AttackVFX` | 攻擊特效 | `RCG_AttackVFXGenData` | `AttackVFX` | |
| `m_DetailSetting` | 詳細設定 | `RCG_AttackSetting.DetailSetting` | `DetailSetting` | **共用** `RCG_AttackSetting.DetailSetting` 巢狀類別 |

### A.3 重要 Method 摘要
*   **`AddAction`**：解析 `m_Attacker` → `aUser`（防呆：取 `RCG_BattleManager.Ins.AllAliveUnits[0]`）。依 `m_DetailSetting.m_AttackAtOnce` 分流（一次砸全部 / 逐段重選），建立 `AttackData` 並 `RCG_AttackAction`。
*   **`GetDescriptionFormat`**：先套單一攻擊描述（與 `RCG_AttackSetting` 共用 `LocalizeKey` 邏輯），再以 i18n key `AttackDommand_Des` 包覆攻擊者敘述。

### A.4 與其他系統的互動
*   **`RCG_AttackSetting.DetailSetting / DescriptionType`** — 直接共用巢狀類別與枚舉。
*   **`AttackData / RCG_AttackAction`** — 與「攻擊」一致的下游。
*   **`RCG_BattleField.CurActiveUnit` / `RCG_BattleManager.AllAliveUnits`** — 攻擊者解析的後備來源。

### A.5 已知議題
*   與 `RCG_AttackSetting` 重複碼極多（attack 力 / 次數 / VFX / DetailSetting / preview 邏輯）— 應重構為共用基底或 helper。
*   **i18n key 命名 typo**：`AttackDommand_Des` 應為 `AttackCommand_Des`，但改名會破壞舊資料故保留。
*   **本類無 GetPreviewDamage / Fusion 覆寫**：兩者都走父類預設（-1 / null），與 AttackSetting 行為不同 —— 設計師若想要預覽傷害請改用 AttackSetting。
