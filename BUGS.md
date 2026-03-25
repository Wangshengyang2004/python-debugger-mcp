# Known Bugs

## Critical

### BUG-1: `readline()` blocks on `(Pdb) ` prompt — output never arrives until process exits

**File:** `src/python_debugger_mcp/main.py:37`

`read_pdb_output` uses `iter(process.stdout.readline, b'')`. PDB's prompt `(Pdb) ` is written to stdout **without a trailing newline**. `readline()` blocks waiting for `\n` that never comes, so the prompt line is never put into the queue. The consumer (`get_pdb_output`) then falls through on timeout every time, and the `(Pdb)` early-exit heuristic at line 70 never fires. All command responses are delayed by the full timeout, and the prompt bleeds into the next response.

**Reproduction:** Any `send_pdb_command` call — the `(Pdb) ` prefix visible in responses like `(Pdb) Breakpoint 1 at ...` is the leaked prompt from the *previous* command.

**Fix:** Read stdout in raw byte chunks (e.g. `os.read(fd, 4096)`) and split on `(Pdb) ` as the frame delimiter, or run the child under a PTY.

---

### BUG-2: `VIRTUAL_ENV` set to wrong path for conda environments

**File:** `src/python_debugger_mcp/main.py:384`

```python
venv_dir = os.path.dirname(os.path.dirname(venv_bin_dir))
```

For a standard venv, `venv_bin_dir` = `/project/.venv/bin`, so two `dirname` calls yield `/project` (the project root), not `/project/.venv`. For conda, `find_venv_details` returns `CONDA_PREFIX/bin` as `venv_bin_dir`, so two `dirname` calls yield the *parent* of the conda environment (e.g. `/Users/simonwsy` instead of `/Users/simonwsy/miniconda3`).

**Reproduction:** Confirmed by Codex live run — `VIRTUAL_ENV: /Users/simonwsy` in subprocess logs.

**Fix:** Use `os.path.dirname(venv_bin_dir)` (one level up) for standard venvs. For conda, `VIRTUAL_ENV` should be set to `CONDA_PREFIX` directly.

---

### BUG-3: Conditional breakpoints use wrong PDB syntax

**File:** `src/python_debugger_mcp/main.py:1086`

```python
command = f"b {rel_file_path}:{line_number} if {escaped_condition}"
```

PDB's syntax for conditional breakpoints is `b filename:lineno, condition` (comma-separated), **not** `b filename:lineno if condition`. The `if` form is only valid for `b functionname`. Using `if` with a file:line target causes PDB to return `*** Bad lineno: <line> if <condition>`.

**Reproduction:** Confirmed by Codex live run — `set_breakpoint_with_condition` returns `*** Bad lineno`.

**Fix:** Change to `f"b {rel_file_path}:{line_number}, {condition}"`.

---

### BUG-4: Breakpoint restore on restart ignores `temporary` and `condition` flags

**File:** `src/python_debugger_mcp/main.py:513`

```python
bp_command_rel = f"b {bp_rel_path}:{line_num}"
```

All breakpoints are restored with plain `b`, regardless of whether they were set as `tbreak` (temporary) or with a condition. Temporary breakpoints become permanent after restart; conditional breakpoints lose their condition.

**Fix:** Check `bp_data` dict for `"temporary"` and `"condition"` keys and reconstruct the appropriate command.

---

## Medium

### BUG-5: `display` IDs are not synchronized with PDB

**File:** `src/python_debugger_mcp/main.py:1212`

```python
display_id = len(displays) + 1
```

The server assigns its own sequential IDs, but PDB assigns its own independent IDs. `clear_display` passes the server-side ID to `undisplay`, which may clear the wrong expression or fail silently.

---

### BUG-6: `enter_postmortem_mode` is not implemented

**File:** `src/python_debugger_mcp/main.py:1327`

The function only returns a help text string. It does not call `pdb.pm()`, access `sys.last_traceback`, or start any debug session. Listed as a feature in README.

---

## Low

### BUG-7: `sanitize_arguments` blocks `<` and `>` unnecessarily

**File:** `src/python_debugger_mcp/main.py:244`

Blocking `<` and `>` prevents valid pytest arguments like `--timeout=>5` or shell-style redirects that are legitimate in some test frameworks. These characters are only dangerous in shell context; since `subprocess.Popen` is called with a list (not `shell=True`), they pose no injection risk.

### BUG-8: `send_pdb_command` has no input validation

**File:** `src/python_debugger_mcp/main.py:544`

`start_debug` sanitizes its `args` parameter, but `send_pdb_command` passes the command string directly to PDB stdin with no validation. PDB's `!` prefix executes arbitrary Python, so `send_pdb_command("!import os; os.system('...')")` works as-is.
