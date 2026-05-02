---
title: 攻擊設定說明
description: 對目標造成傷害的戰鬥設定；含攻擊力、攻擊次數、類型、標籤、特效與進階詳細設定
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 攻擊

> 程式類別名稱：`RCG_AttackSetting`

## 用途
對**指定目標**造成傷害。遊戲中最常見、欄位最多的戰鬥設定，幾乎所有「會打怪」的卡片、敵人技能、反擊狀態都用它。

## 編輯器中的樣貌
```
▼ ?  ✓  [攻擊(Attack)] 給予 1 物傷
    ▶ 攻擊目標（目標）
      攻擊力    [數值] 1
      攻擊次數  [數值] 1
      攻擊類型  [物傷(Normal)]
    ▶ 攻擊標籤(0)
    ▶ 攻擊特效  [普通攻擊特效(打擊)(AttackEffectPlain)]
    ▶ 詳細設定
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **攻擊目標** | 是 | 目標選擇器。展開後可設定範圍（單體 / AOE / 全敵 / 全友 …）與目標類型。 |
| **攻擊力** | 是 | 一次攻擊造成的傷害（支援變數綁定，如「等於護甲值」「等於某狀態層數」）。**負值會被自動夾到 0**。 |
| **攻擊次數** | 是 | 攻擊段數。每段獨立結算，搭配「詳細設定」可決定是否每段重選目標。 |
| **攻擊類型** | 是 | 影響傷害結算方式。詳見下方對照表。 |
| **攻擊標籤** | 否 | 攻擊上附帶的標籤（吸血、火、爆擊…）。標籤本身可帶進階規則，例如「吸血」會把傷害的 X% 轉為治療。展開可加多個。 |
| **攻擊特效** | 是 | 命中動畫資料；至少要選一個基礎攻擊特效，否則玩家看不到揮砍動畫。 |
| **詳細設定** | 否 | 進階參數，見下方 §進階設定。 |

### 攻擊類型對照
| 顯示 | 內部代號 | 物理意義 |
|---|---|---|
| **物傷** | Normal | 受護甲與物傷減免影響 |
| **法傷** | Magic | 受魔抗與法傷減免影響 |
| **生命削減** | Status | 無視護甲與一切減免（狀態傷害不視為一般攻擊） |
| **反傷** | Counter | 反擊用；類型會與被攻擊的傷害類型一致 |
| **傷害** | Any | 任意攻擊類型；用於泛用觸發條件 |

## 進階設定（展開「詳細設定」）

| 編輯器顯示 | 預設 | 說明 |
|---|---|---|
| **特攻** (`AntiAttack`) | （空） | 對特定怪物標籤造成加倍傷害。例如選「龍族」即「對龍特攻」。空清單代表無特攻。 |
| **AttackAtOnce** | ✗ | 打勾 = 同一段攻擊一次砸到全部目標（同步動畫）；不打勾 = 逐個目標依序攻擊。 |
| **StopAttackAfterDeath** | ✓ | 多段攻擊時，目標若被擊殺則停止後續段數（避免「砍空氣」）。 |
| **CounterAttackVFX** | ✗ | 反擊時使用受到的攻擊動畫（沿用對方的揮砍特效）。 |
| **IgnoreBuff** | ✗ | 忽略攻擊者所有 Buff/Debuff（顯示與結算都會無視）。 |
| **描述類型** (`DescriptionType`) | Default | 影響卡片描述句型；選 `InflictDamageOn` 會用「對 X 造成 Y 傷害」的句法。 |

## 行為說明

### 預覽傷害（卡片右上角小數字）
*   **僅在「攻擊目標」設為「攻擊選中目標」**時計算（其他範圍模式如 AOE 或自身回傳「非攻擊牌」）。
*   會把當前已套用的 Buff/Debuff/特攻 都計入。
*   每段傷害獨立預覽 — **不乘以攻擊次數**，玩家看到的是單段值。

### 攻擊力顯示（描述中的數字）
*   未勾選 IgnoreBuff 時：顯示「**已套 Buff 後**」的攻擊力（含 +X 調整值的格式）。
*   勾選 IgnoreBuff 時：顯示原始 `攻擊力` 數值。

### 卡片資訊（懸停 Tooltip）
會自動列出：
1. 攻擊類型（物傷 / 法傷 …）
2. 每個攻擊標籤（吸血、火 …）
3. 若有「特攻」：列出加倍規則 + 對哪些怪物有效
4. 「攻擊目標」帶來的額外資訊

## 注意事項

*   **多段 + AOE 是雙倍重選**：「攻擊次數 = 3 + AOE = 全敵」預設每段都會**重新解析全敵**（敵人陣亡後就少一個目標）。要做「全敵各打 3 次」請改用「**重複觸發(迴圈)**」包一個 AOE 攻擊。
*   **特攻和描述要對齊**：如果「特攻」設了「對龍 1.5 倍」但卡片描述自己寫「對龍 2 倍」，玩家會看到不一致 — 兩處都是手動維護，**請仔細核對**。
*   **攻擊特效不可空**：一定要選一個 `AttackEffectXxx`，否則戰鬥中無動畫播放會 NRE。即使只想做「無視覺反擊」也請選 `AttackEffectPlain`。
*   **變數型攻擊力的顯示**：`攻擊力` 用變數時，描述會自動串接 `(變數名)`。如果想要乾淨的顯示，請挑「隱藏變數」型態，描述只會出現最終數字。

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_AttackSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **`[System.Serializable]`** 標記
*   **i18n 類別名 key**：`RCG_AttackSetting` → 「攻擊」

### A.2 欄位對照（主類別）

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_AttackTarget` | 攻擊目標 | `RCG_SelectTargetData` | `AttackTarget` | 取代舊的 `AttackRange` enum |
| `m_Atk` | 攻擊力 | `IntVariable` | `Atk` | `[UCL_FieldOnGUI]` |
| `m_AtkTimes` | 攻擊次數 | `IntVariable` | `AtkTimes` | `[UCL_FieldOnGUI]` |
| `m_AttackType` | 攻擊類型 | `AttackType` (enum) | `AttackType` | 列舉 i18n：`AttackType_Normal` (物傷) / `_Magic` (法傷) / `_Status` (生命削減) / `_Counter` (反傷) / `_Any` (傷害) |
| `m_AttackTagGenDatas` | 攻擊標籤 | `List<RCG_AttackTagGenData>` | `AttackTagGenDatas` | 同 key 也覆蓋 `AttackTags` / `AttackTag` 等別名 |
| `m_AttackVFX` | 攻擊特效 | `RCG_AttackVFXGenData` | `AttackVFX` | |
| `m_DetailSetting` | 詳細設定 | `DetailSetting` | `DetailSetting` | 巢狀類別，欄位見 A.3 |

