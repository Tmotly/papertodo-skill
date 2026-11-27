---
name: papertodo-ops
description: 操作 PaperTodo 桌面应用的纸片、待办与笔记。当用户要查看或整理 PaperTodo 里的待办清单、新建/追加/修改/删除纸片与待办、显示隐藏或展开折叠纸片、调整 PaperTodo 公开设置、设置或取消待办提醒、处理工作区数据被外部编辑后的热重载与三方合并、向 Markdown 笔记导入或查看图片时使用。只用于 PaperTodo，不处理其他应用的待办、笔记或设置。
---

# PaperTodo 操作

## 基本约定

- 仅在任务需要操作 PaperTodo 时使用 `papertodo` MCP。工具的完整名称（前缀）以当前客户端实际显示为准，下文只写短名。
- 优先使用本次上下文里已有的纸片 ID / 待办 ID。目标不确定时先 `list_papers`。
- 改任何已有内容之前先 `get_paper` 取回原文，只在原文基础上改用户要求的部分，保留其余内容和结构。
- 遵循 MCP 权限：写入、删除、改设置等操作受 PaperTodo 中授予的 MCP 权限限制。被拒或连接失败时据实说明原因，不要绕过权限直接改 `data.json` 或插件数据文件。
- 工具返回成功才算完成。写入结果不明（超时、报错含歧义）时先 `get_paper` 核对现状，严禁盲目重复追加或 toggle。

## 新建与追加

- 新建待办纸片：`create_todo_paper`；新建笔记：`create_note`（内容用 Markdown）。具体参数以工具定义为准。
- 追加待办：`add_todos`；追加笔记正文：`write_note(mode="append")`。
- 标题长度有上限（可在设置中调整）。普通短标题不必先查设置，被拒时按错误里报出的上限改写。
- 待办文本长度同编辑器限制，超长时拆分或提示用户，不要截断用户内容。

## 修改

- 改待办：`update_todo`，只传需要改的字段；`text` 是整体替换而非追加。
- `clear_linked_paper=true` 与绑定 ID 不能同时传。
- 替换笔记正文：`write_note(mode="replace")`，需保留用户未提及的原有段落与结构。

## 显示、隐藏、展开、折叠

- 幂等操作用专用工具：`show_paper`（显示，保留折叠状态）、`hide_paper`（隐藏，不删内容）、`expand_paper`（展开并显示）、`collapse_paper`（折叠，不显示隐藏中的纸片）、`activate_paper`（显示并激活，不强制展开）。
- 也可用 `toggle_paper_visibility` / `toggle_paper_collapsed` 切换；但结果不明时先查询状态再决定，不能盲目重试 toggle。
- 不抢焦点时，给显示/展开类工具传 `activate=false`。
- 只在用户要求时操作窗口（显示、隐藏、展开、折叠、激活）；不要为了"让用户看到结果"自行弹出纸片。

## 设置

- 先 `list_settings`（可用 `category` 过滤）拿到公开设置 ID、类型、取值范围与本调用方的 `writable`；单项详情用 `get_setting(id)`。
- 只有任务明确要求更改时才 `set_setting`，取值必须匹配查询到的类型与范围。
- 这些是软件设置，不同于插件自己的设置；禁止改内部字段，也不要自行提升未获准的权限。

## 提醒

- `set_todo_reminder`：时间必须包含日期和时区（如 `2026-11-28T09:00:00+08:00`）。“明天上午”这类相对时间先按用户时区换算成绝对时间。

## 删除

- `delete_paper` / `delete_todo` 仅在用户明确要求删除时调用；删除前确认目标是哪一个，不要顺手清理相邻内容。

## 待办关联笔记

- 长内容需要挂到待办旁时，先 `get_setting(id="todo.paper_links")`。关闭或状态未知则跳过关联方案，不要先建一个孤立 Note。
- 关联时先 `create_note` 拿到 `paper_id`，再在 `create_todo_paper` / `add_todos` 的 `linked_paper_id` 或 `update_todo(linked_paper_id=...)` 中引用。
- 解绑用 `update_todo(clear_linked_paper=true)`，Note 本身保留。

## 笔记图片

- 查看：`get_paper` 找到正文中的 `i:图片ID` 引用，再 `read_note_image`。
- 追加到已有 Markdown 笔记：原始图片字节上限 4 MiB，只能通过 PaperTodo 可执行文件 `--import-note-image` 以二进制 stdin 写入。完整步骤与 PowerShell 子进程写法见 [references/note-images.md](references/note-images.md)。
- 不要把 Base64 写进笔记，不要用文件路径或网址作为图片来源参数；客户端拿不到图片字节时直接说明限制。

## 用户授权外部编辑工作区文件

常规内容修改一律走上面的领域工具。只有用户明确授权直接编辑工作区数据文件时，才按 [references/data-sync.md](references/data-sync.md) 的步骤执行：先查状态确认目标文件并等待就绪 → 编辑该工作区文件 → `reload_data` 接受 → 复查状态；需要合并独立修改时用 `merge_data` 并保留真实原始基线。
