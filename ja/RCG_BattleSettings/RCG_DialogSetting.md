---
title: ダイアログ
description: キャラクターのダイアログバブルを表示（フォントサイズ・揺れ・効果音含む）；純粋な視覚演出
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# ダイアログ

> クラス名：`RCG_DialogSetting`

## 用途
**キャラクターのダイアログバブルを表示**。純粋な視覚演出で、ゲーム挙動への影響なし。よくある用途：
*   敵の攻撃前のセリフ
*   特殊スキルの劇情演出
*   Boss 戦中のダイアログ橋渡し

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Dialog** | はい | ダイアログ内容（`RCG_LocalizeData`、多言語対応）。 |
| **Duration** | はい | ダイアログ表示時間（秒）、デフォルト 0.8 秒。 |
| **FontSize** | — | フォントサイズ、デフォルト 40。 |
| **Shake** | — | 揺れ表示するか（強調用）。 |
| **PlayAudio** | — | 効果音を再生するか。 |
| **Audio** | PlayAudio チェック時必須 | 再生する効果音（`RCG_SEGenData`）；`Conditional` で表示制御。 |
| **HideDescription** | — | カード説明文上で隠すか — チェック = ダイアログ内容が**カード説明には出ない**（純粋な背景演出）。 |

## 挙動
*   発動時に `VFX_Dialog` を生成し、`vfx.SetDialog(this, iData.User)` で発話者の頭上に表示。
*   説明にはダイアログ内容そのものを表示（`HideDescription` がオンを除く）。
*   **短縮説明は常に空**（敵意図バーには出ない）。

## 注意点
*   **HideDescription は「純粋な雰囲気ダイアログ」向け**：例：Boss 戦の背景セリフ — カード説明がダイアログ文字で埋まる事態を回避。
*   **Duration が短すぎる**：0.5 秒未満ではプレイヤーが読めない；ダイアログが長くなるほど Duration を伸ばすべき。
*   **音効未指定だが PlayAudio オン**：空の `RCG_SEGenData` を再生しようとして console に warning を出す可能性。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_DialogSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_DialogSetting` → 「ダイアログ」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_Dialog` | `Dialog` | `RCG_LocalizeData` | — | |
| `m_Duration` | `Duration` | `float` | — | デフォルト 0.8f |
| `m_FontSize` | `FontSize` | `int` | — | デフォルト 40 |
| `m_Shake` | `Shake` | `bool` | — | |
| `m_PlayAudio` | `PlayAudio` | `bool` | — | |
| `m_Audio` | `Audio` | `RCG_SEGenData` | — | `[Conditional(nameof(m_PlayAudio), false, true)]` |
| `m_HideDescription` | `HideDescription` | `bool` | — | |

### A.3 主なメソッド
*   **`AddAction`** (async)：`RCG_VFXManager.CreateVFX(VFX_Dialog) as RCG_VFX_Dialog` → `vfx.SetDialog(this, iData.User)`。
*   **`GetDescriptionFormat`**：`m_HideDescription ? string.Empty : m_Dialog.Name`。
*   **`GetDescriptionShort`** → 常に空文字。

### A.4 他システムとの連携
*   **`CommonVFX.VFX_Dialog` / `RCG_VFX_Dialog.SetDialog`**：ダイアログ UI エントリ。