### A.3 欄位對照（`DetailSetting` 子類別）

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_AntiAttack` | 特攻 | `List<RCG_MonsterTagGenData>` | `AntiAttack` | 對特定怪物標籤加倍傷害 |
| `m_AttackAtOnce` | `AttackAtOnce`（未本地化） | `bool` | — | 一次砸全部目標 vs 逐個 |
| `m_StopAttackAfterDeath` | `StopAttackAfterDeath`（未本地化） | `bool` | — | 預設 `true` |
| `m_CounterAttackVFX` | `CounterAttackVFX`（未本地化） | `bool` | — | 反擊用對方動畫 |
| `m_IgnoreBuff` | `IgnoreBuff`（未本地化） | `bool` | — | 忽略攻擊者 Buff/Debuff |
| `m_DescriptionType` | 描述類型 | `DescriptionType` (enum) | `DescriptionType` | `Default` / `InflictDamageOn` |

### A.4 重要 Method 摘要
*   **`Infos`** → 組合 `[AttackType, ...AttackTagGenDatas, AntiAttakDes(若有特攻), ...AttackTarget.Infos]`，最後一段去重。
*   **`GetAtkStr(iUser, iIsShowPoint, iData)`**：
    *   非 `IgnoreBuff` 且 user 非 null → 透過 `iUser.p_BattleUnit.m_UnitStatus.GetAtkDescription(...)` 套上 Buff 修飾。
    *   `m_VariableType` 為 `Value` 或 `HiddenVariable` 時 `iHasModification = false`（不顯示調整值格式）。
*   **`GetAttackData(iData, iUser, iAttackTargets)`** → 預覽用 `AttackData`（不含反擊處理）。
*   **`GetAtk(iData)`** → `Mathf.Max(0, m_Atk.GetValue(iData))`。
*   **`GetPreviewDamage`** → 僅 `m_AttackTarget.m_SelectTargetType == Range && m_TargetPos.m_UnitRange == Target` 時計算，否則回傳 -1；其餘走 `iTarget.GetDamage(FinalAtk, AttackData)`。
*   **`DeserializeFromJson(JsonData)`** → 呼叫 base，無客製欄位轉換（舊轉換邏輯已註解）。

### A.5 與其他系統的互動
*   **`AttackData`** → 攻擊資料封裝，含 `FinalAtk` 計算。
*   **`RCG_PlayerAtkAction`** 系列 → 實際的攻擊 Action（透過 `AddAction` 觸發；本檔未直接呼叫，繼承父類流程）。
*   **`RCG_SelectTargetData` / `SelectTargetType` / `UnitRange`** → 通用目標選擇器；`AttackRange` enum 是淘汰前的舊型態。
*   **`IntVariable`** → 攻擊力 / 攻擊次數的數值容器。
*   **`RCG_MonsterTagGenData`** → 特攻判定用的怪物標籤。

### A.6 已知議題
*   `AttackRange` enum 已被 `RCG_SelectTargetData` 取代，但仍保留枚舉定義 — 反序列化舊 JSON 用。
*   舊的 `CanEnhence` / `Enhence(RestSetting)` 已註解（強化系統重構中）。
*   `GetCommentLine 註解區塊` 中保留了 4 處被棄用的描述邏輯（`//public DescriptionType m_DescriptionType` 等）。
