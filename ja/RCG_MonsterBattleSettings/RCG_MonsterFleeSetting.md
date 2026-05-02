---
title: モンスター逃走
description: モンスター（敵）を戦場から逃走させ、横移動アニメ後に戦場から除去
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# モンスター逃走

> クラス名：`RCG_MonsterFleeSetting`

## 用途
**モンスターを戦場から逃走させる** — ユニットが場外に向かって移動するアニメを再生し、戦場から除去（**討伐扱いではない、討伐系効果はトリガーしない**）。よくある用途：
*   特定敵の「逃走」行動（HP が閾値以下になったとき逃げる）
*   劇情系の「敵が退場」演出

> [!IMPORTANT]
> この設定は**モンスターデータ（`RCG_UnitData` / `RCG_MonsterActionData`）を編集するときのみ**ドロップダウンに現れます。一般カードでは選べません。

## 主なフィールド
（フィールドなし — 純粋な挙動設定）

## 挙動
*   横移動アニメを再生：敵側は右に 2000 単位、味方側は左に 2000 単位（0.8 秒）。
*   アニメ終了後 `aUser.Flee()` で戦場から除去。
*   説明形式は i18n key `FleeActionDescription`（完全説明）/ `FleeActionDescriptionShort`（短縮）。

## 注意点
*   **討伐扱いではない**：「即死」「死亡」とは異なり、**討伐数記録、討伐系条件はトリガーしない**。
*   **フィールドなし = 挙動固定**：移動方向と時間がハードコード；カスタムは他の設定の組み合わせで。
*   **モンスター専用設計**：プレイヤーカードに置いても理論的に動くが、ロジックは「敵逃走」用に設計されている。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_MonsterBattleSettings/RCG_MonsterFleeSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key なし**：エディタ表示は stripped name `MonsterFlee`
*   **選択範囲限定**：`RCG_BattleSetting.s_MonsterDataTypes` に登録（モンスターデータ編集時のみメニューに現れる）。

### A.2 フィールド対照
（自身のフィールドなし）

### A.3 主なメソッド
*   **`AddAction`** (async)：`iData.User.UnitFaction` で `aX = -2000` または `+2000` を決定、`UnitAnimController.MoveUnitLocal(token, new Vector3(aX, 0), 0.8f, false)` を呼び、後 `aUser.Flee()` で戦場から除去。
*   **`GetDescription / GetDescriptionShort`**：i18n key `FleeActionDescription` / `FleeActionDescriptionShort`。

### A.4 他システムとの連携
*   **`RCG_BattleUnit.Flee`**：ユニット除去エントリ（「キャプチャー」と共有）。
*   **`UnitAnimController.MoveUnitLocal`**：横移動アニメ。
