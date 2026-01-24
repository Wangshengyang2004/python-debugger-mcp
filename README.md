# python-debugger-mcp: Python Debugger Interface for Claude/LLMs

![Python](https://img.shields.io/badge/Python-3.12+-blue.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)

python-debugger-mcp provides tools for using Python's debugger (pdb) with Claude and other LLMs through the Model Context Protocol (MCP). This was inspired by [debug-gym](https://microsoft.github.io/debug-gym/) by Microsoft, which showed gains in various coding benchmarks by providing a coding agent access to a python debugger.

Forked from [mcp-pdb](https://github.com/danielgafni/mcp-pdb).

[**中文文档**](README_CN.md)

## ⚠️ Security Warning

This tool executes Python code through the debugger. Use in trusted environments only.

## Installation

Works best with [uv](https://docs.astral.sh/uv/getting-started/features/)

### Claude Code
```bash
# Install the MCP server directly from GitHub
claude mcp add python-debugger-mcp -- uv run --with git+https://github.com/Wangshengyang2004/python-debugger-mcp python-debugger-mcp

# Alternative: Install with specific Python version
claude mcp add python-debugger-mcp -- uv run --python 3.13 --with git+https://github.com/Wangshengyang2004/python-debugger-mcp python-debugger-mcp

# Note: The -- separator is required for Claude Code CLI
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

## Available Tools

### Session Management
| Tool | Description |
|------|-------------|
| `start_debug(file_path, use_pytest, args)` | Start a debugging session for a Python file |
| `restart_debug()` | Restart the current debugging session |
| `end_debug()` | End the current debugging session |
| `get_debug_status()` | Show the current state of the debugging session |

### Execution Control
| Tool | Description |
|------|-------------|
| `send_pdb_command(command)` | Send a raw PDB command |

### Stack Navigation
| Tool | Description |
|------|-------------|
| `navigate_stack(direction, count)` | Move up/down the stack trace |
| `get_stack_trace()` | Print the current stack trace |

### Breakpoints
| Tool | Description |
|------|-------------|
| `set_breakpoint(file_path, line_number)` | Set a breakpoint at a specific line |
| `set_breakpoint_with_condition(file, line, condition)` | Set a breakpoint with a condition |
| `set_temporary_breakpoint(file, line)` | Set a temporary breakpoint (deleted after first hit) |
| `enable_breakpoint(bp_number)` | Enable a disabled breakpoint |
| `disable_breakpoint(bp_number)` | Disable a breakpoint |
| `ignore_breakpoint(bp_number, count)` | Ignore a breakpoint for N hits |
| `clear_breakpoint(file_path, line_number)` | Clear a breakpoint |
| `list_breakpoints()` | List all current breakpoints |

### Variable Watch
| Tool | Description |
|------|-------------|
| `examine_variable(variable_name)` | Get detailed info about a variable |
| `set_display(expression)` | Add an expression to auto-display |
| `list_displays()` | List all active display expressions |
| `clear_display(expression_or_id)` | Remove a display expression |

### Code Inspection
| Tool | Description |
|------|-------------|
| `list_source(long_list)` | List source code around current breakpoint |
| `get_function_args()` | Print function arguments |
| `get_return_value()` | Print return value of last function call |
| `get_variable_type(expression)` | Print type of a variable |

### Postmortem Debugging
| Tool | Description |
|------|-------------|
| `enter_postmortem_mode()` | Enter postmortem debugging mode |

## Common PDB Commands

| Command | Description |
|---------|-------------|
| `n` | Next line (step over) |
| `s` | Step into function |
| `c` | Continue execution |
| `r` | Return from current function |
| `p variable` | Print variable value |
| `pp variable` | Pretty print variable |
| `b file:line` | Set breakpoint |
| `cl num` | Clear breakpoint |
| `l` | List source code |
| `q` | Quit debugging |

## Features

- Project-aware debugging with automatic virtual environment detection
- Support for both direct Python debugging and pytest-based debugging
- Automatic breakpoint tracking and restoration between sessions
- Works with UV package manager
- Variable inspection with type information and attribute listing
- Conditional breakpoints and temporary breakpoints
- Variable watch expressions (display)
- Stack navigation and inspection
- Postmortem debugging support

## Troubleshooting

### Claude Code Installation Issues

If you encounter an error like:
```
MCP server "python-debugger-mcp" Connection failed: spawn /Users/xxx/.local/bin/uv run --python 3.13 --with python-debugger-mcp python-debugger-mcp ENOENT
```

Make sure to include the `--` separator when using `claude mcp add`:
```bash
# ✅ Correct
claude mcp add python-debugger-mcp -- uv run --with git+https://github.com/Wangshengyang2004/python-debugger-mcp python-debugger-mcp

# ❌ Incorrect (missing --)
claude mcp add python-debugger-mcp uv run --with git+https://github.com/Wangshengyang2004/python-debugger-mcp python-debugger-mcp
```

To verify your installation:
```bash
# Check if python-debugger-mcp is listed
claude mcp list | grep python-debugger-mcp

# Check server status in Claude Code
# Type /mcp in Claude Code to see connection status
```

## License

MIT License - See LICENSE file for details.

## Acknowledgments

- Original project: [mcp-pdb](https://github.com/danielgafni/mcp-pdb) by danielgafni
- Inspired by [debug-gym](https://microsoft.github.io/debug-gym/) by Microsoft
