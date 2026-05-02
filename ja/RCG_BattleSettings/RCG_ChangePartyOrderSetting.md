---
title: チームオーダーを変更
description: 味方の出場順を切り替え（前列のローテーションまたは指定キャラを最前面に）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# チームオーダーを変更

> クラス名：`RCG_ChangePartyOrderSetting`

## 用途
**味方キャラクターの並び順を変更**して別のキャラを出陣させる。よくある用途：
*   「キャラスイッチ」カード — 1 番目を最後尾へ
*   「前出し」 — 指定キャラ（脅威にさらされている子など）を 1 番目に

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **ChangePartyOrderType** | はい | 変更方式：<br>• **FirstToLast** — 1 番目のキャラを最後尾へ（後輪ローテーション）<br>• **TargetToFirst** — 選択キャラを 1 番目に |
| **対象** (`Target`) | はい | 対象セレクタ（`TargetToFirst` モードでは単体必須）。 |

## 挙動
*   `FirstToLast` は `RCG_BattleManager.Ins.SwitchCharacterUI.FirstToLast()` を直接呼ぶ。
*   `TargetToFirst` は単体ターゲットに対し「光点」VFX を再生してから UI を切替。
*   説明は ChangePartyOrderType に応じて i18n key を切替（`ChangePartyOrderDes_FirstToLast` / `ChangePartyOrderDes_TargetToFirst`）。

## 注意点
*   **TargetToFirst の多ターゲット**：セレクタが 0 個または複数個を返した場合、**期待通りに動作しない**ことがあります（コードは `aTargets.Count == 1` のときだけ VFX を再生、他はスキップする可能性）。「単体」セレクタを設定。
*   **バトルテンポ**：順序切替は通常「次の行動者が変わる」ことを意味するため、後続のトリガーポイントが正しい当該キャラに依存しているか確認を。
*   **常に展開**：この設定は `[AlwaysExpendOnGUI]` 属性、Inspector 上で**折りたたまれない**。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ChangePartyOrderSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **`[System.Serializable] + [AlwaysExpendOnGUI]`** 標記
*   **i18n クラス名 key**：`RCG_ChangePartyOrderSetting` → 「チームオーダーを変更」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_ChangePartyOrderType` | `ChangePartyOrderType` | `ChangePartyOrderType` (内部 enum) | — | `FirstToLast` / `TargetToFirst` |
| `m_Target` | 対象 | `RCG_SelectTargetData` | `Target` | 旧 `m_Range`（廃止）を置換 |

### A.3 主なメソッド
*   **`AddAction`** (async)：ChangePartyOrderType で分岐。`TargetToFirst` は `aTargets.Count == 1` のときに `RCG_VFXManager.CreateVFX(VFX_LightTarget)` を呼んで切替。
*   **`GetDescriptionShort`** → 直接 i18n key `RCG_ChangePartyOrderSetting`（=「チームオーダーを変更」）を短縮ラベルとして返す。

### A.4 他システムとの連携
*   **`RCG_BattleManager.Ins.SwitchCharacterUI`**：実際のチーム切替 UI / ロジック。
*   **`CommonVFX.VFX_LightTarget`**：前出しの視覚効果。
