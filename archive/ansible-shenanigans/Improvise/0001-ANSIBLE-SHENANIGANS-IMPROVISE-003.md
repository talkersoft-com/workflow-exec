# Improvisation: Phase 0001 — verify's runner check failed (silent chown)

## Improvisation ID
`0001-ANSIBLE-SHENANIGANS-IMPROVISE-003`

## Source
- Task: `Tasks/0001-ANSIBLE-SHENANIGANS-TASK.md`
- Test: `Test/0001-ANSIBLE-SHENANIGANS-TEST.md`
- Failed: `verify` check 5 (`ansible_runner.run no-op playbook (localhost)`)

## What Failed

`./ansible-cli verify` ran 5/6 PASS. The `ansible_runner.run` noop check returned `rc != 0` with a Python traceback truncated to 300 chars by `verify.py`'s error formatter. Reproducing the noop *manually* (creating the private_data_dir as root then chowning to `cloudmanager` cleanly) succeeded with `rc: 0`. So the playbook + runner work — it's the verify harness that's wrong.

## Root Cause

The `ansible-cli` wrapper exempted `verify` from sudo on the assumption it was a read-only smoke test. But `verify.py:check_runner_noop` uses `tempfile.TemporaryDirectory()` (created as the invoking user, `todd`) and then calls `subprocess.run(["chown", "-R", "cloudmanager:cloudmanager", td], check=False)`. Running as `todd`, the chown silently fails (no perms to chown to a different user). The dir stays `todd`-owned with default 700 mode. When `cloudmanager` (via sudo -u inside `run_as_user`) tries to run ansible-runner against it, runner's init fails because it can't write `artifacts/`, `env/`, etc. into a directory it doesn't own.

## Adaptation

Two changes:

1. **Wrapper fix (source-level)**: `ansible-cli` now requires sudo for `verify` too. Only `configure` is exempt (it just needs `VAULT_TOKEN`, not user-switching). Mirrors `install`'s sudo requirement and aligns with the fact that *every* verify check that calls `run_as_user(install_user, ...)` already needs sudo to work.
2. **No verify.py change needed** — once running as root via the wrapper, the chown actually succeeds and the runner gets a properly-owned dir.

## Outcome

(To be filled after retry.)

## Reusable Knowledge

`subprocess.run(["chown", ...], check=False)` is dangerous in privileged-vs-unprivileged ambiguity — when run unprivileged it silently no-ops. If a script depends on the chown succeeding, either run it `check=True` or design the privilege model so the script *always* runs as root. The latter is usually simpler.
