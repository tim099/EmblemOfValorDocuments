---
title: 自定義狀態 (RCG_CustomStatusData) 說明
description: 戰鬥中可加在單位身上的狀態（buff / debuff / DoT 等）：層數變化、抵銷、免疫、抗性、攻防 buff
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-ja
---

> [!WARNING]
> 翻訳待機中 — このファイルは日本語翻訳が必要です。
参考用に zh-Hant 原文を以下に掲載しています。


# 自定義狀態

> 程式類別名稱：`RCG_CustomStatusData`

## 用途

**戰鬥中可加在單位身上的狀態（buff / debuff）模板**。例如「中毒：每回合扣 5 血」「力量+2」「免疫負面狀態」「閃避：層數 ≥ 攻擊力時免傷」。本類別把所有狀態相關屬性集中一處：層數變化規則、抵銷關係、免疫 / 抗性、攻防 buff 加成、單位特殊狀態（暈眩、詠唱⋯）。

繼承自 `RCG_Asset<RCG_CustomStatusData>`，實作介面：`RCGI_Status`。

## 編輯器中的樣貌

```
RCG_CustomStatusData: <ID>
    Name / StatusEffectType / StatusClass
    AtkBuff / DefBuff           ← 攻防加成設定
    IconSprite / Tags
    LayerIncreaseVFX / StatusPersistantVFX
    StatusOffsets / OffsetTags  ← 互相抵銷的狀態
    StatusAlterOn               ← 哪些 trigger 上層數會變
    Effects                     ← 觸發效果
    UnitStates                  ← 對應的特殊狀態（暈眩、閃避⋯）
    StopDecayStatus             ← 使目標狀態不衰減
    ImmuneStatus                ← 免疫狀態（不再獲得）
    Resistance                  ← 抗性（每層 1% 機率使狀態無效）
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **Name** | 是 | 狀態名（多語系） |
| **StatusEffectType** | 是 | `Buff` / `Debuff` / 其他類型；影響顯示色與抵銷判斷 |
| **StatusClass** | — | `Normal` / `AtkBuff` / `DefBuff` / `Immune`（免疫所有負面） |
| **AtkBuff / DefBuff** | 否 | 多個 atk / def buff 設定（傷害倍率、護甲加成⋯） |
| **IconSprite** | 是 | 狀態圖示（`RCG_IconSprite` 引用） |
| **Tags** | 否 | 狀態分類標籤（Buff / Debuff / DoT / Feature⋯） |
| **LayerIncreaseVFX** | — | 層數增加時的特效 |
| **StatusPersistantVFX** | — | 常駐單位特效（持有此狀態時恆顯示） |
| **StatusOffsets** | 否 | 互相抵銷的具體狀態（例：力量 ↔ 虛弱） |
| **OffsetTags** | 否 | 與哪些**標籤類型**抵銷（例：所有 Debuff 都被某 buff 抵銷） |
| **StatusAlterOn** | 否 | 哪些 trigger 上層數變化（dictionary：`trigger → DecreaseType`） |
| **Effects** | 否 | 各 trigger 觸發的效果 |
| **UnitStates** | 否 | 對應的單位特殊狀態列舉值（`Stun` / `Guard` / `Dodge`⋯） |
| **StopDecayStatus** | 否 | 使目標狀態不衰減的清單 |
| **ImmuneStatus** | 否 | 免疫的狀態（不再獲得） |
| **Resistance** | 否 | 每層 1% 機率使目標狀態無效 |

## 行為說明

### 層數變化 (`GetDecreaseType`)
查 `m_StatusAlterOn` 表，依 trigger 取得對應的 `eStatusDecreaseType`：
*   `None` 不變 / `DecreaseOneLayer` -1 / `Clear` 全清 / `DecreaseHalf` 減半
*   `AddOneLayer` +1 / `ClearImmediately` 立刻清 / `Eliminate` 消除（不觸發結束效果）

### 抵銷判斷 (`CheckIsOffset`)
*   `StatusClass = Immune` → 所有 Tag 含 `StatusDebuff` 的狀態都會被抵銷。
*   `OffsetTags` 命中對方 Tag → 抵銷。
*   `StatusOffsets` 含對方 ID → 抵銷。

### 描述生成
分兩段（中間空行）：
1. **抵銷段**：列出此狀態會抵銷的對象。
2. **效果段**：UnitStates 描述 / Atk-DefBuff 描述 / StopDecay / Immune / Resistance / Effects 觸發描述。

### IconTMPKey
`RCG_BattleSetting.IsShowOnUI = true` 時回傳 IconSprite 的 TMPKey（給 TextMeshPro 圖示用）；否則回 LocalizedName。

### 觸發效果 (`OnTriggerEffect`)
從 `m_Effects` 取對應 trigger 並依序觸發。

## 注意事項

*   **`Immune` 類型自動抵銷所有 Debuff**：不需手動列 `OffsetTags = StatusDebuff`，邏輯內建。
*   **`m_AtkBuff / m_DefBuff` 是 list**：可疊多個（不同條件下不同加成）；舊版 `m_AtkBuffData / m_DefBuffData` 已被取代並註解。
*   **`UnitStates` 直接對應 enum**：暈眩、閃避、格擋等是寫死在 enum 內的 13 種特殊狀態；無法新增（要改程式）。
*   **Resistance 是機率制**：每層 1%，10 層 = 10% 抗性，效果不疊到 100%。
*   **StatusFeature Tag**：含此 tag 的狀態名稱前面會加「特性: 」前綴。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_CustomStatusData.cs`
*   **繼承自**：`RCG_Asset<RCG_CustomStatusData>`
*   **實作介面**：`RCGI_Status`
*   **AssetGroup**：`EditCharacter`

