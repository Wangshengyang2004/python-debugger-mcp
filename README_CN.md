# python-debugger-mcp: Python 调试器接口 (Claude/LLM)

![Python](https://img.shields.io/badge/Python-3.12+-blue.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)

python-debugger-mcp 提供通过模型上下文协议 (MCP) 在 Claude 和其他 LLM 中使用 Python 调试器 (pdb) 的工具。这是受 Microsoft 的 [debug-gym](https://microsoft.github.io/debug-gym/) 启发的项目。

Forked from [mcp-pdb](https://github.com/danielgafni/mcp-pdb)。

## ⚠️ 安全警告

此工具通过调试器执行 Python 代码。仅在受信任的环境中使用。

## 安装

与 [uv](https://docs.astral.sh/uv/getting-started/features/) 配合使用效果最佳

### Claude Code
```bash
# 直接从 GitHub 安装 MCP 服务器
claude mcp add python-debugger-mcp -- uv run --with git+https://github.com/Wangshengyang2004/python-debugger-mcp python-debugger-mcp

# 替代方案：使用特定 Python 版本安装
claude mcp add python-debugger-mcp -- uv run --python 3.13 --with git+https://github.com/Wangshengyang2004/python-debugger-mcp python-debugger-mcp

# 注意：使用 claude mcp add 时需要 -- 分隔符
```

### Windsurf
```json
{
  "mcpServers": {
    "python-debugger-mcp": {
      "command": "uv",
      "args": [
        "run",
        "--with",
        "git+https://github.com/Wangshengyang2004/python-debugger-mcp",
        "python-debugger-mcp"
      ]
    }
  }
}
```

### Cursor

**方法 1：使用 mcp.json（推荐）**

创建或编辑 `~/.cursor/mcp.json`（全局）或 `.cursor/mcp.json`（项目专用）：

```json
{
  "mcpServers": {
    "python-debugger-mcp": {
      "command": "uv",
      "args": [
        "run",
        "--with",
        "git+https://github.com/Wangshengyang2004/python-debugger-mcp",
        "python-debugger-mcp"
      ]
    }
  }
}
```

然后重启 Cursor 以加载 MCP 服务器。

**方法 2：一键安装**

打开 Cursor 设置 > MCP > 点击 "+ Add MCP Server" 并粘贴：
```
uv run --with git+https://github.com/Wangshengyang2004/python-debugger-mcp python-debugger-mcp
```

### OpenCode

OpenCode 基于 VS Code 架构构建，同样支持 MCP 服务器。

**方法 1：使用 Settings JSON**

1. 打开命令面板（Ctrl+Shift+P）
2. 运行 "首选项: 打开设置 (JSON)"
3. 添加以下配置：

```json
{
  "mcpServers": {
    "python-debugger-mcp": {
      "command": "uv",
      "args": [
        "run",
        "--with",
        "git+https://github.com/Wangshengyang2004/python-debugger-mcp",
        "python-debugger-mcp"
      ]
    }
  }
}
```

**方法 2：使用 .mcp.json 文件**

创建 `~/.opencode/mcp.json` 用于全局设置：

```json
{
  "mcpServers": {
    "python-debugger-mcp": {
      "command": "uv",
      "args": [
        "run",
        "--with",
        "git+https://github.com/Wangshengyang2004/python-debugger-mcp",
        "python-debugger-mcp"
      ]
    }
  }
}
```

然后重启 OpenCode 以应用更改。

## 可用工具

### 会话管理
| 工具 | 描述 |
|------|-------------|
| `start_debug(file_path, use_pytest, args)` | 启动 Python 文件的调试会话 |
| `restart_debug()` | 重启当前调试会话 |
| `end_debug()` | 结束当前调试会话 |
| `get_debug_status()` | 显示调试会话的当前状态 |

### 执行控制
| 工具 | 描述 |
|------|-------------|
| `send_pdb_command(command)` | 发送原始 PDB 命令 |

### 堆栈导航
| 工具 | 描述 |
|------|-------------|
| `navigate_stack(direction, count)` | 在堆栈跟踪中向上/向下移动 |
| `get_stack_trace()` | 打印当前堆栈跟踪 |

### 断点
| 工具 | 描述 |
|------|-------------|
| `set_breakpoint(file_path, line_number)` | 在特定行设置断点 |
| `set_breakpoint_with_condition(file, line, condition)` | 设置带条件的断点 |
| `set_temporary_breakpoint(file, line)` | 设置临时断点（首次命中后删除） |
| `enable_breakpoint(bp_number)` | 启用已禁用的断点 |
| `disable_breakpoint(bp_number)` | 禁用断点 |
| `ignore_breakpoint(bp_number, count)` | 忽略断点 N 次 |
| `clear_breakpoint(file_path, line_number)` | 清除断点 |
| `list_breakpoints()` | 列出所有当前断点 |

### 变量监视
| 工具 | 描述 |
|------|-------------|
| `examine_variable(variable_name)` | 获取变量的详细信息 |
| `set_display(expression)` | 添加自动显示的表达式 |
| `list_displays()` | 列出所有活动的显示表达式 |
| `clear_display(expression_or_id)` | 移除显示表达式 |

### 代码检查
| 工具 | 描述 |
|------|-------------|
| `list_source(long_list)` | 列出当前断点周围的源代码 |
| `get_function_args()` | 打印函数参数 |
| `get_return_value()` | 打印上一个函数调用的返回值 |
| `get_variable_type(expression)` | 打印变量的类型 |

### 事后调试
| 工具 | 描述 |
|------|-------------|
| `enter_postmortem_mode()` | 进入事后调试模式 |

## 常用 PDB 命令

| 命令 | 描述 |
|---------|-------------|
| `n` | 下一行（跳过） |
| `s` | 单步进入函数 |
| `c` | 继续执行 |
| `r` | 从当前函数返回 |
| `p variable` | 打印变量值 |
| `pp variable` | 漂亮地打印变量 |
| `b file:line` | 设置断点 |
| `cl num` | 清除断点 |
| `l` | 列出源代码 |
| `q` | 退出调试 |

## 功能特点

- 项目感知调试，自动虚拟环境检测
- 支持直接 Python 调试和基于 pytest 的调试
- 自动断点跟踪和会话间恢复
- 与 UV 包管理器配合使用
- 变量检查，包含类型信息和属性列表
- 条件断点和临时断点
- 变量监视表达式（display）
- 堆栈导航和检查
- 事后调试支持

## 故障排除

### Claude Code 安装问题

如果遇到类似错误：
```
MCP server "python-debugger-mcp" Connection failed: spawn /Users/xxx/.local/bin/uv run --python 3.13 --with python-debugger-mcp python-debugger-mcp ENOENT
```

确保在使用 `claude mcp add` 时包含 `--` 分隔符：
```bash
# ✅ 正确
claude mcp add python-debugger-mcp -- uv run --with git+https://github.com/Wangshengyang2004/python-debugger-mcp python-debugger-mcp

# ❌ 错误（缺少 --）
claude mcp add python-debugger-mcp uv run --with git+https://github.com/Wangshengyang2004/python-debugger-mcp python-debugger-mcp
```

验证安装：
```bash
# 检查 python-debugger-mcp 是否已列出
claude mcp list | grep python-debugger-mcp

# 在 Claude Code 中检查服务器状态
# 在 Claude Code 中输入 /mcp 查看连接状态
```

## 许可证

MIT 许可证 - 有关详细信息，请参阅 LICENSE 文件。

## 致谢

- 原始项目：[mcp-pdb](https://github.com/danielgafni/mcp-pdb) by danielgafni
- 灵感来源：Microsoft 的 [debug-gym](https://microsoft.github.io/debug-gym/)
