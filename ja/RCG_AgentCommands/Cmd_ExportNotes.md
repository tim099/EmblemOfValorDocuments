---
title: Cmd_ExportNotes API
description: targets パラメータに応じて Card / Equipment / Item の Note / Description を Markdown として書き出す。旧版の Cmd_ExportAllNotes / Cmd_ExportCardNotes / Cmd_ExportEquipmentNotes / Cmd_ExportItemNotes 4 指令を統合
source_file: CardGame/Assets/Scripts/RCG_Scripts/RCG_AgentCommands/Cmd_ExportNotes.cs
namespace: RCG.AgentCommands
last_updated: 2026-05-06
target_audience: [AI_Agent, Tools_Maintainer, Game_Designer]
---

# Cmd_ExportNotes

> プログラムクラス名：`RCG.AgentCommands.Cmd_ExportNotes`
> ファイルパス：[`CardGame/Assets/Scripts/RCG_Scripts/RCG_AgentCommands/Cmd_ExportNotes.cs`](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_AgentCommands/Cmd_ExportNotes.cs)

## 1. 概要

`Cmd_ExportNotes` は RCG プロジェクト層の Agent Command で、`targets` パラメータに応じて Card / Equipment / Item の Note / Description を Markdown ファイル（`docs/Catalogs/<Type>_Notes_Export.md`）として書き出します。

> [!IMPORTANT]
> **旧版 4 指令を置き換え**（2026-05-06 統合）：
>
> | 旧指令 | 新しい等価呼び出し |
> |---|---|
> | `ExportAllNotes` | `Type:ExportNotes`、`Args:{}` または `targets=all` |
> | `ExportCardNotes` | `Type:ExportNotes`、`Args:{"targets":"card"}` |
> | `ExportEquipmentNotes` | `Type:ExportNotes`、`Args:{"targets":"equipment"}` |
> | `ExportItemNotes` | `Type:ExportNotes`、`Args:{"targets":"item"}` |

典型的な用途：
- エディタ内 / Agent 側で**全 Notes 文書を一括更新**（3 種類同時）
- **特定の種類のみ更新**（Card を編集した後 Catalog の差分を確認）
- **複数の組み合わせ更新**（Card + Item のバランス調整後、Equipment は触らない）

## 2. パラメータ仕様 (Args Schema)

| パラメータ | 必須 | デフォルト | 説明 |
|---|:-:|---|---|
| `targets` | ❌ | `all` | カンマ区切りの書き出し対象。トークンは `card` / `equipment` / `item` / `all` から選択（大文字小文字不問）。空文字列、未指定、`all` を含む場合は 3 種類すべてを書き出し |

**有効な書き方の例**：

| `targets` 値 | 実際の書き出し |
|---|---|
| (Args 全体を省略) | Card + Equipment + Item |
| `all` | Card + Equipment + Item |
| `card` | Card のみ |
| `card,item` | Card + Item |
| `Card, Item` | Card + Item（大文字小文字 / 空白も許容） |
| `all,card` | Card + Equipment + Item（`all` を展開後 `card` は冗長になり重複排除） |
| `card,card` | Card のみ（自動重複排除） |
| `""` または `" , "` | Card + Equipment + Item（空は all にフォールバック） |

**不正な入力**は `ArgumentException` を投げます（Runner は `LastRunError` に記録）：

| 不正な入力 | エラーメッセージ |
|---|---|
| `cards`（s が余分） | `Unknown targets: [cards]. Valid: all, card, equipment, item` |
| `equipments` | `Unknown targets: [equipments]. Valid: all, card, equipment, item` |
| `random` | `Unknown targets: [random]. Valid: all, card, equipment, item` |

## 3. 内部動作

各 target は対応する `EditorPage` の static メソッドを呼び出します（順序：card → equipment → item）：

| target | 呼び出し | 出力ファイル |
|---|---|---|
| `card` | `RCG_CardDataEditorPage.ExportNotesToMarkdown()` | `docs/Catalogs/Card_Notes_Export.md` |
| `equipment` | `RCG_EquipmentDataEditorPage.ExportNotesToMarkdown()` | `docs/Catalogs/Equipment_Notes_Export.md` |
| `item` | `RCG_ItemDataEditorPage.ExportNotesToMarkdown()` | `docs/Catalogs/Item_Notes_Export.md` |

