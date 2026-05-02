---
title: カスタム状態 (RCG_CustomStatusData)
description: 戦闘中にユニットに付与可能な状態（buff / debuff / DoT 等）：層数変化、相殺、免疫、抵抗、攻防 buff
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# カスタム状態

> クラス名：`RCG_CustomStatusData`

## 用途

**戦闘中にユニットに付与可能な状態（buff / debuff）テンプレート**。例：「中毒：毎ターン 5 血削り」「力 +2」「負面状態免疫」「回避：層数 ≥ 攻撃力時に攻撃回避」。本クラスが状態関連属性を一箇所に集中：層数変化ルール、相殺関係、免疫 / 抵抗、攻防 buff 加成、ユニット特殊状態（スタン、詠唱…）。

`RCG_Asset<RCG_CustomStatusData>` を継承。実装：`RCGI_Status`。

## エディタ上の見た目

```
RCG_CustomStatusData: <ID>
    Name / StatusEffectType / StatusClass
    AtkBuff / DefBuff           ← 攻防加成設定
    IconSprite / Tags
    LayerIncreaseVFX / StatusPersistantVFX
    StatusOffsets / OffsetTags  ← 相殺する状態
    StatusAlterOn               ← どの trigger で層数が変わるか
    Effects                     ← 発動効果
    UnitStates                  ← 対応する特殊状態（スタン、回避…）
    StopDecayStatus             ← ターゲット状態を減衰させない
    ImmuneStatus                ← 免疫状態（取得不可）
    Resistance                  ← 抵抗（毎層 1% 確率でターゲット状態を無効化）
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Name** | はい | 状態名（多言語） |
| **StatusEffectType** | はい | `Buff` / `Debuff` / その他、表示色と相殺判定に影響 |
| **StatusClass** | — | `Normal` / `AtkBuff` / `DefBuff` / `Immune`（全負面免疫） |
| **AtkBuff / DefBuff** | いいえ | 複数の atk / def buff 設定（ダメージ倍率、防御加成） |
| **IconSprite** | はい | 状態アイコン（`RCG_IconSprite` 参照） |
| **Tags** | いいえ | 状態分類タグ（Buff / Debuff / DoT / Feature 等） |
| **LayerIncreaseVFX** | — | 層数増加時のエフェクト |
| **StatusPersistantVFX** | — | 常駐ユニットエフェクト（保持中常時表示） |
| **StatusOffsets** | いいえ | 相殺する具体的状態（例：力 ↔ 弱体） |
| **OffsetTags** | いいえ | どの**タグタイプ**と相殺するか（例：あるバフが全 Debuff を相殺） |
| **StatusAlterOn** | いいえ | どの trigger で層数変化（dictionary：`trigger → DecreaseType`） |
| **Effects** | いいえ | 各 trigger での発動効果 |
| **UnitStates** | いいえ | 対応するユニット特殊状態列挙値（`Stun` / `Guard` / `Dodge` 等） |
| **StopDecayStatus** | いいえ | ターゲット状態を減衰させない一覧 |
| **ImmuneStatus** | いいえ | 免疫状態（取得不可） |
| **Resistance** | いいえ | 毎層 1% 確率でターゲット状態を無効化 |

## 動作説明

### 層数変化 (`GetDecreaseType`)
`m_StatusAlterOn` テーブルを引き、trigger に応じて `eStatusDecreaseType` を取得：
*   `None` 不変 / `DecreaseOneLayer` -1 / `Clear` 全クリア / `DecreaseHalf` 半減
*   `AddOneLayer` +1 / `ClearImmediately` 即時クリア / `Eliminate` 消去（終了効果発動なし）

### 相殺判定 (`CheckIsOffset`)
*   `StatusClass = Immune` → Tag に `StatusDebuff` を含む全状態が相殺。
*   `OffsetTags` が相手 Tag に命中 → 相殺。
*   `StatusOffsets` が相手 ID を含む → 相殺。

### 説明生成
2段に分割（中間に空行）：
1. **相殺段**：この状態が相殺する対象を列挙。
2. **効果段**：UnitStates 説明 / Atk-DefBuff 説明 / StopDecay / Immune / Resistance / Effects 発動説明。

### IconTMPKey
`RCG_BattleSetting.IsShowOnUI = true` 時は IconSprite の TMPKey を返す（TextMeshPro アイコン用）；それ以外は LocalizedName 返却。

### 発動効果 (`OnTriggerEffect`)
`m_Effects` から該 trigger を取得し順次発動。

## 注意事項

*   **`Immune` タイプは全 Debuff を自動相殺**：手動で `OffsetTags = StatusDebuff` を列挙する必要なし、ロジック内蔵。
*   **`m_AtkBuff / m_DefBuff` はリスト**：複数積層可（条件下で異なる加成）；旧版 `m_AtkBuffData / m_DefBuffData` は置換されコメントアウト。
*   **`UnitStates` は enum に直接対応**：スタン、回避、ガード等の13種特殊状態が enum にハードコード；追加不可（プログラム改修必須）。
*   **Resistance は確率制**：1層 1%、10 層 = 10% 抵抗、効果は 100% に積層しない。
*   **StatusFeature Tag**：このタグを含む状態の名前前に「特性: 」プレフィックス追加。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_CustomStatusData.cs`
*   **継承**：`RCG_Asset<RCG_CustomStatusData>`
*   **実装**：`RCGI_Status`
*   **AssetGroup**：`EditCharacter`

