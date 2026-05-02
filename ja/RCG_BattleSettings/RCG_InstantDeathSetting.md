---
title: 即死
description: ターゲットを直接撃殺；無条件即死または HP が指定値以下で即死の 2 モード
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 即死

> クラス名：`RCG_InstantDeathSetting`

## 用途
**ターゲットを直接撃殺**（HP / ブロック無視）。よくある用途：
*   「斬首」「処刑」系の超強カード：HP が低い目標を即死
*   特殊イベントの「無条件撃殺」（劇情演出 / 呪い）

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Type** | はい | 即死モード：<br>• **Default** — 無条件で全ターゲットを即死<br>• **HP** — ターゲット HP ≤ 指定値で即死 |
| **対象** (`Target`) | はい | 即死対象セレクタ。 |
| **HP** | Type=HP 必須 | 即死の血量閾値；ターゲットの血量 ≤ この値で即死。`Conditional` で表示制御。 |

## 挙動
*   **Default**：全ターゲットに対し `RCG_TermInstantDeath.TriggerInstantDeath(iData, targets)` を呼ぶ（用語トリガーを含む）。
*   **HP**：各ターゲットを独立判定 `target.HP <= m_HP`、合致なら `target.InstantDeath()` を呼ぶ。
*   説明形式：
    *   `Default` → 「**{対象} を即死**」
    *   `HP` → 「**血量が {HP} 以下のとき {対象} を即死**」

### カード情報
`RCG_Term.GetTerm(Term.InstantDeath)` の用語枠を表示（即死メカニズムを説明）。

## 注意点
*   **Default モードは強力**：ボスにも直接撃殺。「即死無視」状態がない限り瞬殺されます。設計時は慎重に。
*   **HP モードは AOE で公平**：各ターゲットを独立判定するため、「AOE だが最初の 1 体しか死なない」という他の即死実装の差を避ける。
*   **負値 HP**：`m_HP` を負値にすると永久に発動しない（ターゲット血量は 0 未満にならない）；「絶対に即死しない」と等しい。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_InstantDeathSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_InstantDeathSetting` → 「即死」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_Type` | `Type` | `InstantDeathType` (内部 enum) | — | `Default` / `HP` |
| `m_Target` | 対象 | `RCG_SelectTargetData` | `Target` | |
| `m_HP` | `HP` | `IntVariable` | `HP` | `[Conditional(nameof(m_Type), false, HP)]`、デフォルト 1 |

### A.3 主なメソッド
*   **`AddAction`**：`m_Type` で分岐：
    *   `Default` → `RCG_TermInstantDeath.TriggerInstantDeath(iData, targets)`。
    *   `HP` → 各 target を `target.HP <= m_HP.GetValue(iData)` でチェック、合致なら `target.InstantDeath()`。
*   **`Info`** → `new CardInfoData(RCG_Term.GetTerm(Term.InstantDeath))`。
*   **`GetDescriptionFormat`**：`InstantDeath_Des` (Default) / `InstantDeath_HP_Des` (HP)。

### A.4 他システムとの連携
*   **`RCG_TermInstantDeath.TriggerInstantDeath`**：用語トリガーエントリ、特効演出含む。
*   **`RCG_BattleUnit.InstantDeath`**：HP=0 を直接設定し死亡フローを走るエントリ。
*   **`Term.InstantDeath`**：対応する用語 enum。

### A.5 既知の課題
*   `Condition` モード（`m_Conditions`）はコメントアウト — 「条件付き即死」をやりたい場合は「条件判断」+「即死(Default)」の組み合わせを使ってください。
