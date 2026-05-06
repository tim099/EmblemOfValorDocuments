---
title: Cmd_ExportNotes API
description: 依 targets 参数导出 Card / Equipment / Item / Story 的 Note / Description 为 Markdown；整合自旧版 Cmd_ExportAllNotes / Cmd_ExportCardNotes / Cmd_ExportEquipmentNotes / Cmd_ExportItemNotes 四个独立指令，并扩充支持 story（2026-05-06）
source_file: CardGame/Assets/Scripts/RCG_Scripts/RCG_AgentCommands/Cmd_ExportNotes.cs
namespace: RCG.AgentCommands
last_updated: 2026-05-06
target_audience: [AI_Agent, Tools_Maintainer, Game_Designer]
---

# Cmd_ExportNotes

> 程式类别名称：`RCG.AgentCommands.Cmd_ExportNotes`
> 文件路径：[`CardGame/Assets/Scripts/RCG_Scripts/RCG_AgentCommands/Cmd_ExportNotes.cs`](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_AgentCommands/Cmd_ExportNotes.cs)

## 1. 概览

`Cmd_ExportNotes` 是 RCG 项目层的 Agent Command，依 `targets` 参数导出 Card / Equipment / Item / Story **四类**资料的 Note / Description 为 Markdown 文件（落在 `docs/Catalogs/<Type>_Notes_Export.md`）。

> [!NOTE]
> **Story target 行为**（2026-05-06 新增）：
> Story 没有 `m_Note` 字段，导出内容仿 Editor Preview 显示 — 含每个 Story 的 Tags / SubStory 结构、每个 SubStory 的 Description / Image / Options 表格（含 Transition 与 MapEvents 摘要）。输出文件：`docs/Catalogs/Story_Notes_Export.md`。

> [!IMPORTANT]
> **取代旧版 4 个独立指令**（2026-05-06 整合）：
>
> | 旧指令 | 等价新调用 |
> |---|---|
> | `ExportAllNotes` | `Type:ExportNotes`，`Args:{}` 或 `targets=all` |
> | `ExportCardNotes` | `Type:ExportNotes`，`Args:{"targets":"card"}` |
> | `ExportEquipmentNotes` | `Type:ExportNotes`，`Args:{"targets":"equipment"}` |
> | `ExportItemNotes` | `Type:ExportNotes`，`Args:{"targets":"item"}` |

典型用途：
- 编辑器内 / Agent 端**批量刷新所有 Notes 文件**（一次三类）
- **单独刷新某类**（刚编完 Card 想看 Catalog 变化）
- **组合刷新多类**（刚调过 Card + Item 平衡，不想动 Equipment）

## 2. 参数格式 (Args Schema)

| 参数 | 必填 | 默认 | 说明 |
|---|:-:|---|---|
| `targets` | ❌ | `all` | 逗号分隔的导出对象，token 可选 `card` / `equipment` / `item` / `story` / `all`（大小写不敏感）。空字符串、缺省、或包含 `all` 时会导出全部四类 |

**合法写法范例**：

| `targets` 值 | 实际导出 |
|---|---|
| (省略整个 Args) | Card + Equipment + Item + Story |
| `all` | Card + Equipment + Item + Story |
| `card` | 仅 Card |
| `story` | 仅 Story（含 SubStory 结构与 Options 表格） |
| `card,item` | Card + Item |
| `card,story` | Card + Story |
| `Card, Story` | Card + Story（大小写 / 空白皆 OK） |
| `all,card` | Card + Equipment + Item + Story（`all` 展开后 `card` 变冗余，会去重） |
| `card,card` | 仅 Card（自动去重） |
| `""` 或 `" , "` | Card + Equipment + Item + Story（空 fallback 到 all） |

**错误输入**会抛出 `ArgumentException`（Runner 写入 `LastRunError`）：

| 错误输入 | 错误消息 |
|---|---|
| `cards`（多 s） | `Unknown targets: [cards]. Valid: all, card, equipment, item, story` |
| `equipments` | `Unknown targets: [equipments]. Valid: all, card, equipment, item, story` |
| `stories`（多 ies） | `Unknown targets: [stories]. Valid: all, card, equipment, item, story` |
| `random` | `Unknown targets: [random]. Valid: all, card, equipment, item, story` |

## 3. 内部行为

