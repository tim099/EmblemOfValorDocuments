---
title: 回復
description: 選択ターゲットの HP を回復；固定値またはパーセンテージの2モード対応
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 回復

> クラス名：`RCG_HealSetting`

## 用途
**選択ターゲットの HP を回復**。最も単純な「ヒールカード」「治療スキル」「自己修復ステータス」が使う設定です。

## エディタ表示
```
▼ ✓ [回復(Heal)] (対象) に 1 回復
    HealType    [Amount / Percentage]
    治療量      [数値] 1
    対象の回復  ▶ (ターゲットセレクタ)
```

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **HealType** | はい | 回復モード：<br>• **数値 (Amount)** — 固定ポイントで回復、例：「+5 HP」。<br>• **百分比 (Percentage)** — 最大 HP の % で回復、例：「+30% HP」。 |
| **治療量** | はい | 数値（変数バインド可）。Amount モードはポイント、Percentage モードは 0~100 の % 値。 |
| **対象の回復** | はい | 治療対象セレクタ（自身 / 味方全体 / 選択対象など）。 |

> [!NOTE]
> **治療量** は変数（「残り手札数」「特定状態のスタック数」など）にバインド可能。ドロップダウンで切替えられます。

## 挙動

### バトル中の挙動
1. 「対象の回復」を解析して実際のターゲット一覧を取得。
2. 一覧が空でなければ、**HealType** に応じて各ターゲットを回復。
3. UI 上で緑色の数字（HP 上昇）が出ます。

### 説明文の表示
*   **HealType = 数値**：「{治療量} HP を {対象} に回復」（ハートアイコン付き）
*   **HealType = 百分比**：「{治療量}% HP を {対象} に回復」

### カードフュージョン
2枚の「回復」カードをフュージョンすると、**治療量が加算**されます（他のフィールドは前者基準）。例：「+3」+「+5」→「+8」。

## 注意点

*   **攻撃カードに「回復」を組み込まない**：「**コンボ効果**」でラップして「攻撃」と「回復」を別々の子設定にしてください — 2 つは直交した概念です。
*   **治療量に負数**：crash しませんがセマンティクス的に間違い（治療がマイナスでダメージ？）。**ダメージは「攻撃」設定**を使ってください。
*   **対象を「敵方」に設定**：合法ですが奇妙（敵に回復は debuff か冗談カード）；意図したものか確認を。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_HealSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_HealSetting` → 「回復」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_HealType` | `HealType` | `HealType` (enum) | `HealType` | 値の i18n：`HealType_Amount` (数値) / `HealType_Percentage` (百分比) |
| `m_HealAmount` | 治療量 | `IntVariable` | `HealAmount` | `[UCL_FieldOnGUI]` |
| `m_HealTarget` | 対象の回復 | `RCG_SelectTargetData` | `HealTarget` | 旧 `m_HealRange` を置換 |

### A.3 主なメソッド
*   **`AddAction`**：`m_HealTarget.GetTargets(iData)` でターゲット解析 → `RCG_PlayerHealAction(m_HealType, User, targets, m_HealAmount.GetValue, iData)` を生成 → `InsertAction` で挿入。
*   **`GetDescriptionFormat`**：`HealType.Amount` は i18n key `HealIcon_Des`；他は `m_HealType.GetLocalizeDes(...)` で対応フォーマットを取得。
*   **`GetDescriptionShort`**：`Percentage` → `{治療量}%`；他 → `{治療量}`。
*   **`Fusion(other)`**：相手も `RCG_HealSetting` 必須；clone 後 `IntVariable.FuseAdd` で治療量合算。

### A.4 他システムとの連携
*   **`RCG_PlayerHealAction`**：実際のヒール Action；アニメと HP 修正を担当。
*   **`RCG_SelectTargetData.GetTargets`**：共有ターゲットセレクタ。
*   **`IntVariable`**：シリアライズ可能な数値コンテナ；リテラル / 変数 / 隠し変数 3 モード対応。

### A.5 既知の課題
*   旧 `m_HealRange` フィールドと `DeserializeFromJson` の移行ロジックは廃止；現在は `m_HealTarget` 一本化。
