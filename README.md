# python-debugger-mcp: Python Debugger Interface for Claude/LLMs

![Python](https://img.shields.io/badge/Python-3.12+-blue.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)

python-debugger-mcp provides tools for using Python's debugger (pdb) with Claude and other LLMs through the Model Context Protocol (MCP). This was inspired by [debug-gym](https://microsoft.github.io/debug-gym/) by Microsoft, which showed gains in various coding benchmarks by providing a coding agent access to a python debugger.

Based on [mcp-pdb](https://github.com/danielgafni/mcp-pdb) with significant enhancements:

| Feature | mcp-pdb | python-debugger-mcp |
|---------|---------|---------------------|
| Tools Count | 16 | **24** |
| Stack Navigation | - | navigate_stack, get_stack_trace |
| Conditional Breakpoints | - | set_breakpoint_with_condition |
| Temporary Breakpoints | - | set_temporary_breakpoint |
| Breakpoint Management | - | enable_breakpoint, disable_breakpoint, ignore_breakpoint |
| Variable Watch | - | set_display, list_displays, clear_display |
| Code Inspection | - | list_source, get_function_args, get_return_value, get_variable_type |
| Postmortem Debugging | - | enter_postmortem_mode |
| Tests | - | Comprehensive unit tests |
| Multi-language Docs | - | English + Chinese (README_CN.md) |
| IDE Support | Limited | Claude Code, Cursor, OpenCode, Windsurf |

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

### Cursor

**Method 1: Using mcp.json (Recommended)**

Create or edit `~/.cursor/mcp.json` (global) or `.cursor/mcp.json` (project-specific):

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

Then restart Cursor to load the MCP server.

**Method 2: One-click Install**

Open Cursor Settings > MCP > Click "+ Add MCP Server" and paste:
```
uv run --with git+https://github.com/Wangshengyang2004/python-debugger-mcp python-debugger-mcp
```

### OpenCode

OpenCode is built on VS Code architecture and supports MCP servers similarly.

**Method 1: Using Settings JSON**

1. Open Command Palette (Ctrl+Shift+P)
2. Run "Preferences: Open Settings (JSON)"
3. Add the following configuration:

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

**Method 2: Using .mcp.json file**

Create `~/.opencode/mcp.json` for global settings:

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

Then restart OpenCode to apply changes.

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

## Success Stories

### Debugging 2048 Game Logic

Used python-debugger-mcp to debug a 2048 game project and successfully found and fixed a bug:

**Bug Found**: In `game.py`, the `_check_game_over()` method was returning early when empty cells existed, but forgot to set `game_over = False`. This caused the game to incorrectly report "game over" even when empty cells were available.

**How MCP Helped**:
1. Set breakpoints using `set_breakpoint()` to inspect game state
2. Step through code with `send_pdb_command("n")`
3. Examine variables like `game.board` with `examine_variable()`
4. Clear and manage breakpoints with `clear_breakpoint()`
5. Verify the fix worked correctly

**Result**: Fixed the bug - now properly detects game over state.

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