### A.2 欄位對照（節選）

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_StatusEffectType` | StatusEffectType | `StatusEffectType` enum | `Buff` / `Debuff` |
| `m_StatusClass` | StatusClass | `StatusClass` enum | `Normal` / `AtkBuff` / `DefBuff` / `Immune` |
| `m_AtkBuff` | AtkBuff | `List<RCG_AtkBuffData>` | |
| `m_DefBuff` | DefBuff | `List<RCG_DefBuffData>` | |
| `m_IconSprite` | IconSprite | `RCG_IconSpriteGenData` | |
| `m_Tags` | Tags | `List<RCG_StatusTagGenData>` | |
| `m_LayerIncreaseVFX` | LayerIncreaseVFX | `RCG_CommonVFXGenData` | 預設 `VFX_StatusLayerID` |
| `m_StatusPersistantVFX` | StatusPersistantVFX | `RCG_CommonVFXGenData` | |
| `m_StatusOffsets` | StatusOffsets | `List<RCG_CustomStatusGenData>` | |
| `m_OffsetTags` | OffsetTags | `List<RCG_StatusTagGenData>` | |
| `m_StatusAlterOn` | StatusAlterOn | `Dictionary<RCG_EffectTriggerOn, eStatusDecreaseType>` | |
| `m_Effects` | Effects | `List<RCG_CommonEffect>` | |
| `m_UnitStates` | UnitStates | `List<UnitState>` | enum: Stun / Guard / Dodge / etc. |
| `m_StopDecayStatus` | StopDecayStatus | `List<RCG_CustomStatusGenData>` | |
| `m_ImmuneStatus` | ImmuneStatus | `List<RCG_CustomStatusGenData>` | |
| `m_Resistance` | Resistance | `List<RCG_CustomStatusGenData>` | |

### A.3 重要 Method 摘要

*   **`GetDecreaseType(triggerOn)`** — 查 m_StatusAlterOn 表。
*   **`CheckIsOffset(targetStatus)`** — 三層判斷：Immune class / OffsetTags / StatusOffsets。
*   **`OnTriggerEffect(triggerOn, data)`** — 觸發效果。
*   **`TriggerOnUnitState(triggerOn)`** — 是否會在此 trigger 變化或觸發效果（quick check）。
*   **`Description` (property)** — 大型 StringBuilder 組合：Offsets + UnitStates + AtkBuff + DefBuff + StopDecay + Immune + Resistance + Effects。
*   **`IconTMPKey`** — UI 模式下回 TMPKey，否則回 LocalizedName。
*   **`CreateLayerIncreaseVFX / CreateStatusPersistantVFX`** — async VFX 建立。

### A.4 與其他系統的互動

*   **`RCG_StatusGenData`** — Asset Entry 包裝；`Status` (property) 回 `new RCG_StatusGenData(ID)`。
*   **`RCG_CustomStatusGenData`** — 直接引用此資料的型別；含 `s_Default` / `s_ChargeUp` 預設實例。
*   **`RCG_AtkBuffData` / `RCG_DefBuffData`** — 攻防 buff 子資料。
*   **`UnitState` (enum)** — 13 種寫死的特殊狀態。
*   **`RCG_VFXManager`** — 特效建立。

### A.5 已知議題

*   舊版 `m_AtkBuffData` / `m_DefBuffData` 單一欄位已被 list 取代，反序列化遷移邏輯已註解。
*   `Init()` 是空殼，預留進入大地圖時的初始化 hook。