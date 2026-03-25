# TODO

---

### ~~Slow command output desync~~ ✅ Fixed

Replaced timeout-based framing with prompt-event framing. `read_pdb_output`
now emits `("prompt", None)` events when it sees `(Pdb) ` in the byte stream.
`send_to_pdb` waits for the next prompt event via `_wait_for_prompt()` instead
of sleeping for a fixed duration. `continue` and other long-running commands
now work correctly regardless of how long they take.

---

### Windows conda: `VIRTUAL_ENV` set to wrong path

On Windows, `find_venv_details()` returns `CONDA_PREFIX` itself as `venv_bin_dir`
(no `bin` subdirectory). The subsequent `os.path.dirname(venv_bin_dir)` resolves
to the parent of the conda environment instead of the environment root.

Fix: detect the Windows conda case and skip the `dirname` call.
