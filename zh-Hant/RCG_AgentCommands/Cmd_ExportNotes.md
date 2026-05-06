---
title: Cmd_ExportNotes API
description: 依 targets 參數匯出 Card / Equipment / Item / Story 的 Note / Description 為 Markdown；整合自舊版 Cmd_ExportAllNotes / Cmd_ExportCardNotes / Cmd_ExportEquipmentNotes / Cmd_ExportItemNotes 四個獨立指令，並擴充支援 story（2026-05-06）
source_file: CardGame/Assets/Scripts/RCG_Scripts/RCG_AgentCommands/Cmd_ExportNotes.cs
namespace: RCG.AgentCommands
last_updated: 2026-05-06
target_audience: [AI_Agent, Tools_Maintainer, Game_Designer]
---

# Cmd_ExportNotes

> 程式類別名稱：`RCG.AgentCommands.Cmd_ExportNotes`
> 檔案路徑：[`CardGame/Assets/Scripts/RCG_Scripts/RCG_AgentCommands/Cmd_ExportNotes.cs`](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_AgentCommands/Cmd_ExportNotes.cs)

## 1. 概覽

`Cmd_ExportNotes` 是 RCG 專案層的 Agent Command，依 `targets` 參數匯出 Card / Equipment / Item / Story **四類**資料的 Note / Description 為 Markdown 檔（落在 `docs/Catalogs/<Type>_Notes_Export.md`）。

> [!NOTE]
> **Story target 行為**（2026-05-06 新增）：
> Story 沒有 `m_Note` 欄位，匯出內容仿 Editor Preview 顯示 — 含每個 Story 的 Tags / SubStory 結構、每個 SubStory 的 Description / Image / Options 表格（含 Transition 與 MapEvents 摘要）。輸出檔：`docs/Catalogs/Story_Notes_Export.md`。

> [!IMPORTANT]
> **取代舊版 4 個獨立指令**（2026-05-06 整合）：
>
> | 舊指令 | 等價新呼叫 |
> |---|---|
> | `ExportAllNotes` | `Type:ExportNotes`，`Args:{}` 或 `targets=all` |
> | `ExportCardNotes` | `Type:ExportNotes`，`Args:{"targets":"card"}` |
> | `ExportEquipmentNotes` | `Type:ExportNotes`，`Args:{"targets":"equipment"}` |
> | `ExportItemNotes` | `Type:ExportNotes`，`Args:{"targets":"item"}` |

典型用途：
- 編輯器內 / Agent 端**批次刷新所有 Notes 文件**（一次三類）
- **單獨刷新某類**（剛編完 Card 想看 Catalog 變化）
- **組合刷新多類**（剛調過 Card + Item 平衡，不想動 Equipment）

## 2. 參數格式 (Args Schema)

| 參數 | 必填 | 預設 | 說明 |
|---|:-:|---|---|
| `targets` | ❌ | `all` | 逗號分隔的匯出對象，token 可選 `card` / `equipment` / `item` / `story` / `all`（大小寫不敏感）。空字串、缺省、或包含 `all` 時會匯出全部四類 |

**合法寫法範例**：

| `targets` 值 | 實際匯出 |
|---|---|
| (省略整個 Args) | Card + Equipment + Item + Story |
| `all` | Card + Equipment + Item + Story |
| `card` | 只 Card |
| `story` | 只 Story（含 SubStory 結構與 Options 表格） |
| `card,item` | Card + Item |
| `card,story` | Card + Story |
| `Card, Story` | Card + Story（大小寫 / 空白皆 OK） |
| `all,card` | Card + Equipment + Item + Story（`all` 展開後 `card` 變冗餘，會去重） |
| `card,card` | 只 Card（自動去重） |
| `""` 或 `" , "` | Card + Equipment + Item + Story（空 fallback 到 all） |

**錯誤輸入**會丟 `ArgumentException`（Runner 寫入 `LastRunError`）：

| 錯誤輸入 | 錯誤訊息 |
|---|---|
| `cards`（多 s） | `Unknown targets: [cards]. Valid: all, card, equipment, item, story` |
| `equipments` | `Unknown targets: [equipments]. Valid: all, card, equipment, item, story` |
| `stories`（多 ies） | `Unknown targets: [stories]. Valid: all, card, equipment, item, story` |
| `random` | `Unknown targets: [random]. Valid: all, card, equipment, item, story` |

