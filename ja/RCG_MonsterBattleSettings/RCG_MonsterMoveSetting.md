---
title: モンスター移動
description: モンスターを戦場で移動（前後列切替）；AI 行動が使用
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# モンスター移動

> クラス名：`RCG_MonsterMoveSetting`

## 用途
**モンスターを戦場で移動させる** — 通常は前後列切替。モンスター AI 行動で使用。よくある用途：
*   敵が能動的に後列に退いて避戦
*   特殊ユニットの位置移動スキル

> [!IMPORTANT]
> この設定は**モンスターデータを編集するときのみ**ドロップダウンに現れます。一般カードでは選べません。

## 主なフィールド
（フィールドなし — 純粋な挙動設定）

## 挙動
*   `CreateAction.MoveAction(iData.User)` を呼んでユニット移動 Action をトリガー。
*   実際の移動ロジックは `MoveAction` の内部ルールが決定（一般的には前後列切替）。
*   説明は i18n key `UnitAction_MoveActionDes`、短縮説明は移動 sprite を表示。

## 注意点
*   **「移動」設定との違い**：「移動」(`RCG_MoveSetting`) は**汎用**移動設定で、方向とターゲットを指定可能；本設定は**モンスター AI 専用**で、挙動固定、フィールドで調整不可。
*   **フィールドなし = 挙動固定**：移動挙動は `CreateAction.MoveAction` が決定；カスタムは「**移動**」を使用。
*   **User なしのときスキップ**：`iData.User == null` のとき何もしない。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_MonsterBattleSettings/RCG_MonsterMoveSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key なし**：エディタ表示は stripped name `MonsterMove`
*   **選択範囲限定**：`RCG_BattleSetting.s_MonsterDataTypes` に登録（モンスターデータ編集時のみメニューに現れる）。

### A.2 フィールド対照
（自身のフィールドなし）

### A.3 主なメソッド
*   **`AddAction`**：`iData.User != null` のとき `iData.AddAction(CreateAction.MoveAction(iData.User), iAddActionMode)`。
*   **`GetDescription`** → i18n key `UnitAction_MoveActionDes`。
*   **`GetDescriptionShort`** → 移動 sprite (`EffectIcon.Move`)。

### A.4 他システムとの連携
*   **`CreateAction.MoveAction(unit)`**：実際の移動 Action ビルダー（汎用「移動」と共有？モンスター AI 専用？）。
