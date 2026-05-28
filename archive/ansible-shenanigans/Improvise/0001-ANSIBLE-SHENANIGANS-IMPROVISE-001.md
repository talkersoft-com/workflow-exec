# Improvisation: Phase 0001 — pipx install failed (PermissionError on ~/.local)

## Improvisation ID
`0001-ANSIBLE-SHENANIGANS-IMPROVISE-001`

## Source
- Task: `Tasks/0001-ANSIBLE-SHENANIGANS-TASK.md`
- Orchestration: `Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md`
- Failed Step: Step 2 (`sudo ./ansible-cli install`), step "Install ansible-core"

## What Failed

`sudo -u cloudmanager -H bash -lc "pipx install --include-deps 'ansible-core==2.16.0'"` exited 1 with:

```
PermissionError: [Errno 13] Permission denied: '/home/cloudmanager/.local/share'
```

## Root Cause

The install script's `ensure_install_user_home` method runs `mkdir -p` as root for `~/.local/bin` and `~/.local/pipx`. `mkdir -p` materializes the parent `~/.local` as root-owned too. The follow-up `chown` only targeted the leaf dirs, leaving `~/.local` itself owned by `root:root` and mode 755. When pipx tried to create `~/.local/share/man` (its own subdir), it failed because `cloudmanager` can read but not write to its own `.local`.

```
$ sudo ls -la /home/cloudmanager/.local
drwxr-xr-x 4 root         root          ...  .
drwxr-x--- 3 cloudmanager cloudmanager  ...  ..
drwxr-xr-x 2 cloudmanager cloudmanager  ...  bin
drwxr-xr-x 4 cloudmanager cloudmanager  ...  pipx
```

## Adaptation

Two-part fix:

1. **Immediate**: `sudo chown -R cloudmanager:cloudmanager /home/cloudmanager/.local` to repair the existing perms on this box.
2. **Source fix**: `install.py:ensure_install_user_home` now creates `~/.local` and `~/.local/share` explicitly with proper ownership, plus a final `chown -R` of `~/.local` to the install user. Idempotent — no harm re-running.

Both applied. Re-running install.

## Outcome

(To be filled in after retry passes.)

## Reusable Knowledge

When provisioning a user-scoped tool tree (`~/.local`, `~/.config`, etc.) from a root-running installer, always chown the root of the tree, not just the specific subdirs being created. `mkdir -p` materializes intermediate parents as root, and that bites later when the target user's own tools (pipx, npm, cargo) try to create siblings.
