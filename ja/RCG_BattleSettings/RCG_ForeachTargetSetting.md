---
title: 各ターゲットに個別発動
description: 選択した各ターゲットに対し、内包の組み合わせ効果を個別に発動（AOE を単体ループに分解）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 各ターゲットに個別発動

> クラス名：`RCG_ForeachTargetSetting`

## 用途
**各ターゲットに個別に効果を発動**。AOE を「**敵ごとに 1 回**」のループに分解。よくある用途：
*   「血量 10 以下の各敵を即死」（AOE 即死は分解判定する必要あり）
*   「全敵を 5 HP 回復（演出を同期）」

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Targets** | はい | 個別発動するターゲットセレクタ（通常は AOE）。 |
| **Content** | はい | 各ターゲットに対し実行する「**組み合わせ効果**」。 |

## 挙動
*   `Targets` を解析してターゲットリスト取得。
*   各ターゲットに対し：child `TriggerEffectData` を作成し、`Targets` をそのターゲット単体に設定 → `Content.AddAction` を呼ぶ。
*   説明形式：「**各 {Targets} に対し別々に {Content 説明}**」（i18n key `ForeachTargetDes`）。

## 注意点
*   **「繰り返し発動」との違い**：「繰り返し」は「**同じターゲットに N 回**」；本設定は「**N 個の異なるターゲットに 1 回ずつ**」。
*   **プレビューダメージは適用されない**：本設定は**`GetPreviewDamage` を override しない**ため、親クラス既定値 `-1`（非攻撃カード）に。ダメージ表示が必要なら別の方法で表現を。
*   **ネストの Foreach**：Foreach 内に Foreach は N×M ループ — 説明が極めて複雑になるため避けて。
*   **空ターゲットリスト**：合法だが何も発動しない。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ForeachTargetSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_ForeachTargetSetting` → 「各ターゲットに個別発動」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_Targets` | `Targets` | `RCG_SelectTargetData` | — | |
| `m_Content` | `Content` | `RCG_CombineSetting` | — | 子効果 |

### A.3 主なメソッド
*   **`AddAction`**：`m_Targets.GetTargets(iData)` 内の各 target に対し `iData.CreateChildData(target.BattleName)` → `data.Targets = [target]` → `m_Content.AddAction(data, InsertInOrder)`。
*   **`m_Content` への透過**：`Infos` / `HasTerm` / `GetCollaborators` / `GetAtk`。
*   **`GetBattleSettings<T> / (Type)`**：自身 + `m_Content` を再帰。
*   **`GetFusionCandidateSettings`** → `m_Content` の候補に直接委譲。
*   **`GetFusionBaseSetting`**：自身 clone、内容を placeholder 化された Combine に置換。

### A.4 他システムとの連携
*   **`TriggerEffectData.CreateChildData / Targets`**：各ターゲットごとの独立 child data；子効果が Targets を共有して干渉しないことを保証。
*   **`LocalizedStringUtils.CapitalizeString`**：暫定の先頭大文字化処理（LoopSetting と共有 hack）。
