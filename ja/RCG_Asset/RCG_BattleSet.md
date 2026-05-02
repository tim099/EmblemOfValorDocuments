---
title: 戰鬥組合 (RCG_BattleSet) 說明
description: 一場具體戰鬥的怪物配置 + 標籤 + 敵人類型；地圖節點實際進入戰鬥時用此資料生成
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-ja
---

> [!WARNING]
> 翻訳待機中 — このファイルは日本語翻訳が必要です。
参考用に zh-Hant 原文を以下に掲載しています。


# 戰鬥組合

> 程式類別名稱：`RCG_BattleSet`

## 用途

**一場具體戰鬥的怪物配置**。例如「3 哥布林 + 1 哥布林大廚」「Boss 鯨魚單體」「精英遭遇：龍 + 法師」都是不同的 BattleSet。地圖節點觸發戰鬥時，從 `RCG_BattleSetDropPool` 抽出一個 `RCG_BattleSet` 來生成本場戰鬥。

繼承自 `RCG_Asset<RCG_BattleSet>`。

## 編輯器中的樣貌

```
RCG_BattleSet: <ID>
    EnemyType  ← 敵人類型（普通 / 精英 / Boss）
    Tags       ← BattleSet 標籤（用於 DropPool 篩選）
    MonsterSet ← 怪物配置（站位、ID、獎勵資源等）
    Preview    ← 即時預覽配置
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **MonsterSet** | 是 | 完整的怪物配置（`RCG_MonsterSet`：每個站位放哪隻怪物、會掉哪些獎勵資源） |
| **Tags** | 否 | BattleSet 標籤（章節 / 場景類型等），DropPool 用標籤過濾 |
| **EnemyType** | 是 | 敵人類型（`Normal` / `Elite` / `Boss`），影響獎勵、戰鬥音樂、UI 標題 |

## 行為說明

### 取預設
`RCG_BattleSet.DefaultBattleSet` 直接取 `RCG_BattleSetGenData.DefaultID` 對應的資料；缺漏時的 fallback。

### 預覽
顯示 ID + EnemyType、Tags 列表、`MonsterSet.Preview` 的怪物站位視覺化。

## 注意事項

*   **EnemyType 是 Tag 物件**而非 enum：原本是 enum (`m_EnemyType = EnemyType.Normal`)，已改用 `RCG_EnemyTypeTagGenData` 並把舊欄位註解；新增類型直接編 Tag asset 即可。
*   **MonsterSet 內的獎勵資源**：曾有過 `m_MonsterSet.m_RewardResources.Clear()` 的反序列化清理（已註解掉）；若需特殊清理邏輯可解註參考。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSet.cs`
*   **繼承自**：`RCG_Asset<RCG_BattleSet>`
*   **AssetGroup**：`EditBattleSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_MonsterSet` | MonsterSet | `RCG_MonsterSet` | 主要配置容器 |
| `m_Tags` | Tags | `List<RCG_BattleSetTagGenData>` | DropPool 用 |
| `m_EnemyType` | EnemyType | `RCG_EnemyTypeTagGenData` | 取代舊 enum |

### A.3 重要 Method

*   **`Preview` / `OnGUI`** — 編輯器渲染。
*   **`DefaultBattleSet`** (static) — `Util.GetData(RCG_BattleSetGenData.DefaultID)`。

### A.4 與其他系統的互動

*   **`RCG_MonsterSet`** — 怪物站位 + 獎勵的詳細容器。
*   **`RCG_BattleSetDropPool`** — 戰鬥組合的隨機池。
*   **`RCG_BattleSetGenData`** — Asset Entry 包裝。

### A.5 已知議題

*   舊版 `m_EnemyType` (enum) 已註解，標示遷移成 Tag 型別的歷史。