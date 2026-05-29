# CHANGE-0009 — ansible-runner system install + ProtectHome=yes

## apt vs pipx — decision: pipx
Verified with `apt-cache policy ansible-runner` on Ubuntu 24.04: no candidate version (returned empty). `ansible-core` and `ansible` themselves are in apt; `ansible-runner` is not.

Fallback to pipx. **pipx 1.4.3 (Ubuntu 24.04 default) does not have `--global`** (added in pipx 1.5+). Worked around by setting `PIPX_HOME=/opt/pipx PIPX_BIN_DIR=/usr/local/bin` so the install lands at `/opt/pipx/venvs/ansible-runner/` with the launcher at `/usr/local/bin/ansible-runner` — no chain through `/home/cloudmanager/.local` or `/root/.local`.

## Final binary location
`/usr/local/bin/ansible-runner` → symlink → `/opt/pipx/venvs/ansible-runner/bin/ansible-runner`.

Manual install verified: `ansible-runner --version` → `2.4.3`.

## Unit changes
- `Environment="ANSIBLE_RUNNER_BIN=/usr/local/bin/ansible-runner"` added to the template.
- `ProtectHome=yes` (was already present after the previous edit).

## Uninstall changes
- Added cleanup of FIX-001 leftovers (`/usr/local/bin/ansible-runner` + `/usr/local/bin/ansible-runner.real`).

## Install script step
Added `ensure_ansible_runner` step to the `install()` pipeline. Tries apt first (future-proof), falls back to the `PIPX_HOME=/opt/pipx` pipx invocation. Re-symlinks `/usr/local/bin/ansible-runner` regardless so the path is deterministic.

## Build status
`python3 -m py_compile scripts/porch/install-porch-service.py scripts/porch/uninstall-porch-service.py` exits 0.
