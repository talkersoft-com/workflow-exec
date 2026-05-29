# Lessons: cloud-manager/jazzy-herring

## What would have helped

### Verify the existing path actually works end-to-end before assuming you can just "move" it
The charmed-panda PLAN was built on "vorch already does playbook execution; move that code into porch". Verifying the playbook execution path actually worked end-to-end would have surfaced the inventory-generation gap (FIX-002 root cause #2) during planning, not during Phase 9 verification. The Python `cloud-manager-worker` had been silently winning ~50% of `playbook-runs` deliveries for 4+ days, so any time a playbook DID work the Python side was probably the one running it. Vorch's path was probably never end-to-end-functional in production. Going forward: **for any carve-out plan that says "the existing path already works", actually run that path against a known-good test case before drafting**.

### `sed`-based bulk import rewrites are fine, but verify with `go vet` not just `go build`
The Phase 1 module rename used `find -exec sed -i` to rewrite ~12 import lines. Build succeeded. `go vet` also succeeded. But the qualified-reference rewrites in Phase 2 (e.g. `handlers.HandleCreateVmCommand` → `vm.HandleCreateVmCommand`) needed careful sed scripts that distinguished VM vs ansible call sites in the same package. One miss (`vmsub` needed `"internal/vm"` not `"internal/ansible"`) only surfaced because the build threw "undefined: vmsub". `go vet` caught nothing extra. For larger renames, write a tiny test that imports both target packages and instantiates one symbol from each — that catches more than build does.

### Go's `module main` is a footgun
The vorch-service module was declared `module main` in go.mod. This worked exactly as long as there was only one binary at the package root. Adding `cmd/vorch-service/` and `cmd/porch/` required renaming to a real module path FIRST. Plan to look for and rename any `module main` / `module foo` (no slash) declarations as the absolute first step of any multi-cmd split — these are silent timebombs.

### `ProtectHome=yes` blocks /root too, not just /home
I lost ~10 minutes thinking `sudo pipx install` would let me bypass ProtectHome by landing things under `/root/.local/`. ProtectHome=yes blocks BOTH `/home` AND `/root`. For root-owned ansible-runner installs that need to be visible to a hardened systemd unit, the only options are: (a) `ProtectHome=no` (what we chose), (b) install under `/opt` or `/usr/local` (requires a binary, not a script that imports from /root's venv), (c) BindReadOnlyPaths to punch a hole through.

### The healthprobe endpoint never matched the API — a hazy-kite plan was supposed to fix it
Vorch's healthprobe was PATCHing `/api/v1/admin/feature-flags/playbooks` with `{healthy, last_probed_at}`. The API had `/api/v1/FeatureFlags/{key}` accepting `{Enabled}` only. Hazy-kite was meant to wire the API side — that plan was either never completed or completed differently. PHASE 3 verification caught it and we shipped the fix as part of charmed-panda. Going forward: **a plan that builds on another plan's "scheduled" work should verify, not trust**. The hazy-kite scaffold was there (probe code already existed); the matching API was not.

### When you remove a service that's silently doing work, find ALL the things it was doing
The Python `cloud-manager-worker` was doing four things we cared about: (1) consuming playbook-runs, (2) minting per-run Vault tokens, (3) generating ansible inventory files, (4) writing back to `vm_playbook_assignments`. The PLAN identified (1), (2), and (4) — and we built or ported all three. (3) was missed because the inventory generation didn't leave a paper trail in the PLAN's source files (it was inside the Python `_LogPublisher` neighborhood, not labeled "inventory"). FIX-002 documents the gap and proposes the follow-up plan. Going forward: **before retiring a service, `strings <binary>` + `grep "ansible\|vault\|inventory\|host" /opt/<service>/` AND ask "what does this service do that nobody else does?"** That's a different question than "what queues does it consume?"

## Fix files written
- `FIX-001.md` — ansible-runner not on PATH; ProtectHome=yes blocked /home and /root venvs. Fixed by sudo pipx + symlink + Environment override + ProtectHome=no.
- `FIX-002.md` — ansible-runner invocation form (`run <workdir> --project-dir <workdir> -p site.yml --json`) + pre-existing inventory-generation gap (out of scope; `[[follow-up-porch-inventory-generation]]`).

## Deviations from plan
- **No `internal/amqp/` package** (PLAN Phase 0002): the AMQP connection bootstrap was already in each main.go, not in messagesub.go. Skipping the dedicated package since there was nothing to share.
- **No separate `/etc/cloud-manager/porch.env`** (PLAN Phase 0006): the existing project convention uses inline `Environment=` lines in the systemd unit + `config-server.yaml` for Vault/AMQP/DB. Followed the existing convention.
- **Assignment writeback in API, not porch** (PLAN Phase 0005): porch only has HTTP access to the API, no direct DB access. The cleanest place to write `vm_playbook_assignments.LastRun*` is `PlaybookRunTargetService.UpdateAsync` on the API side (same SaveChanges as the target terminal status). This pushes part of the work into the cloud-manager-api PR — acceptable.

## Follow-ups
- `[[follow-up-porch-inventory-generation]]` — generate `inventory/hosts` per-target with VM IP + user + ssh_key path. Likely requires a new GET `/api/v1/vm/{id}` field exposing the libvirt-known IP, or porch calls vorch via a new internal HTTP endpoint.
- `[[follow-up-ansible-runner-system-install]]` — package ansible-runner via apt or vendor under `/opt/` so we can drop `ProtectHome=no` and the pipx-symlink dance.
- `[[follow-up-materialize-into-project-subdir]]` — change `PlaybookMaterializationService` to write playbook + roles under `project/` instead of workdir root, so the `--project-dir` flag isn't needed and ansible-runner's defaults work.
- `[[follow-up-redactor-stderr]]` — porch's `redactingWriter` only wraps `os.Stdout` via `log.SetOutput`. Stderr from subprocess (ansible-runner --json) is captured via a separate file handle on `cmd.Stderr` and bypasses the scrubber. Audit needed.
- `[[follow-up-clean-up-stale-runs]]` — three Queued runs from earlier failed pg triggers (`run_T9742WY3BH`, `run_RCPQK77JHG`, `run_M233W3Y51N`, `run_87KST5NXPB`, `run_894HG5R1TF`, plus AJKH6HRY2T and 5FPJ81F7ZT from this plan's verification) sit in the DB. Mark Cancelled or delete during a maintenance window.
