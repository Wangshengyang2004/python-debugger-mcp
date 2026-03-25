# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- Run the server: `uv run python-debugger-mcp`
- Lint: `uv run ruff check`
- Type check: `uv run pyright`
- Run all tests: `uv run pytest`
- Run a single test file: `uv run pytest tests/test_helpers.py`
- Run a single test: `uv run pytest tests/test_helpers.py::TestFindProjectRoot::test_finds_pyproject_toml`

## Architecture

The entire server is implemented in a single file: `src/python_debugger_mcp/main.py`. It exposes PDB as MCP tools via the FastMCP framework.

### Core mechanism

The server spawns a `pdb` subprocess with `subprocess.Popen` (stdin/stdout piped), then drives it by writing commands to stdin and reading responses via a background thread that drains stdout into a `queue.Queue`. All global state is module-level: `pdb_process`, `pdb_running`, `pdb_output_queue`, `breakpoints`, `displays`, `current_file`, `current_project_root`.

### Environment detection

`start_debug` auto-detects the debuggee's runtime environment:
1. **uv project** — detected by presence of `uv.lock` + `uv` in PATH; runs via `uv run -- python -m pdb` or `uv run -- pytest --pdb`
2. **venv** — `find_venv_details()` scans for `.venv/venv/env/...` directories in the project root and parent; intentionally scans the filesystem *before* checking `VIRTUAL_ENV`/`CONDA_PREFIX` env vars (which may point to this server's own uv env, not the debuggee's)
3. **System Python** — fallback

Project root is found by walking upward from the file's directory until `pyproject.toml`, `.git`, `setup.py`, `requirements.txt`, `Pipfile`, or `poetry.lock` is found.

### MCP tools exposed

| Tool | PDB equivalent |
|---|---|
| `start_debug` | Launch pdb subprocess |
| `send_pdb_command` | Pass-through; auto-appends `l .` after navigation commands |
| `set_breakpoint` / `clear_breakpoint` / `list_breakpoints` | `b` / `cl` / `b` |
| `set_breakpoint_with_condition` | `b file:line if condition` |
| `set_temporary_breakpoint` | `tbreak` |
| `enable_breakpoint` / `disable_breakpoint` / `ignore_breakpoint` | `enable` / `disable` / `ignore` |
| `navigate_stack` | `up` / `down` |
| `get_stack_trace` | `where` |
| `examine_variable` | `p`, `pp`, `type()`, `dir()` combined |
| `get_variable_type` | `whatis` |
| `set_display` / `list_displays` / `clear_display` | `display` / `undisplay` |
| `list_source` | `l` or `ll` |
| `get_function_args` | `a` |
| `get_return_value` | `retval` |
| `restart_debug` | `end_debug` + `start_debug` with same params |
| `end_debug` | SIGINT → `q` → SIGTERM → SIGKILL |

### Breakpoint persistence

Breakpoints are tracked in memory across `restart_debug` calls using the `breakpoints` dict keyed by absolute file path. On session start, previously tracked breakpoints are re-sent to the new pdb process. Breakpoints are stored with both their line number and PDB-assigned `bp_number` (used for `clear`/`enable`/`disable`).

### Tests

Tests in `tests/test_helpers.py` cover only the three pure helper functions: `find_project_root`, `find_venv_details`, and `sanitize_arguments`. The subprocess-driving MCP tools are not unit-tested.
