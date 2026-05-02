---
title: ストーリーデータ (RCG_StoryData)
description: 1つの「ストーリー」の完全な定義：開始イベント、サブストーリー辞書、条件、ランダム開始、Tag フィルタ
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# ストーリーデータ

> クラス名：`RCG_StoryData`

## 用途

**1つの「ストーリー」の完全な脚本データ**。`RCG_MapEventData` よりも複雑 — 複数のサブストーリー (`SubStory`) が構成する会話ツリーを含み、プレイヤーが選択すると異なるサブストーリーに飛ぶ。例：「商人」は1つの Story：開始画面 → 選択 →「購入」/「離脱」/「商人撃破」 → 各々のサブストーリー → 結末。

`RCG_Asset<RCG_StoryData>` を継承。実装：`UCLI_ShortName`。

## エディタ上の見た目

```
RCG_StoryData: <ID>
    StoryData (m_StoryData)         ← ストーリーのメタデータ
        Tags / CanTriggerRepeatedly / Conditions
        IsRandomStart / RandomStartStories
    Story (StartEvent)              ← 開始 SubStory（key = "Start"）
    SubStory (m_EventDic)           ← 全サブストーリー dictionary：{ id → RCG_OptionEventData }
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Tags** | いいえ | ストーリータグ（StoryDropPool フィルタ用） |
| **CanTriggerRepeatedly** | — | 繰返し発動可能か |
| **Conditions** | いいえ | 発動条件（OR；空 = 無条件） |
| **IsRandomStart** | — | 複数の開始点からランダムに1つ選ぶか |
| **RandomStartStories** | IsRandomStart=true 時 | ランダム開始点一覧 + 重み |
| **Story** | はい | 主開始イベント（key 固定で `"Start"`） |
| **SubStory** | いいえ | 全サブストーリー辞書（key = state name、value = `RCG_OptionEventData`） |

各 `RCG_OptionEventData`（サブストーリー）の内訳は会話 / オプション / 結果効果、次の SubStory の ID を指定可能でツリー構造形成。

## 動作説明

### 開始点選択 (`GetStartSubStory(iSubStory)`)
*   `iSubStory` 指定済 → 使用。
*   それ以外で `m_IsRandomStart` かつ `m_RandomStartStories` 非空 → 重み付きランダム抽選。
*   それ以外で fallback `StartStateName = "Start"`。

### 発動 (`StartStory(token, iSubStory, iIsRoot)`)
1. `GetStartSubStory(iSubStory)` で開始点決定。
2. `RCG_DataService.Ins.StartStory(ID, iSubStory, iIsRoot)` 書込（発動済ストーリー記録）。
3. `s_CurStoryData = this` 設定（他システムが現ストーリーを問い合わせ可）。
4. 対応 SubStory の `RCG_OptionEventData.StartEvent(...)` を取得し演出開始。
5. 終了後 `s_CurStoryData = null` クリア。

### 条件判定 (`StoryData.CheckCondition`)
*   `m_Conditions` 空 → 常に true 返却。
*   非空 → `CheckConditions_OR`（OR 関係）。

## 注意事項

*   **`StartStateName = "Start"` は magic string**：開始イベントの key は固定でこれ；他名前は開始とみなされない。
*   **繰返し発動フラグ**：`CanTriggerRepeatedly` のデフォルトは true（MapEventData と逆）；ストーリーは多くの場合繰返し可。
*   **`s_CurStoryData` はグローバル static**：ストーリー実行期間に外部から現ストーリー検索可；ただし**複数ストーリーを並行不可**（互いに上書き）。
*   **`m_EventDic` 自動空項目補完**：`GetSubStory(state)` で key 不在時に自動的に空 `RCG_OptionEventData` を add、呼出側が NRE しない、ただし空白ストーリーが見える。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_StoryData.cs`
*   **継承**：`RCG_Asset<RCG_StoryData>`
*   **実装**：`UCLI_ShortName`
*   **AssetGroup**：`EditQuestSetting`
*   **定数**：`StartStateName = "Start"`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_StoryData` | StoryData | `StoryData`（入れ子） | メタデータ：tags / conditions / random start |
| `m_EventDic` | SubStory | `Dictionary<string, RCG_OptionEventData>` | サブストーリーツリー |

`StoryData` 入れ子に `m_Tags` / `m_CanTriggerRepeatedly` / `m_Conditions` / `m_IsRandomStart` / `m_RandomStartStories`。

### A.3 主要メソッド

*   **`StartStory(token, iSubStory, iIsRoot)`** — 主発動；async；開始点選択 + DataService 記録 + s_CurStoryData 切替を含む。
*   **`StartEvent` (property)** — `GetSubStory(StartStateName)`、開始イベントショートカット。
*   **`GetSubStory(state)`** — サブストーリー取得；不在時に空項目を自動 add。
*   **`StoryData.CheckCondition(data)`** — 発動条件 OR チェック。
*   **`StoryData.GetStartSubStory(iSubStory)`** — 開始点選択ロジック。
*   **`DefaultStory` (static)** — デフォルトストーリー fallback。

### A.4 他システムとの連携

*   **`RCG_OptionEventData`** — 各 SubStory の内容（会話、オプション、結果）。
*   **`RCG_StoryDropPool`** — ランダム池ソース。
*   **`RCG_DataService.Ins.StartStory`** — runtime 発動済ストーリー記録。
*   **`RCG_StoryGenData`** — Asset Entry ラッパー。
*   **`RCG_GameManager.Random`** — ランダム開始点抽選。

### A.5 既知の問題

*   `s_CurStoryData` がストーリー並行をサポートしない；極端な状況で中断処理を考慮する必要。
*   `// public class StoryState` と `m_OptionEventData` がコメントアウト、旧版単一イベント構造から dictionary への移行履歴を示す。
*   `DeserializeFromJson` 内の旧版移行ロジック（`m_EventDic[StartStateName] = m_OptionEventData`）はコメントアウト済。