> [!NOTE]
> **エラーハンドリング**：いずれかの EditorPage が実行中に例外を投げた場合、**残りの target は実行されません**（fail fast）。Runner は `LastRunResult: Failed` と例外メッセージを記録し、ユーザーが問題を修正してから再実行できます。

## 4. queue.json での呼び出し

**すべて書き出し（最も一般的）**：
```json
{
  "Id": "20260506-export-all",
  "Type": "ExportNotes",
  "Mode": "OneShot",
  "Args": {},
  "Description": "Card / Equipment / Item の 3 種類の Notes を更新"
}
```

**Card のみ書き出し**：
```json
{
  "Id": "20260506-export-card",
  "Type": "ExportNotes",
  "Mode": "OneShot",
  "Args": { "targets": "card" },
  "Description": "Card Notes のみ更新（Card のバランス調整後）"
}
```

**Card + Item の組み合わせ書き出し**：
```json
{
  "Id": "20260506-export-cardItem",
  "Type": "ExportNotes",
  "Mode": "OneShot",
  "Args": { "targets": "card,item" },
  "Description": "Card + Item 同時更新（Equipment は変更なし）"
}
```

## 5. Python ラッパーでの呼び出し

```bash
# すべて書き出し（既定 all）
python CardGame/Assets/UCL/UCL_Core/Tools~/AgentCommands/run_cmd.py run ExportNotes

# Card のみ
python CardGame/Assets/UCL/UCL_Core/Tools~/AgentCommands/run_cmd.py run ExportNotes \
    --arg targets=card

# Card + Item
python CardGame/Assets/UCL/UCL_Core/Tools~/AgentCommands/run_cmd.py run ExportNotes \
    --arg targets=card,item
```

## 6. 設計理由

### 6.1 なぜ 4 つを 1 つに統合？

- **Registry のノイズ削減**：`UCL_AgentCommandsPage` の Available Commands ドロップダウンが 4 行から 1 行に
- **将来の target 追加が一箇所に集中**：例えば `Status` Notes の書き出しを追加する際、`Cmd_ExportNotes` の `KnownTargets` と `RunSingleTarget` switch case のみ修正、新しいファイルを作る必要なし
- **組み合わせの柔軟性**：旧版で「Card + Item で Equipment は除外」する場合は 2 つの独立指令を実行する必要があったが、新版では `targets=card,item` 一発で完了
- **Typo 防御**：ホワイトリストチェック + 明確なエラーメッセージ（旧版では指令名の typo は Registry が見つからないが、エラーメッセージが曖昧だった）

### 6.2 なぜ `targets`（複数形）で `target`（単数形）でない？

`targets` は「複数選択可能」を暗黙に示し、`card,item` のようなカンマリスト記法と意味的に整合します。`target=all` だとユーザーが「単一値の列挙」と誤解し組み合わせができないと思い込む恐れがあります。

### 6.3 なぜ `all` は文字列トークンで暗黙のデフォルトでない？

両方サポートしますが（空文字列は `all` にフォールバック）、**明示的に `all` と書く方が読みやすい**：
```json
{ "targets": "all" }    // ✅ 意図が一目瞭然
{ "targets": "" }       // ⚠ 入力忘れに見える
{ }                     // ✅ 可読（既定が all であるとドキュメントから知っている前提）
```

## 7. 関連ドキュメント

- [docs/Workflows/AgentCommands_Workflow.md](../../../docs/Workflows/AgentCommands_Workflow.md) — プロジェクト層 Agent Commands ワークフロー
- [docs/Workflows/HelpURL_Workflow.md](../../../docs/Workflows/HelpURL_Workflow.md) — 本ドキュメントのパス解決機構（`eov_docs:` prefix）
- [Cmd_ExportNotes.cs ソースコード](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_AgentCommands/Cmd_ExportNotes.cs)
