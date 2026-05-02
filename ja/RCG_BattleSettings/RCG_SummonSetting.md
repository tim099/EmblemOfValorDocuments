---
title: 召喚
description: ユニットを戦場に召喚（敵側または味方）；召喚タイプ（通常 / 死亡位置）を選択可能
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 召喚

> クラス名：`RCG_SummonSetting`

## 用途
**ユニットを戦場に召喚**。よくある用途：
*   味方友軍を召喚（治療師、補助、肉壁）
*   敵が雑兵を召喚
*   特殊「死亡ユニット位置で召喚」効果

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **UnitData** | はい | 召喚ユニットのデータ；ネストの `RCG_UnitSummonData`、召喚タイプ・ユニット・モンスターか否か等を含む。 |

> [!NOTE]
> ネストクラス `RCG_UnitSummonData` が実際の設定先：
> *   `m_Unit` — 召喚するユニットタイプ
> *   `m_IsMonster` — デフォルト陣営（**敵方トリガー時に自動反転**）
> *   `m_SummonType` — `Default`（空き位置を探す）/ `UnitDeadPosiiton`（死亡ユニットの位置で召喚）
> *   `m_Pos` — 召喚位置（前 / 後 / All）

## 挙動
*   **陣営反転**：モンスター方がトリガーした場合、`IsMonster` が反転（モンスターが味方を召喚 = プレイヤー視点では敵）。
*   **Default モード**：`iData.TargetPositions[0]` で指定された空き位置を優先；指定なしは自動的に空き位置を探す。
*   **UnitDeadPosiiton モード**：戦死したユニットの位置に召喚（蘇生 / 屍位メカニズム）。
*   `VFX_Summon` の特効を再生し、召喚アニメ完了後にユニットの `OnBattleStart`（入場効果）をトリガー。
*   説明形式：`SummonDes_{Monster|Player|UnitDeadPosiiton}` + ユニット名。

## 注意点
*   **空き位置がないとき暗黙的失敗**：見つからないと `Debug.LogError` を出すがプレイヤーには通知せず — カードが「無効果」に見えるかも。
*   **召喚数の上限**：戦場の最大ユニット数による制限；満員なら召喚不可。
*   **入場効果のトリガー**：`OnBattleStart` がトリガーされ、その時点での能力が即発動。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_SummonSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_SummonSetting` → 「召喚」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_UnitData` | `UnitData` (=「戦闘キャラクター」) | `RCG_UnitSummonData` | `UnitData` | `[AlwaysExpendOnGUI]` |

### A.3 主なメソッド
*   **`AddAction`** (async)：
    1. `iData.Faction == UnitFaction.Enemy` で `aIsMonster` を反転。
    2. `m_SummonType` で召喚位置を決定（`Default` 空き位置探索 / `UnitDeadPosiiton` 戦死ユニット位置）。
    3. `RCG_BattleField.SpawnUnit` でユニット生成 + `VFX_Summon` 特効 + `SummonAnim` 待機 + `OnBattleStart`。
*   **`LocalizeKey` (private)**：SummonType と IsMonster で対応 i18n key を選択。
*   **`Info / GetDescriptionFormat`**：召喚ユニット情報と説明を表示。

### A.4 他システムとの連携
*   **`RCG_UnitSummonData`**：召喚データコンテナ（ユニット・タイプ・位置・陣営）。
*   **`RCG_BattleField.SpawnUnit / TryGetEmptyPositions`**：戦場生成エントリ。
*   **`CommonVFX.VFX_Summon`**：召喚特効。
*   **`RCG_Unit.SummonAnim / OnBattleStart`**：召喚アニメと入場効果トリガー。

### A.5 既知の課題
*   旧版 `TriggerAsync` と一部召喚ロジックがコメントとして保存。
