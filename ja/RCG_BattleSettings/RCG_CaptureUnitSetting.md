---
title: キャプチャー
description: ターゲットユニットを捕獲（戦場から除去）し、召喚アイテムとしてインベントリに追加
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# キャプチャー

> クラス名：`RCG_CaptureUnitSetting`

## 用途
**指定したターゲットユニットを捕獲** — 戦場から除去し、**対応する「召喚アイテム」を生成**してプレイヤーのインベントリに追加。最も典型的な用途は「モンスターボール式」捕獲カード：戦闘後にそのモンスターを召喚して協力させられる。

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **対象** (`Target`) | はい | 捕獲対象のユニットセレクタ（通常は「敵単体」）。 |
| **CaptureUnitItem** | はい | 捕獲後に生成するアイテムテンプレート；デフォルトは `Vivarium_Captured`（獣籠）。 |

## 挙動
*   実行時：
    1. ターゲットに捕獲特効（`VFX_Summon`）+ 縮小アニメ 0.6 秒を再生。
    2. ターゲットが `Flee()` で除去（戦場から消える、**討伐扱いではない**）。
    3. `CaptureUnitItem` をテンプレートとして clone した新アイテム。名前は `{アイテム名}[{捕獲モンスター名}]` 形式に変更。
    4. アイテム ID は `{原 ID}_{捕獲モンスター ID}` に変更され、同名衝突を回避。
    5. アイテムの「使用効果」に「召喚（捕獲したユニット）」ロジックが自動書込。
    6. アイテムをインベントリに追加し、「アイテム獲得」パネルを表示。
*   説明形式：「**{対象} を捕獲**」（i18n key `CaptureUnitDes`）。

## 注意点
*   **捕獲は討伐ではない**：`Flee()` で戦場から除去するため、「討伐後にトリガー」「討伐数」などの条件**には記録されない**。設計時に注意。
*   **多ターゲットセレクタ**：技術的にセレクタが複数を返した場合、**最初のもののみ捕獲**（コード `aTargets[0]`）、他は無視。単体ターゲットを選んでください。
*   **CaptureUnitItem は空にできない**：`Vivarium_Captured` がデフォルト；存在しない ID に変えるとアイテム生成時に例外発生。
*   **アイテム名衝突**：同種モンスターを 2 度捕獲すると同 ID アイテムが生成 → 後者が前者を上書き — 既知の挙動。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CaptureUnitSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_CaptureUnitSetting` → 「キャプチャー」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_Target` | 対象 | `RCG_SelectTargetData` | `Target` | 捕獲対象セレクタ |
| `m_CaptureUnitItem` | `CaptureUnitItem` | `RCG_ItemGenData` | — | デフォルト `RCG_ItemGenData("Vivarium_Captured")` |

### A.3 主なメソッド
*   **`AddAction`** (async)：
    1. `m_Target.GetTargets(iData)[0]` → `aTarget`。
    2. `RCG_VFXManager.CreateVFX(CommonVFX.VFX_Summon)` を生成し位置設定。
    3. `aTarget.UnitAnimController.ScaleUnitLocal(...)` 縮小演出。
    4. `aTarget.Flee()` で戦場から除去。
    5. `m_CaptureUnitItem.GetData()` を `RCG_ItemData` に clone：名前変更（`LocalizeDic`）、ID 変更（unitID サフィックス追加）。
    6. `RCG_CommonEffect`（`OnPlay`）+ `RCG_SummonSetting`（`SummonType.Default`, `IsMonster=false`, `Unit=被捕獲モンスター`）を内蔵。
    7. `RCG_DataService.Ins.AddRuntimeData(aItemData, DataType.InGameRuntime)` で runtime データとして登録。
    8. `RCG_Item(aItemData, RCG_Item.ItemType.Runtime)` を作成し `AddItem()` でインベントリに。
    9. `RCG_AquireItemPanel` で獲得結果を表示。
*   **`GetDescriptionFormat`** → i18n key `CaptureUnitDes`、`{Target}` 1 個のみのパラメータ。

### A.4 他システムとの連携
*   **`RCG_VFXManager` / `CommonVFX.VFX_Summon`**：捕獲特効。
*   **`RCG_BattleUnit.Flee`**：「逃走」方式でユニットを除去（討伐扱いではない）。
*   **`RCG_DataService` / `DataType.InGameRuntime`**：runtime アイテムの登録先。
*   **`RCG_SummonSetting`**：アイテムの使用効果に組み込まれる。
*   **`RCG_AquireItemPanel`**：UI 表示。
