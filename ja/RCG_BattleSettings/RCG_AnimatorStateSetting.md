---
title: アニメーション状態 (AnimatorState)
description: ユニットの Animator 状態切替または1回だけのアニメ再生；純粋に視覚（説明文表示なし）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# アニメーション状態

> クラス名：`RCG_AnimatorStateSetting`

## 用途
**純粋なアニメーション駆動**で、ゲーム挙動への影響なし：
*   **持続的状態切替**（例：「防御姿勢に入る / 抜ける」）
*   **1回だけのアニメ再生**（例：「振り」「瞬き」）

> [!NOTE]
> この設定は**カード説明には表示されません**（`GetDescriptionFormat` も `GetDescriptionShort` も空文字を返す）。純粋に視覚的演出。

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **ActionType** | はい | アクションタイプ：<br>• **SetState** — 持続的な Animator bool 状態を切替（例：防御姿勢）。<br>• **PlayAnim** — 1回限りのアニメを再生し終了を待つ。 |
| **AnimType** (`UnitAnimType`) | はい | 操作するアニメタイプ（攻撃・被弾・防御…）。 |
| **Enable** | ActionType=SetState 必須 | bool；持続状態の切替先（true = 入る、false = 抜ける）。`PlayAnim` モードでは自動で隠れる。 |
| **対象** | はい | アニメ再生のターゲットユニットセレクタ。 |

## 挙動
*   **SetState** モード：各ターゲットの `UnitAnimController.SetAnimState(AnimType, Enable)` を呼ぶ。
*   **PlayAnim** モード：各ターゲットで await `PlayAnim(AnimType)`（**バトルフローを完了まで止める**）。
*   AnimController がないターゲットは黙ってスキップ。

## 注意点
*   **PlayAnim は後続アクションを遅延**：await 式なのでバトルテンポが伸びます；ループ内に置く場合は注意。
*   **SetState ペアの整合**：`SetState(true)` を発動したら対応する `SetState(false)` も用意。さもないと戦闘終了まで状態が持続。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_AnimatorStateSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key なし**：エディタ表示は stripped name `AnimatorState`

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_ActionType` | `ActionType` | `AnimActionType` (enum) | — | `SetState` / `PlayAnim` |
| `m_AnimType` | `AnimType` | `UnitAnimType` (enum) | — | アニメタイプ |
| `m_Enable` | `Enable` | `bool` | `Enable` | `[Conditional(nameof(m_ActionType), false, AnimActionType.SetState)]` |
| `m_Target` | 対象 | `RCG_SelectTargetData` | `Target` | |

### A.3 主なメソッド
*   **`GetDescriptionFormat / GetDescriptionShort`** どちらも `string.Empty`（意図的に隠す）。
*   **`AddAction`** → `AddAsyncActionTrigger` ラップ、`ActionType` で分岐。

### A.4 他システムとの連携
*   **`UnitAnimController.SetAnimState / PlayAnim`**：実際のアニメ駆動エントリ。
*   **`UnitAnimType` enum**：Spine animation 名と対応。
