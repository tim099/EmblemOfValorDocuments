---
title: 玩家能量變化 說明
description: 改變玩家當前能量（費用）— 加 / 減 / 設定為指定值
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 玩家能量變化

> 程式類別名稱：`RCG_CostAlterSetting`

## 用途
**改變玩家當前的能量值（費用 / Cost）**。常見用途：
*   「補 1 能量」
*   「消耗 2 能量觸發強力效果」
*   「將能量設為 0」（過載）

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **CostAlter** | 是 | 變化量（變數可變）。 |
| **CostAlterType** | 是 | 變化模式：<br>• **Add** — 增加能量<br>• **Sub** — 減少能量（**會卡可玩判定**）<br>• **Set** — 設為指定值 |

## 行為說明
*   **可玩判定**：`Sub` 模式 + 純數值時，玩家當前能量必須 ≥ `CostAlter` 才能打出（避免出現負能量）。其他模式不檢查。
*   **觸發**：依模式呼叫 `RCG_Player.Ins.AlterCost(...)` 或 `SetCost(...)`。
*   **VFX**：增加會播 `ChargeEffect`，減少會播 `OverloadEffect`，並在角色身上跳出能量變化數字。
*   描述格式依模式套不同 i18n key（`AddCostIcon_Des` / `SubCostIcon_Des` / `SetCostIcon_Des`）+ 能量圖示。

### 卡片融合
僅同 `CostAlterType` 之間可融合（變化量加總）。

## 注意事項
*   **舊資料反序列化**：以前用「`CostAlter` 為負值」表示減少；現在用 `CostAlterType.Sub`。`DeserializeFromJson` 會自動把舊資料的負值轉換為 `Sub` + 正值。
*   **變數型攻擊力的 Sub 不檢查**：可玩判定只在 `m_VariableType == Value`（純數值）時生效；變數型減能量**不會擋打出**，可能讓能量短暫變負。
*   **Set 不受可玩判定限制**：永遠可打出，請小心避免「Set 0」型卡導致玩家失去戰術空間。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CostAlterSetting.cs`
*   **繼承自**：`RCG_BattleSetting`
*   **i18n 類別名 key**：`RCG_CostAlterSetting` → 「玩家能量變化」

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_CostAlter` | `CostAlter` | `IntVariable` | — | |
| `m_CostAlterType` | `CostAlterType` | enum (檔內) | — | `Add` / `Sub` / `Set` |

### A.3 重要 Method 摘要
*   **`CheckPlayable`**：`Sub` + 純數值時 `RCG_Player.Ins.Cost >= m_CostAlter`；其他模式 true。
*   **`DeserializeFromJson`**：舊資料負值 → 自動轉為 `Sub` + 正值（內部翻轉 `m_Value` 或 `m_Mult`）。
*   **`AddAction`** (async)：依模式呼叫 `RCG_Player.Ins.AlterCost(aValue, isCardCost: false)` 或 `SetCost(aValue)`，並播放對應 VFX (`ChargeEffect` / `OverloadEffect` + `VFX_Cost`)。
*   **`Fusion`**：要求同 `m_CostAlterType`；clone + `IntVariable.FuseAdd`。

### A.4 與其他系統的互動
*   **`RCG_Player.Ins.AlterCost / SetCost`**：能量修改入口。
*   **`RCG_CommonVFXGenData.s_ChargeEffect / s_OverloadEffect`**：VFX 來源。
*   **`CommonVFX.VFX_Cost`** (透過 `aCostVFX.SetAlterHP`)：能量變化數字浮動 UI。
