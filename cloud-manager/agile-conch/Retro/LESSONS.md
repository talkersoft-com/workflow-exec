# Retro: cloud-manager/agile-conch

## The six retired triage patches

Each `fix_*.sql` was a live in-DB UPDATE against `vm.playbooks.content` that papered over a real bug. This workflow replaced every one with a code fix.

| Triage patch | Masked bug | Replaced by |
|---|---|---|
| `fix_pg_run_id.sql` | porch didn't inject `run_public_id` into ansible extravars, so `{{ run_public_id }}` rendered empty and the Vault write landed in the wrong path | **Phase 1** (porch extravars) + **Phase 4** (playbook revert) |
| `fix_hvac.sql` | originally bridged the missing `hvac` import; later obsoleted by the `delegate_to` workaround which itself is now gone | **Phase 4** (playbook revert — neither hvac patch nor delegate are needed) |
| `fix_delegate.sql` | `VAULT_ADDR` was `127.0.0.1`, unreachable from the target VM, so the playbook had to `delegate_to: localhost` to talk to Vault from the porch host | **Phase 2** (FQDN `VAULT_ADDR`) + **Phase 4** (playbook revert) |
| `fix_become.sql` | follow-on from `delegate_to: localhost` — root's tmp dir blocked the Vault module's working area when running under become | **Phase 4** (delegate goes away, so does the become hack) |
| `fix_tmp.sql` | porch's systemd unit had `ProtectHome=yes`, blocking `/root/.ansible/tmp` under the delegate path | **Phase 4** (delegate goes away, so does the tmp hack) |
| `fix_tls.sql` | Vault cert was issued for the FQDN, not `127.0.0.1`, so the original `VAULT_ADDR` triggered a TLS hostname mismatch and the patch disabled verification | **Phase 2** (FQDN `VAULT_ADDR`) → cert validates cleanly, no skip needed |

The pattern is clear: **fix_pg_run_id** was the root bug (porch missing a single field). The `127.0.0.1` choice for `VAULT_ADDR` was the secondary root bug. Every other patch was a downstream consequence of working around those two. Once Phases 1 + 2 landed, Phase 4 simply deleted four extra layers of compensation.

## What worked well

- **Read-the-stack-first approach.** Phase 1 + Phase 2 both came from a clean diff against porch's known-good behavior; we didn't speculate about ansible quirks before checking what porch actually emitted.
- **Seed-playbook plumbing (Phases 5/6/7) before E2E (Phase 8).** Once `bootstrap-policies.py` + `import-playbooks.py` were wired into `bootstrap-worker.py`, the E2E ran with zero manual setup. The Phase 8 run was reproducible from a fresh worker.
- **Idempotent scripts.** Both `bootstrap-policies.py` and `import-playbooks.py` are safe to re-run. That's what let Phase 7 land as plain "call them again after `hv init`" with no special-casing.
- **Per-run-scoped Vault token already existed.** cloud-manager-api was already minting tight per-run tokens — we only needed to fix the path it wrote to (`secretsPrefix`) and the policy that gated reads (Phase 5). No new auth machinery.
- **Phase 8 E2E succeeded on the first try.** Status `3` (succeeded), Vault secret present at the expected path, no operator intervention. That's the strongest signal the patches really were redundant.

## What would have helped

- **Earlier visibility into porch's extravars JSON.** The `run_public_id` omission would have been caught immediately if porch logged the rendered extravars at debug level on every run. Worth a follow-up: add a `--debug-extravars` flag or a one-line log at INFO when a playbook is dispatched.
- **A "playbook content drift" alarm.** Six live patches against `vm.playbooks.content` accumulated over a session before anyone noticed. A simple checksum compare between `seed/playbooks/` and the live DB content (now possible thanks to Phase 6) would have surfaced the drift in a single CI run. Worth wiring into CI as a post-deploy check.
- **VAULT_ADDR sourced from one place.** The `127.0.0.1` value was hard-coded in porch's emitter while the cert was provisioned for the FQDN. A single config source (already present in `/kvm-automator/config-server.yaml`) consulted by both porch and the cert provisioner would have made the mismatch impossible.
- **Phase 9 (rebootstrap) deferred.** We chose not to tear down and re-provision the dev box this session — the full `bootstrap-server.py` cycle is destructive and slow, and the Phase 7 wiring is the part that actually mattered for that proof. A follow-up workflow should run Phase 9 cleanly against a throwaway host to close the loop.
- **Test files weren't consulted automatically.** Each phase has a matching `Test/NNNN-TEST.md`, but the orchestration treated those as advisory rather than required. Worth making the workflow runner fail-closed if the test file isn't acknowledged.
