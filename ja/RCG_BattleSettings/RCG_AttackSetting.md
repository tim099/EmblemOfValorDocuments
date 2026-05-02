---
title: 攻撃
description: ターゲットにダメージを与える；攻撃力・回数・タイプ・タグ・特効・詳細パラメータを完備
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 攻撃

> クラス名：`RCG_AttackSetting`

## 用途
**指定ターゲットにダメージ**を与える。ゲーム中で最も使われ、フィールドが最も多いバトル設定。「敵を殴る」あらゆるカード・敵スキル・反撃ステータスがこれを使います。

## エディタ表示
```
▼ ?  ✓  [攻撃(Attack)] 1 物理ダメージ
    ▶ 攻撃ターゲット (対象)
      攻撃力          [数値] 1
      攻撃回数        [数値] 1
      攻撃タイプ      [物理(Normal)]
    ▶ 攻撃タグ(0)
    ▶ 攻撃効果        [通常攻撃エフェクト(打撃)(AttackEffectPlain)]
    ▶ 詳細設定
```

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **攻撃ターゲット** | はい | ターゲットセレクタ。範囲（単体 / AOE / 敵全体 / 味方全体…）と対象タイプを設定。 |
| **攻撃力** | はい | 1 ヒットあたりのダメージ（変数バインド可：「ブロック値」「特定状態の層数」など）。**負値は自動的に 0** にクランプ。 |
| **攻撃回数** | はい | 攻撃段数。各段は独立解決；詳細設定でターゲット再選択可否を制御。 |
| **攻撃タイプ** | はい | ダメージ計算方式に影響。下表参照。 |
| **攻撃タグ** | いいえ | 攻撃に付加するタグ（吸血・火・クリティカル…）。タグは独自ルールを持てます（例：吸血 = ダメージの X% を回復に変換）。 |
| **攻撃効果** | はい | ヒット時の特効データ；ベース攻撃特効を必ず1つ選択（さもないと振りモーションが見えない）。 |
| **詳細設定** | いいえ | 高度パラメータ。下記の§詳細設定を参照。 |

### 攻撃タイプ対照
| 表示 | 内部 | 物理意味 |
|---|---|---|
| **物理 (Normal)** | Normal | ブロック・物理減免の影響を受ける |
| **魔法 (Magic)** | Magic | 魔法抵抗の影響を受ける |
| **生命削減 (Status)** | Status | ブロック・全減免を無視（状態ダメージは攻撃扱いにならない） |
| **反撃 (Counter)** | Counter | 反撃用；タイプは受けたダメージのタイプに合わせる |
| **任意 (Any)** | Any | 任意攻撃タイプ；トリガー条件のワイルドカード |

## 詳細設定（高度）

| エディタ表示 | デフォルト | 説明 |
|---|---|---|
| **特攻 (AntiAttack)** | （空） | 特定モンスタータグへの倍率ダメージ。例：「ドラゴン」を選ぶと「対ドラゴン特攻」。 |
| **AttackAtOnce** | ✗ | チェック = 同段で全ターゲットを一斉攻撃（モーション同期）；オフ = ターゲット個別。 |
| **StopAttackAfterDeath** | ✓ | 多段攻撃で目標が倒れたら残り段をキャンセル（空振り防止）。 |
| **CounterAttackVFX** | ✗ | 反撃時に受けた攻撃の特効を流用。 |
| **IgnoreBuff** | ✗ | 攻撃者の Buff/Debuff を無視（表示と計算の両方）。 |
| **説明タイプ (DescriptionType)** | Default | 説明文の文型；`InflictDamageOn` で「対 X に Y ダメージ」式。 |

## 挙動

### プレビューダメージ（カード右上の小数字）
*   **「攻撃ターゲット」が「選択した対象を攻撃」のときのみ**計算（他の範囲モード（AOE / 自身など）は「非攻撃カード」扱い）。
*   現在適用中の Buff/Debuff/特攻を計入。
*   各段独立に予測 — **攻撃回数を掛けない**、表示は単段値。

### 攻撃力表示（説明文中の数字）
*   IgnoreBuff オフ：**Buff 適用後**の攻撃力を表示（+X 調整値の形式含む）。
*   IgnoreBuff オン：生の `攻撃力` 値を表示。

### カード情報（ホバーツールチップ）
自動列挙：
1. 攻撃タイプ（物理 / 魔法 …）
2. 各攻撃タグ（吸血、火 …）
3. 「特攻」設定がある場合：倍率ルール + 対象モンスター
4. 「攻撃ターゲット」由来の追加情報

## 注意点