### A.2 フィールドマッピング（抜粋）

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_StatusEffectType` | StatusEffectType | `StatusEffectType` enum | `Buff` / `Debuff` |
| `m_StatusClass` | StatusClass | `StatusClass` enum | `Normal` / `AtkBuff` / `DefBuff` / `Immune` |
| `m_AtkBuff` | AtkBuff | `List<RCG_AtkBuffData>` | |
| `m_DefBuff` | DefBuff | `List<RCG_DefBuffData>` | |
| `m_IconSprite` | IconSprite | `RCG_IconSpriteGenData` | |
| `m_Tags` | Tags | `List<RCG_StatusTagGenData>` | |
| `m_LayerIncreaseVFX` | LayerIncreaseVFX | `RCG_CommonVFXGenData` | デフォルト `VFX_StatusLayerID` |
| `m_StatusPersistantVFX` | StatusPersistantVFX | `RCG_CommonVFXGenData` | |
| `m_StatusOffsets` | StatusOffsets | `List<RCG_CustomStatusGenData>` | |
| `m_OffsetTags` | OffsetTags | `List<RCG_StatusTagGenData>` | |
| `m_StatusAlterOn` | StatusAlterOn | `Dictionary<RCG_EffectTriggerOn, eStatusDecreaseType>` | |
| `m_Effects` | Effects | `List<RCG_CommonEffect>` | |
| `m_UnitStates` | UnitStates | `List<UnitState>` | enum: Stun / Guard / Dodge / etc. |
| `m_StopDecayStatus` | StopDecayStatus | `List<RCG_CustomStatusGenData>` | |
| `m_ImmuneStatus` | ImmuneStatus | `List<RCG_CustomStatusGenData>` | |
| `m_Resistance` | Resistance | `List<RCG_CustomStatusGenData>` | |

### A.3 主要メソッド

*   **`GetDecreaseType(triggerOn)`** — m_StatusAlterOn テーブルを引く。
*   **`CheckIsOffset(targetStatus)`** — 三層判定：Immune class / OffsetTags / StatusOffsets。
*   **`OnTriggerEffect(triggerOn, data)`** — 効果発動。
*   **`TriggerOnUnitState(triggerOn)`** — この trigger で変化または発動するか（quick check）。
*   **`Description` (property)** — 大型 StringBuilder 構成：Offsets + UnitStates + AtkBuff + DefBuff + StopDecay + Immune + Resistance + Effects。
*   **`IconTMPKey`** — UI モードで TMPKey、それ以外で LocalizedName。
*   **`CreateLayerIncreaseVFX / CreateStatusPersistantVFX`** — async VFX 構築。

### A.4 他システムとの連携

*   **`RCG_StatusGenData`** — Asset Entry ラッパー；`Status` (property) が `new RCG_StatusGenData(ID)` を返す。
*   **`RCG_CustomStatusGenData`** — このデータを直接参照する型；`s_Default` / `s_ChargeUp` プリセットインスタンス含む。
*   **`RCG_AtkBuffData` / `RCG_DefBuffData`** — 攻防 buff サブデータ。
*   **`UnitState` (enum)** — 13種ハードコード特殊状態。
*   **`RCG_VFXManager`** — エフェクト構築。

### A.5 既知の問題

*   旧版 `m_AtkBuffData` / `m_DefBuffData` 単一フィールドはリストに置換、デシリアライズ移行ロジックはコメントアウト済。
*   `Init()` は空殻、大マップ突入時の初期化 hook 予約。