每个 target 对应调用一个 `EditorPage` 的 static 方法（顺序：card → equipment → item）：

| target | 调用 | 输出文件 |
|---|---|---|
| `card` | `RCG_CardDataEditorPage.ExportNotesToMarkdown()` | `docs/Catalogs/Card_Notes_Export.md` |
| `equipment` | `RCG_EquipmentDataEditorPage.ExportNotesToMarkdown()` | `docs/Catalogs/Equipment_Notes_Export.md` |
| `item` | `RCG_ItemDataEditorPage.ExportNotesToMarkdown()` | `docs/Catalogs/Item_Notes_Export.md` |
| `story` | `RCG_StoryDataEditorPage.ExportNotesToMarkdown()` | `docs/Catalogs/Story_Notes_Export.md` |

> [!NOTE]
> **错误处理**：若某个 EditorPage 在执行中抛异常，**剩余 target 不会继续执行**（fail fast）。Runner 会记录 `LastRunResult: Failed` 与异常消息，用户修完问题后可重跑。

## 4. 在 queue.json 中调用

**全部导出（最常用）**：
```json
{
  "Id": "20260506-export-all",
  "Type": "ExportNotes",
  "Mode": "OneShot",
  "Args": {},
  "Description": "刷新 Card / Equipment / Item 三类 Notes"
}
```

**单独导出 Card**：
```json
{
  "Id": "20260506-export-card",
  "Type": "ExportNotes",
  "Mode": "OneShot",
  "Args": { "targets": "card" },
  "Description": "仅刷 Card Notes（刚改完 Card 平衡）"
}
```

**组合导出 Card + Item**：
```json
{
  "Id": "20260506-export-cardItem",
  "Type": "ExportNotes",
  "Mode": "OneShot",
  "Args": { "targets": "card,item" },
  "Description": "Card + Item 双更新（装备未动）"
}
```

## 5. Python 包装器调用

```bash
# 全部导出（默认 all）
python CardGame/Assets/UCL/UCL_Core/Tools~/AgentCommands/run_cmd.py run ExportNotes

# 仅 Card
python CardGame/Assets/UCL/UCL_Core/Tools~/AgentCommands/run_cmd.py run ExportNotes \
    --arg targets=card

# Card + Item
python CardGame/Assets/UCL/UCL_Core/Tools~/AgentCommands/run_cmd.py run ExportNotes \
    --arg targets=card,item
```

## 6. 设计理由

### 6.1 为什么从 4 个合成 1 个？

- **降低 Registry 噪音**：`UCL_AgentCommandsPage` 的 Available Commands 下拉菜单从 4 行压到 1 行
- **未来新增 target 集中改一处**：例如将来要加 `Status` Notes 导出 — 只动 `Cmd_ExportNotes` 的 `KnownTargets` 与 `RunSingleTarget` switch case，不再开新文件
- **组合弹性**：旧版要「Card + Item 不要 Equipment」必须跑两个独立指令；新版 `targets=card,item` 一次搞定
- **Typo 防御**：白名单检查 + 明确错误消息（旧版若 typo 指令名 Registry 也会找不到，但错误消息较模糊）

### 6.2 为什么选 `targets`（复数）而非 `target`（单数）？

`targets` 隐含「可多选」，与 `card,item` 这种逗号 list 写法语意一致。`target=all` 会让用户误以为是「单值枚举」而不能组合。

### 6.3 为什么 `all` 是字符串 token 而非空字符串隐式默认？

两者都支持（空字符串会 fallback 到 `all`），但**写成显式 `all` 更易读**：
```json
{ "targets": "all" }    // ✅ 一眼看出意图
{ "targets": "" }       // ⚠ 看起来像忘填
{ }                     // ✅ 可读（依赖文档知道默认为 all）
```

## 7. 关联文档

- [docs/Workflows/AgentCommands_Workflow.md](../../../docs/Workflows/AgentCommands_Workflow.md) — 项目层 Agent Commands 工作流
- [docs/Workflows/HelpURL_Workflow.md](../../../docs/Workflows/HelpURL_Workflow.md) — 本文档路径解析机制（`eov_docs:` prefix）
- [Cmd_ExportNotes.cs 源代码](../../../CardGame/Assets/Scripts/RCG_Scripts/RCG_AgentCommands/Cmd_ExportNotes.cs)
