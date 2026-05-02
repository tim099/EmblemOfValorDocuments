---
title: 攻撃命令
description: 「攻撃」と類似だが攻撃者を指定可能；「他のユニットに攻撃させる」シーン用
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 攻撃命令

> クラス名：`RCG_AttackCommandSetting`

## 用途
「**攻撃**」設定とほぼ一致しますが、**攻撃者を指定するフィールドが追加**されています。「**他のユニットに攻撃させる**」シーンで使用：
*   召喚物が主人公の代わりに攻撃発動
*   仲間との協同攻撃
*   多人数連撃技

> [!NOTE]
> 通常の攻撃は「**攻撃**」設定 (`RCG_AttackSetting`) を優先；攻撃者を明示的に指定する必要があるときだけこの設定を使ってください。

## 主なフィールド

「**攻撃**」とほぼ同じ、ただし **Attacker（攻撃者）** が追加：

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Attacker** | はい | 攻撃発動者セレクタ（戦闘中の `User` を上書き）。 |
| **攻撃ターゲット** | はい | 「攻撃」と同じ。 |
| **攻撃力** | はい | 「攻撃」と同じ。 |
| **攻撃回数** | はい | 「攻撃」と同じ。 |
| **攻撃タイプ** | はい | 「攻撃」と同じ（`Counter` 反撃タイプは受けたダメージのタイプを自動継承）。 |
| **攻撃タグ** | いいえ | 「攻撃」と同じ。 |
| **攻撃効果** | はい | 「攻撃」と同じ。 |
| **詳細設定** | いいえ | 「攻撃」と同じ（特攻、AttackAtOnce など）。 |

## 挙動
*   `Attacker` を解析して実際の攻撃者を取得；解析失敗時（条件に該当するユニットが存在しない）は「全生存ユニットの最初」をフォールバック。
*   `IgnoreBuff = false` の場合、攻撃回数は攻撃者の `m_UnitStatus.GetAtkTimes(...)` で修飾。
*   説明：「**{攻撃者} が {対象} に {攻撃力} ダメージ**」（i18n key `AttackDommand_Des`）。
*   その他の挙動（プレビュー、特攻表示、AttackAtOnce など）は「攻撃」と同様。

## 注意点
*   **Attacker フォールバックに注意**：セレクタが攻撃者を解析できない場合、**生存ユニットの最初を強制使用**するため NRE 回避；ただし「敵命令」が誤って「味方攻撃」に流れる危険。Attacker セレクタは慎重に設計を。
*   **「攻撃」との違い**：両者のフィールドはほぼ重複；攻撃者指定が不要なら必ず「攻撃」を選択。
*   **Counter タイプ**：「受け攻撃」由来でない場合は `Normal`（物理）に降格。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_AttackCommandSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **`[System.Serializable]`** 標記
*   **i18n クラス名 key**：`RCG_AttackCommandSetting` → 「攻撃命令」
*   **TODO コメント**：`// TODO: 測試 QWQ 能可以跟 RCG_AttackSetting 共用邏輯?` — `RCG_AttackSetting` と大量重複コード、将来統合可能性。

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_Attacker` | `Attacker` | `RCG_SelectTargetData` | — | 攻撃発動者 |
| `m_AttackTarget` | 攻撃ターゲット | `RCG_SelectTargetData` | `AttackTarget` | |
| `m_Atk` | 攻撃力 | `IntVariable` | `Atk` | `[UCL_FieldOnGUI]` |
| `m_AtkTimes` | 攻撃回数 | `IntVariable` | `AtkTimes` | `[UCL_FieldOnGUI]` |
| `m_AttackType` | 攻撃タイプ | `AttackType` | `AttackType` | |
| `m_AttackTagGenDatas` | 攻撃タグ | `List<RCG_AttackTagGenData>` | `AttackTagGenDatas` | |
| `m_AttackVFX` | 攻撃効果 | `RCG_AttackVFXGenData` | `AttackVFX` | |
| `m_DetailSetting` | 詳細設定 | `RCG_AttackSetting.DetailSetting` | `DetailSetting` | **`RCG_AttackSetting.DetailSetting`** ネストクラスを共用 |

### A.3 主なメソッド
*   **`AddAction`**：`m_Attacker` を解析 → `aUser` を取得（フォールバック：`RCG_BattleManager.Ins.AllAliveUnits[0]`）。`m_DetailSetting.m_AttackAtOnce` に応じて分岐（一度に全攻撃 / 段毎再選択）、`AttackData` 構築 + `RCG_AttackAction` キュー追加。
*   **`GetDescriptionFormat`**：単攻撃説明を先に組立（`RCG_AttackSetting` と共有 `LocalizeKey` ロジック）、最後に i18n key `AttackDommand_Des` で攻撃者陳述を包む。

### A.4 他システムとの連携
*   **`RCG_AttackSetting.DetailSetting / DescriptionType`** — ネストクラスと enum を直接共用。
*   **`AttackData / RCG_AttackAction`** — 「攻撃」と同じ下流。
*   **`RCG_BattleField.CurActiveUnit` / `RCG_BattleManager.AllAliveUnits`** — 攻撃者解析のフォールバック。

### A.5 既知の課題
*   `RCG_AttackSetting` と重複コードが極めて多い（attack 力 / 回数 / VFX / DetailSetting / preview ロジック）— 共有基底か helper への refactor が望ましい。