*   **多段 + AOE は二重再選択**：「攻撃回数 = 3 + AOE = 全敵」は段ごとに**全敵を再解決**（途中死亡で対象数減少）。「全敵を各 3 回攻撃」したい場合は「**繰り返し発動(ループ)**」で AOE 攻撃をラップ。
*   **特攻と説明文の不一致**：「特攻」が「対ドラゴン 1.5 倍」設定だがカード説明は「対ドラゴン 2 倍」と書くと矛盾 — **両方とも手動メンテ**なので確認必須。
*   **攻撃効果は空にできない**：必ず `AttackEffectXxx` を 1 つ選択。「無視覺反撃」でも `AttackEffectPlain` を選んでください。
*   **変数型攻撃力の表示**：変数を使うと説明に `(変数名)` が自動付与。クリーンな表示が欲しい場合は「隠し変数」モードで（最終数字のみ表示）。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_AttackSetting.cs`
*   **継承元**：`RCG_BattleSetting`、`[System.Serializable]`
*   **i18n クラス名 key**：`RCG_AttackSetting` → 「攻撃」

### A.2 フィールド対照（メインクラス）

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_AttackTarget` | 攻撃ターゲット | `RCG_SelectTargetData` | `AttackTarget` | 旧 `AttackRange` enum を置換 |
| `m_Atk` | 攻撃力 | `IntVariable` | `Atk` | `[UCL_FieldOnGUI]` |
| `m_AtkTimes` | 攻撃回数 | `IntVariable` | `AtkTimes` | `[UCL_FieldOnGUI]` |
| `m_AttackType` | 攻撃タイプ | `AttackType` (enum) | `AttackType` | サブ key：`AttackType_Normal/_Magic/_Status/_Counter/_Any` |
| `m_AttackTagGenDatas` | 攻撃タグ | `List<RCG_AttackTagGenData>` | `AttackTagGenDatas` | エイリアス：`AttackTags`、`AttackTag` |
| `m_AttackVFX` | 攻撃効果 | `RCG_AttackVFXGenData` | `AttackVFX` | |
| `m_DetailSetting` | 詳細設定 | `DetailSetting` (内部クラス) | `DetailSetting` | A.3 参照 |

### A.3 フィールド対照（`DetailSetting` ネストクラス）

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_AntiAttack` | 特攻 | `List<RCG_MonsterTagGenData>` | `AntiAttack` | |
| `m_AttackAtOnce` | `AttackAtOnce` | `bool` | `AttackAtOnce` | |
| `m_StopAttackAfterDeath` | `StopAttackAfterDeath` | `bool` | `StopAttackAfterDeath` | デフォルト `true` |
| `m_CounterAttackVFX` | `CounterAttackVFX` | `bool` | `CounterAttackVFX` | |
| `m_IgnoreBuff` | `IgnoreBuff` | `bool` | `IgnoreBuff` | |
| `m_DescriptionType` | 説明タイプ | `DescriptionType` (enum) | `DescriptionType` | `Default` / `InflictDamageOn` |

### A.4 主なメソッド
*   **`Infos`** → `[AttackType, ...AttackTagGenDatas, AntiAttakDes(特攻あり時), ...AttackTarget.Infos]` を組み立て、最後の段で重複排除。
*   **`GetAtkStr`**：user あり + 非 IgnoreBuff → `iUser.p_BattleUnit.m_UnitStatus.GetAtkDescription(...)` で Buff 適用済表示。`Value` / `HiddenVariable` モードでは `iHasModification = false`。
*   **`GetAttackData`** → プレビュー用 `AttackData`（反撃処理なし）。
*   **`GetAtk`** → `Mathf.Max(0, m_Atk.GetValue(iData))`。
*   **`GetPreviewDamage`** → `m_AttackTarget.m_SelectTargetType == Range && m_TargetPos.m_UnitRange == Target` のみ計算；他は `-1`。
*   **`DeserializeFromJson`** → base 呼び出しのみ；旧マイグレーションはコメントアウト。

### A.5 他システムとの連携
*   **`AttackData / RCG_PlayerAtkAction`** 系列 → 実際の攻撃 Action ランタイムパス。
*   **`RCG_SelectTargetData / SelectTargetType / UnitRange`** → 共有ターゲットセレクタ；旧 `AttackRange` enum は廃止予定だが旧 JSON 互換のため残存。
*   **`IntVariable`** → 攻撃力 / 回数の数値コンテナ。
*   **`RCG_MonsterTagGenData`** → 特攻判定用モンスタータグ。

### A.6 既知の課題
*   `AttackRange` enum は `RCG_SelectTargetData` で置換済みだが、旧 JSON デシリアライズ用に保持。
*   旧 `CanEnhence` / `Enhence(RestSetting)` はコメントアウト（強化系リファクタ中）。
*   `GetCommentLine` 系のコメント領域に廃止された説明ロジック 4 箇所が保存されている。
