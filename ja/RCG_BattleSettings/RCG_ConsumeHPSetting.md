---
title: HP を消費する
description: 対象の HP を削る（固定値または現在 / 最大 HP の %）；自損型効果に使用
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# HP を消費する

> クラス名：`RCG_ConsumeHPSetting`

## 用途
**対象の HP を削る**（攻撃扱いではないため、**ブロックの影響を受けない**）。よくある用途：
*   「自身の HP を 5 削る代わりに大量リソース獲得」
*   「現在 HP の 30% を消費して強力な効果発動」

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **ConsumeType** | はい | 消費モード：<br>• **CurHPPercentage** — 現在 HP のパーセンテージ（デフォルト）<br>• **MaxHPPercentage** — 最大 HP のパーセンテージ<br>• **Value** — 固定数値 |
| **量** (`Amount`) | はい | 消費量（変数可）；パーセンテージモードでは 0~100。 |
| **対象** (`Target`) | はい | 対象セレクタ（通常は自身）。 |

## 挙動
*   `ConsumeType` で実際の削減量を計算：
    *   `CurHPPercentage` → `0.01 * Amount * 当前 HP` を四捨五入
    *   `MaxHPPercentage` → `0.01 * Amount * 最大 HP` を四捨五入
    *   `Value` → 直接 `Amount`
*   `target.UnitHeal(-aConsumeAmount)` で HP 削減。
*   数値は `RCG_BattleManager.Ins.m_BattleStats.LogStatsCostHealth(...)` に記録。
*   説明形式：`ConsumeHP_{ConsumeType}_Des`（ハートアイコン付き）。

## 注意点
*   **パーセンテージモードの丸め**：`RoundToInt` 使用。境界値（例：1% × 50 HP = 0.5）は 0 か 1 に丸められる。設計時は**整数で予測可能**であることを基本に。
*   **直接撃殺はしない**：HP 0 で死亡判定が走るが、「強制撃殺」したい場合は「**即死**」を使ってください。
*   **攻撃扱いではない**：ブロック・反撃・ダメージ修飾は適用されない — 純粋な HP 削減。
*   **対象を敵方に**：技術的には合法（敵の HP を削る）ですが、これは通常「攻撃」設定の方が適切 — 2 つはセマンティクスが違う。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ConsumeHPSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_ConsumeHPSetting` → 「HP を消費する」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_ConsumeType` | `ConsumeType` | enum | — | 3 モード |
| `m_Amount` | 量 | `IntVariable` | `Amount` | `[UCL_FieldOnGUI]`、デフォルト 50 |
| `m_Target` | 対象 | `RCG_SelectTargetData` | `Target` | |

### A.3 主なメソッド
*   **`AddAction`**：ConsumeType で `aConsumeAmount` を計算し、`aTarget.UnitHeal(-aConsumeAmount)` を呼び、最後に `LogStatsCostHealth`。
*   **`GetDescriptionFormat`**：i18n key `ConsumeHP_{ConsumeType}_Des`、ハートスプライトを含む。
*   **`GetDescriptionShort`** → 直接 `m_Amount.GetDes(true)`。

### A.4 他システムとの連携
*   **`RCG_BattleUnit.UnitHeal(int)`**：負値で HP 削減（治療エントリと共有）。
*   **`RCG_BattleManager.m_BattleStats.LogStatsCostHealth`**：戦闘統計記録。
