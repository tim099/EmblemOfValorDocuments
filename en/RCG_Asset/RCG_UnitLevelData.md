---
title: 單位等級資料 (RCG_UnitLevelData) 說明
description: 定義「單位的 HP / 攻擊力如何隨等級成長」的曲線；難度系統與此搭配影響怪物強度
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-en
---

> [!WARNING]
> Translation pending — this file needs an English translation.
The original zh-Hant content is included below for reference.


# 單位等級資料

> 程式類別名稱：`RCG_UnitLevelData`

## 用途

定義「**單位的 HP 與攻擊力如何隨等級成長**」的曲線。每個怪物 / 角色可以引用一份 `RCG_UnitLevelData`；遊戲中根據**當前難度**換算等級，再透過此曲線決定該等級下的 HP 倍率與攻擊力倍率。

繼承自 `RCG_Asset<RCG_UnitLevelData>`。

## 編輯器中的樣貌

```
RCG_UnitLevelData: <ID>
    LevelCurve  ← 等級曲線：難度 0~1 → 等級成長值
    HPCurve     ← HP 曲線：等級 0~1 → HP 倍率
    AtkCurve    ← 攻擊曲線：等級 0~1 → 攻擊倍率
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **LevelCurve** | 是 | 等級曲線。輸入 `aDifficultyVal`(0~1)，回傳的是要加到 `BaseLevel` 上的等級值 |
| **HPCurve** | 是 | HP 倍率曲線。輸入等級值 (0~1)，回傳 HP 倍率 |
| **AtkCurve** | 是 | 攻擊倍率曲線。同上 |

每條 `CurveData` 內部：
*   **CurveType**：`Linear`（線性 `Lerp(min, max, x)`）或 `Power`（次方 `pow(base, x * segment)`）
*   **MinVal / MaxVal**（Linear）：起始值與最大值
*   **PowerBase / PowerSegment**（Power）：次方底數與分段數

## 行為說明

### 等級換算 (`GetLevel(BaseLevel)`)
1. 取**當前難度**：`RCG_BigMapManager.Difficulty + RCG_DataService.Ins.Difficulty`。
2. 把難度映射到 0~1 範圍（**分段函式**）：
    *   0~10 → 0.00~0.10（線性）
    *   10~50 → 0.10~0.30
    *   50~100 → 0.30~0.40
    *   100+ → 0.40~1.00
3. `LevelCurve.GetValue(0~1) → 加成`，`Level = BaseLevel + round(加成)`，clamp 在 [1, 100]。

### 屬性計算
*   `GetMaxHP(baseHP, level)` = `round(baseHP × HPCurve(level/99) × DifficultyData.HealthMult)`
*   `GetAtkMult(level)` = `AtkCurve(level/99) × DifficultyData.AtkMult`
注意「全局難度倍率」(`DifficultyData.HealthMult / AtkMult`) 會在這層再乘一次。

### 預覽
編輯器內預設 Preview 只顯示 ID 標籤，不繪製曲線視覺化（要看曲線形狀只能去看數值）。

## 注意事項

*   **MaxLevel = 100**：上限寫死，超過會被 clamp。修改要連動 `m_PowerSegment` 預設值。
*   **MaxDifficulty = 100**（理論值）：實際 100+ 的難度會繼續按 0.001 線性成長，但 `aDifficulty > 1` 時會把 `aDifficulty` 設為 1（**這看起來是 bug**：應該設 `aDifficultyVal`）。
*   **`Debug.LogWarning` 在 `GetMaxHP` / `GetAtkMult` 內**：每次計算都會印 log，會洗版；release 前要拿掉。
*   **全局難度倍率**（`DifficultyData.HealthMult / AtkMult`）會在這層再乘一次；總體強度 = 等級曲線 × 全局倍率。
*   **`m_LevelCurve` 預設 `Linear(0, 100)`** 表示「滿難度時加 100 級」，搭配 `BaseLevel = 1` 就會抵到 MaxLevel。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_UnitLevelData.cs`
*   **繼承自**：`RCG_Asset<RCG_UnitLevelData>`
*   **AssetGroup**：`EditBattleSetting`
*   **常數**：`MinLevel = 1` / `MaxLevel = 100` / `MaxDifficulty = 100`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_LevelCurve` | LevelCurve | `CurveData` | 預設 `Linear(0, 100)` |
| `m_HPCurve` | HPCurve | `CurveData` | 預設 `Linear(1, 10)` |
| `m_AtkCurve` | AtkCurve | `CurveData` | 預設 `Linear(1, 4)` |

### A.3 重要 Method 摘要

*   **`Difficulty` (static property)** — `BigMapManager.Difficulty + DataService.Difficulty`。
*   **`GetLevel(BaseLevel)`** — 主入口；內含分段難度→0~1 的映射。
*   **`GetMaxHP(baseHP, level)` / `GetAtkMult(level)`** — 套曲線 + 全局難度倍率。
*   **`GetLevelValue(level)`** (private) — `(level - 1) / (MaxLevel - 1f)`，把 1~100 對應到 0~1。

### A.4 與其他系統的互動

*   **`RCG_BigMapManager.Difficulty`** — 大地圖層級的難度。
*   **`RCG_DataService.Ins.Difficulty`** — 玩法 / 挑戰 runtime 難度。
*   **`RCG_DataService.Ins.m_DifficultyData.m_HealthMult / m_AtkMult`** — 全局難度倍率。
*   **`RCG_UnitLevelGenData`**（檔內）— Asset Entry 包裝，預設 ID = `"Default"`。

### A.5 已知議題

*   `GetLevel` 內 `aDifficultyVal > 1` 時改的是 `aDifficulty = 1`（型別不對，疑似 bug；應改 `aDifficultyVal = 1`）。
*   `GetMaxHP` / `GetAtkMult` 內 `Debug.LogWarning` 會洗版。