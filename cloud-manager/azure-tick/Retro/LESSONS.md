# Lessons: cloud-manager/azure-tick

## What would have helped

### Verify playbook revision matches current content before chasing materialize bugs
The most painful detour was Phase 12 — after FIX-001 cleared the SSH + inventory + Vault path issues, the playbook ran exactly **one task** (Gathering Facts). The cause was that `pb_EMEJCH08GG`'s `playbook_revisions` row was the original 61-byte probe content; the current 2969-byte pg content lived in `playbooks.content` but had never been snapshotted. `PlaybookRunService.TriggerMultiVmAsync` records the run with `run.PlaybookRevisionId = <stale>`, so porch materialized the stale probe. **For any future plan touching the playbook-run path: before debugging, `curl /api/v1/playbook/<id>/materialized` AND `psql ... playbook_revisions WHERE playbook_id = ... ORDER BY created DESC LIMIT 1` — if the revision content differs from `playbooks.content`, you'll burn an hour chasing ghost behavior.**

### Ubuntu 24.04 ships pipx 1.4.3 — no `--global` flag
The PLAN said "pipx --global" but 1.4.3 doesn't have it (added in 1.5+). `pipx install` reads `PIPX_HOME` + `PIPX_BIN_DIR` env vars though, so `PIPX_HOME=/opt/pipx PIPX_BIN_DIR=/usr/local/bin pipx install ansible-runner` lands the venv at `/opt/pipx/venvs/ansible-runner/` and the launcher at `/usr/local/bin/ansible-runner`. Same outcome. **For any plan that prescribes a tool's CLI flag, verify with `tool --help` against the host's actual version before merging the plan.**

### Libvirt domain name in this stack is the public_id, not the friendly name
`vorch-lib/provision.go` provisions VMs with `domain.Name = publicId` (e.g. `vm_189KV047V2`). The friendly name (`postgres-test`) is for the UI only. Phase 5's backfill script initially passed friendly names → all skipped. **When writing maintenance scripts that bridge the app's data model to libvirt, lead with the public_id.**

### `psycopg2` isn't on the host system Python by default
Backfill script crashed on import. Quick fix via `apt install python3-psycopg2`. Adding it to `install-porch-service.py` next iteration would tighten the deploy story.

### `ProtectHome=yes` blocks both `/home` AND `/root`
Same trap from charmed-panda's FIX-001. `sudo pipx install` lands in `/root/.local/share/pipx/`, which `ProtectHome=yes` hides. **Any "install to root's user-local dir" approach is incompatible with hardened systemd; either go to `/opt` (via `PIPX_HOME` override) or accept `ProtectHome=no`.**

### `become_user: <unprivileged>` requires `acl` on the target VM
Ubuntu cloud images don't include `acl` by default. ansible's `become_user` mechanism uses `chmod +A user:<u>:rx:allow` ACL syntax — without `acl`, GNU chmod errors with `invalid mode: 'A+user:postgres:rx:allow'`. Easy fix on one VM (`apt install -y acl`), but for VM provisioning at scale this belongs in cloud-init userdata.

### When a task has `no_log: true`, the only way to see the error is to strip it
Phase 12's last hour was spent staring at `runner_on_failed` events with `{"censored": "the output has been hidden..."}`. After inserting a debug revision (`pbrev_AZTICK002`) with `no_log` stripped, the real error was visible immediately (`community.postgresql.postgresql_user` 4.2.0 rejects `priv:`). **Operationally we need an admin-only "show error message even when no_log was set" toggle so debugging doesn't require database surgery.**

## Fix files written
- `FIX-001.md` — Vault SSH key Vault path (`vm/instances/<id>/ssh_key` → `vm/instances/<id>` + JSON-extract `private_key`); inventory `ansible_ssh_private_key_file` from relative `env/ssh_key` to absolute `<workdir>/env/ssh_key`.
- `FIX-002.md` — Stale playbook revision (manually inserted `pbrev_AZTICK001` + 002 as workaround) + pg playbook outdated `priv:` param + ancillary fixes (VAULT_TOKEN env on porch unit, Vault TLS skip-verify, `acl` package on target VM).

## Deviations from plan
- **PLAN said `pipx --global`** — used `PIPX_HOME=/opt/pipx PIPX_BIN_DIR=/usr/local/bin` instead because pipx 1.4.3 doesn't support `--global`. Same outcome.
- **PLAN didn't mention `VAULT_TOKEN` env on porch unit** — porch's `vault.NewClient` is gated by `os.Getenv("VAULT_TOKEN")`. Added via `_vault_token()` reader in the install script.
- **PLAN didn't mention Vault TLS skip-verify on the runner side** — charmed-panda's FIX-001 fixed the API side; same issue exists on porch's Go vault client. Added `cfg.ConfigureTLS(&vaultapi.TLSConfig{Insecure: true})` gated by `VAULT_SKIP_VERIFY`.

## Follow-ups
- `[[follow-up-auto-revision-on-edit]]` — when `vm.playbooks.content` is updated without going through a revise endpoint, the next run materializes a stale revision. API-side fix: auto-create a `playbook_revisions` row whenever `content` is updated.
- `[[follow-up-pg-playbook-postgres-user-priv]]` — pg playbook uses `priv:` (renamed to `privs:` in community.postgresql 3.x+).
- `[[follow-up-cloud-init-include-acl]]` — add `acl` to the cloud-init image for VM provisioning so `become_user` Just Works.
- `[[follow-up-no-log-error-toggle]]` — admin-only UI affordance to surface `no_log`-suppressed errors.
- `[[follow-up-vm-ip-refresh]]` — DHCP-change handling (carried from charmed-panda).
- `[[follow-up-pipx-1.5-update]]` — once Ubuntu's pipx package crosses 1.5, switch the install script to `--global` so we can drop the `PIPX_HOME` override.
- `[[follow-up-porch-install-script-psycopg2]]` — install `python3-psycopg2` via apt during porch install so the backfill script works on a fresh host.
