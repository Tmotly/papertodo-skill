# 笔记图片：导入与查看

## 查看已有图片

1. `get_paper` 取得笔记正文，找到 Markdown 图片引用里的 `i:图片ID`。
2. 用该笔记 ID 和图片 ID 调用 `read_note_image`（参数格式以工具定义为准），返回的图片可直接用来回答。

## 追加图片到已有 Markdown 笔记

限制与约束：

- 只追加到已存在的 Markdown 笔记（上下文已有笔记 ID，或先 `list_papers` 找到）。
- 原始图片字节 ≤ 4 MiB，超出直接告知用户，不做压缩后偷偷写入。
- 唯一合法写入路径是 PaperTodo 可执行文件的 `--import-note-image` 子命令，图片字节从其**二进制 stdin** 传入。命令不接受文件路径或 URL 作为来源。
- 不要调用任何 Base64 图片写入工具，也不要把 Base64 放进笔记正文或命令行参数。

步骤：

1. 取得图片字节。可用图片来源：用户附件、本地文件（用文件读取工具按字节读入）、已下载到本地的资源。客户端拿不到字节时，明确说明无法写入。
2. 校验大小 `≤ 4194304` 字节。
3. 定位可执行文件：使用本次 PaperTodo MCP 配置中的可执行文件路径（即启动 `--mcp` 服务的那个程序）。不要自行猜测路径。
4. 用子进程写入并关闭 stdin，只向对话回传简短结果。

PowerShell（本地 Windows 客户端，`Process.StandardInput.BaseStream` 写原始字节；Windows 自带的 PowerShell 5.1 与 PowerShell 7 均可运行）：

```powershell
$exe    = '<PaperTodo 可执行文件路径>'
$noteId = '<笔记 paper_id>'
$img    = '<图片本地绝对路径>'
$name   = '<可选图片名称，不需要则留空>'

$bytes = [System.IO.File]::ReadAllBytes($img)
if ($bytes.Length -gt 4194304) { throw '图片超过 4 MiB' }

# 按 Windows 命令行规则给参数加引号（兼容 PowerShell 5.1，不依赖 ArgumentList）
function Quote-Arg([string]$s) {
    '"' + (($s -replace '(\\*)"', '$1$1\"') -replace '(\\+)$', '$1$1') + '"'
}
$argList = @('--import-note-image', '--paper-id', (Quote-Arg $noteId))
if ($name) { $argList += @('--image-name', (Quote-Arg $name)) }

$psi = New-Object System.Diagnostics.ProcessStartInfo
$psi.FileName = $exe
$psi.Arguments = $argList -join ' '
$psi.UseShellExecute = $false
$psi.RedirectStandardInput = $true
$psi.RedirectStandardOutput = $true
$psi.RedirectStandardError = $true

$p = [System.Diagnostics.Process]::Start($psi)
$p.StandardInput.BaseStream.Write($bytes, 0, $bytes.Length)
$p.StandardInput.BaseStream.Flush()
$p.StandardInput.Close()
$out = $p.StandardOutput.ReadToEnd()
$err = $p.StandardError.ReadToEnd()
$p.WaitForExit()
"exit=$($p.ExitCode)"; $out; $err
```

要点：

- 必须用 `BaseStream` 写入并 `Close()`，不要用 `StandardInput.WriteLine`、文本管道或 `echo |`，否则二进制会被破坏。
- 不要用 `cmd /c ... < file` 之类的重定向；同样可能做文本转换。
- 退出码：`0` 成功，`1` 导入失败，`2` 用法错误。只把 stdout 的简短 JSON 或 stderr 错误回给对话。

## 失败处理

- 退出码非 0 时，先 `get_paper` 核对正文是否已出现新的 `i:` 引用，再决定是否需要重试；不要在未核对前重复追加，避免同一张图写两遍。
- 报错提示权限、路径或文件被占用时，按实际情况说明，不改为直接改数据文件。
