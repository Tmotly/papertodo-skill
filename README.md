# papertodo-ops

让 AI 助手通过 MCP 操作 [PaperTodo](https://github.com/snownico0722/PaperTodo) 桌面便签的 Skill：查看、新建、追加、修改纸片 / 待办 / 笔记，显示或折叠纸片，设置提醒，向笔记导入图片等。

> **非官方项目。** 本 Skill 依据 PaperTodo **v4.0.0-beta2** 设置中“一键复制 AI Skill 提示词”的内容整理而成，与 PaperTodo 作者无关。PaperTodo 的软件、MCP 接口和原始提示词的版权归原作者所有。
> 官方仓库：<https://github.com/snownico0722/PaperTodo> · 官网：<https://snownico0722.github.io/PaperTodo/>

## 适用版本

- PaperTodo **v4.0.0-beta2**（Windows）
- PaperTodo 的本地 MCP 目前是**实验室 / Preview 功能**，之后的版本可能改工具名、参数或规则。如果和新版本对不上，**以 PaperTodo 设置里最新复制的官方提示词为准**。

## 目录结构

```
papertodo-ops/
├── SKILL.md                  # 主说明：基本约定、增删改、窗口、设置、提醒、关联笔记
├── README.md                 # 本文件
└── references/
    ├── note-images.md        # 笔记图片的查看与导入（含 PowerShell 5.1 / 7 通用写法）
    └── data-sync.md          # 用户授权外部编辑工作区文件时的热重载与三方合并
```

## 使用前提

1. 已下载并运行 PaperTodo v4.0.0-beta2。
2. 已开启 PaperTodo 的本地 MCP（以 `--mcp` 启动）。在 PaperTodo **设置**里可以一键复制 MCP 配置，把它添加到你的 AI 客户端。
3. 在 PaperTodo 中给 MCP 授予需要的权限。权限不够时，Skill 会让 AI 说明原因，不会绕过权限去直接改数据文件。

> 本 Skill 只是告诉 AI “怎么正确地用 PaperTodo 的 MCP 工具”，**不包含 MCP 本身**。没连上 MCP，Skill 不会起作用。

## 安装

先下载本仓库，拿到整个 `papertodo-ops` 文件夹（文件夹名保持不变，里面必须有 `SKILL.md`）。

### 豆包（桌面端 / 工作模式）

把 `papertodo-ops` 文件夹放进豆包的用户 Skill 目录，例如：

```
%LOCALAPPDATA%\DoubaoWork\User Data\Default\.doubaowork\agent_mode\workspace\.user_skills\
```

放好后重新打开对话，确认 Skill 列表里出现 `papertodo-ops`。如果你的豆包版本支持在界面里导入 Skill，也可以直接导入这个文件夹。

### Claude Code

放到个人 Skill 目录（所有项目都能用）：

```
~/.claude/skills/papertodo-ops/SKILL.md
```

Windows 上即 `%USERPROFILE%\.claude\skills\papertodo-ops\`。也可以放在某个项目的 `.claude/skills/` 下，只对该项目生效。

### Claude 桌面版 / 网页版

把 `papertodo-ops` 文件夹打包成 zip，在 Claude 的设置里找到 Skills，上传该 zip。

### Cursor 及其他客户端

- 客户端支持 Agent Skills（读取带 `SKILL.md` 的文件夹）时，按该客户端文档放到它的 Skill 目录即可。
- 不支持 Skill 时，把 `SKILL.md` 正文（去掉开头 `---` 之间的部分）粘贴到客户端的自定义规则 / 系统提示里。需要用图片或数据合并时，再把 `references/` 里对应文件的内容一起贴进去。

### 工具名前缀

不同客户端给 MCP 工具加的前缀不一样（例如 `mcp__papertodo__list_papers`、`papertodo.list_papers` 等）。Skill 里只写短名，例如 `list_papers`，AI 会对应到当前客户端的实际工具名。

## 试一试

装好后可以对 AI 说：

- “看看我 PaperTodo 里有哪些纸片。”
- “新建一张叫『本周』的待办纸，加上：周三交周报、周五约老王。”
- “把『本周』里‘交周报’那条标成已完成。”
- “明天上午 9 点提醒我『约老王』这条。”

## 和官方提示词的差异

本 Skill 的规则与官方提示词保持一致，主要做了这些整理：

- 按场景分成章节，便于 AI 按需查阅；图片导入、外部编辑与合并两部分拆到 `references/`，按需加载。
- 图片导入补了一份完整的 PowerShell 示例，**兼容 Windows 自带的 PowerShell 5.1**（不依赖仅 PowerShell 7 才有的 `ProcessStartInfo.ArgumentList`），并在 5.1 下测试过参数传递（空格、双引号、反斜杠、中文文件名）。
- 补充了少量操作性说明，如“不要为了让用户看到结果自行弹出纸片”“删除前确认目标”，都不改变官方规则的含义。
- 官方提示词里没有写明的参数细节（字段结构、取值范围等）一律写成“以工具定义为准”，不自行假设。

## 注意事项与免责声明

- **重要数据请先备份。** PaperTodo 写盘前会自动做快照，但 MCP 仍是 Preview 功能。
- 删除、批量修改、外部编辑数据文件这类操作，建议先让 AI 列出要改的内容，确认后再执行。
- AI 生成的 `!p` 脚本胶囊、图片导入命令等会在本机执行，运行前请自己看一遍。
- 本 Skill 按“现状”提供，不保证适用于所有版本与客户端。因使用本 Skill 导致的数据丢失或其他问题，本 Skill 的整理者不承担责任。
- 如果 PaperTodo 原作者对本 Skill 的分享有异议，请提 issue，我会及时处理。

## 反馈

发现与新版本不一致、AI 调用出错等问题，欢迎提 issue 或 PR。PaperTodo 本身的问题请到[官方仓库](https://github.com/snownico0722/PaperTodo/issues)反馈。
