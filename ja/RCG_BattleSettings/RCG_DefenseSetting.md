---
title: ブロック
description: ブロック値を増加 / 減少 / クリア / 倍化 / 自然減衰 / 設定
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# ブロック

> クラス名：`RCG_DefenseSetting`

## 用途
**ブロック値を変更**。最も多く使われる「防御カード」「Buff カード」のコア、複数の変化モードに対応。

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **DefenseAlterType** | はい | 変化モード：<br>• **Add** — ブロック増加<br>• **Sub** — ブロック減少<br>• **Clear** — ブロックを 0 にする<br>• **Double** — ブロックを倍に<br>• **Decay** — 自然減衰（**特定状態でブロック可**、Clear と異なる）<br>• **Set** — 指定値に設定 |
| **ブロック** (`Defense`) | Add/Sub/Set 必須 | ブロック量（変数可）。`Conditional` で表示制御 — `Clear/Double/Decay` モードでは自動的に隠れる。 |
| **DefenseTarget** | はい | ブロック適用対象セレクタ（旧 `m_Range` を置換）。 |

## 挙動
*   DefenseAlterType に応じて適用：
    *   `Add` / `Sub` / `Set` — 数値を直接加減 / 設定
    *   `Clear` — 強制 0 化（**いかなる状態にも遮断されない**）
    *   `Decay` — 減衰（**「持盾」など特定状態に阻止される**）
    *   `Double` — 現在のブロックを倍化
*   説明はモードに応じて i18n key を切替（`DefIcon_DesAdd` / `DefIcon_DesSub` / `DefIcon_DesClear` / `DecayDes` / ...）+ ブロックアイコン。

## 注意点
*   **Decay と Clear の違い**：`Clear` は強制 0 化、`Decay` は防減衰系の状態に阻まれる — 「ターン終了時の自然減衰」は必ず `Decay` を使う。
*   **Sub は負値にならない**：ブロックの下限は 0；多く引いても 0 に止まり負にはならない。
*   **常に展開**：`[AlwaysExpendOnGUI]`。
*   **「状態」との違い**：ブロックは**独立した数値**で、状態スタックではない；層数系の条件はブロックを見ない。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_DefenseSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **`[System.Serializable] + [AlwaysExpendOnGUI]`** 標記
*   **i18n クラス名 key**：`RCG_DefenseSetting` → 「ブロック」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_DefenseAlterType` | `DefenseAlterType` | enum (内部) | — | 6 モード |
| `m_Defense` | ブロック | `IntVariable` | `Defense` | `[UCL_FieldOnGUI]` + `[Conditional(m_DefenseAlterType, false, Add, Sub, Set)]` |
| `m_DefenseTarget` | `DefenseTarget` | `RCG_SelectTargetData` | — | 旧 `m_Range` を置換 |

### A.3 主なメソッド
*   **`AddAction`**（ファイル 90 行外）：DefenseAlterType に応じて `m_DefenseTarget.GetTargets(iData)` に変化を適用。
*   **`GetDescriptionFormat`**：DefenseAlterType に応じて異なる i18n key（`DefIcon_DesXxx` / `DecayDes` 等）。
*   **`Infos`**：`IsShowOnUI` 有効時にブロックアイコン情報を追加。

### A.4 他システムとの連携
*   **`EffectIcon.Armor.GetTMPSpriteName()`**：説明文中のブロックアイコン。
*   **`RCG_BattleUnit` ブロック層**：実際のブロックコンテナ；減衰判定は `m_UnitStatus` 内の buff によって阻止されることがある。