## 3. 內部行為

每個 target 對應呼叫一個 `EditorPage` 的 static 方法（順序：card → equipment → item）：

| target | 呼叫 | 輸出檔 |
|---|---|---|
| `card` | `RCG_CardDataEditorPage.ExportNotesToMarkdown()` | `docs/Catalogs/Card_Notes_Export.md` |
| `equipment` | `RCG_EquipmentDataEditorPage.ExportNotesToMarkdown()` | `docs/Catalogs/Equipment_Notes_Export.md` |
| `item` | `RCG_ItemDataEditorPage.ExportNotesToMarkdown()` | `docs/Catalogs/Item_Notes_Export.md` |
| `story` | `RCG_StoryDataEditorPage.ExportNotesToMarkdown()` | `docs/Catalogs/Story_Notes_Export.md` |

> [!NOTE]
> **錯誤處理**：若某個 EditorPage 在執行中拋例外，**剩餘 target 不會繼續執行**（fail fast）。Runner 會記錄 `LastRunResult: Failed` 與例外訊息，使用者修完問題後可重跑。

## 4. 在 queue.json 中呼叫

**全部匯出（最常用）**：
```json
{
  "Id": "20260506-export-all",
  "Type": "ExportNotes",
  "Mode": "OneShot",
  "Args": {},
  "Description": "刷新 Card / Equipment / Item 三類 Notes"
}
```

**單獨匯出 Card**：
```json
{
  "Id": "20260506-export-card",
  "Type": "ExportNotes",
  "Mode": "OneShot",
  "Args": { "targets": "card" },
  "Description": "只刷 Card Notes（剛改完 Card 平衡）"
}
```

**組合匯出 Card + Item**：
```json
{
  "Id": "20260506-export-cardItem",
  "Type": "ExportNotes",
  "Mode": "OneShot",
  "Args": { "targets": "card,item" },
  "Description": "Card + Item 雙更新（裝備未動）"
}
```

## 5. Python 包裝器呼叫

```bash
# 全部匯出（預設 all）
python CardGame/Assets/UCL/UCL_Core/Tools~/AgentCommands/run_cmd.py run ExportNotes

# 只 Card
python CardGame/Assets/UCL/UCL_Core/Tools~/AgentCommands/run_cmd.py run ExportNotes \
    --arg targets=card

# Card + Item
python CardGame/Assets/UCL/UCL_Core/Tools~/AgentCommands/run_cmd.py run ExportNotes \
    --arg targets=card,item
```

## 6. 設計理由

### 6.1 為什麼從 4 個合成 1 個？

- **降低 Registry 噪音**：`UCL_AgentCommandsPage` 的 Available Commands 下拉選單從 4 行壓到 1 行
- **未來新增 target 集中改一處**：例如將來要加 `Status` Notes 匯出 — 只動 `Cmd_ExportNotes` 的 `KnownTargets` 與 `RunSingleTarget` switch case，不再開新檔
- **組合彈性**：舊版要「Card + Item 不要 Equipment」必須跑兩個獨立指令；新版 `targets=card,item` 一次搞定
- **Typo 防禦**：白名單檢查 + 明確錯誤訊息（舊版若 typo 指令名 Registry 也會找不到，但錯誤訊息較模糊）

### 6.2 為什麼選 `targets`（複數）而非 `target`（單數）？

`targets` 隱含「可多選」，與 `card,item` 這種逗號 list 寫法語意一致。`target=all` 會讓使用者誤以為是「單值列舉」而不能組合。

### 6.3 為什麼 `all` 是字串 token 而非空字串隱式預設？

兩者都支援（空字串會 fallback 到 `all`），但**寫成顯式 `all` 更易讀**：
```json
{ "targets": "all" }    // ✅ 一眼看出意圖
{ "targets": "" }       // ⚠ 看起來像忘填
{ }                     // ✅ 可讀（依賴文件知道預設為 all）
```

## 7. 關聯文件

- [docs/Workflows/AgentCommands_Workflow.md](../../../docs/Workflows/AgentCommands_Workflow.md) — 專案層 Agent Commands 工作流
- [docs/Workflows/HelpURL_Workflow.md](../../../docs/Workflows/HelpURL_Workflow.md) — 本文件路徑解析機制（`eov_docs:` prefix）
- [Cmd_ExportNotes.cs 原始碼](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_AgentCommands/Cmd_ExportNotes.cs)
