# Lessons: cloud-manager/lilac-redpanda

## What would have helped

1. **systemd `ReadWritePaths` requires path existence.** The first vorch start after Phase 6 failed with `Failed to set up mount namespacing: /usr/share/ansible/collections: No such file or directory`. The plan called for widening the unit but didn't call out the pre-create step. Recommendation: any future plan that adds an entry to `ReadWritePaths` should pair it with an explicit `mkdir -p` step in the installer template.

2. **`apt install ansible-runner` does not install ansible-core for root by default on Ubuntu 24.04.** It pulled the runner via pipx-style isolation that didn't put `ansible-galaxy` on root's `$PATH`. The real fix was `apt install ansible` (which installs ansible-core system-wide at `/usr/bin/`). Recommendation: any future workflow that depends on a CLI vorch shells out to should include a `which <cli>` smoke check under `sudo` in the deploy verifier.

3. **systemd `ProtectHome=yes` breaks ansible-galaxy without `HOME` override.** ansible-galaxy assumes a writable `~/.ansible` for its temp dir. With `ProtectHome=yes` (correct hardening) and `PrivateTmp=yes`, setting `Environment=HOME=/tmp` is the minimal-surgery fix. Recommendation: keep this in the systemd-unit template forever; a comment explains why.

4. **`apt install ansible` bundles community collections under `/usr/lib/python3/dist-packages/`.** Without `--force`, `ansible-galaxy collection install … -p /usr/share/ansible/collections` short-circuits and reports the apt path. Took two flow runs to catch and one more to fix. Recommendation: when the worker-controlled install path matters (it does — Remove's safety guard depends on it), always pass `--force` so ansible-galaxy actually writes there.

5. **`ansible-galaxy collection list <ns>.<name> --format json` returns the parent of `ansible_collections/`, not the path including it.** The doubled `ansible_collections/ansible_collections/` segment in early install_path values came from misreading the docs. Fixed by trusting the JSON key directly. Recommendation: the parser uses a string match instead of any inferred concatenation.

6. **Verify-after-remove can't use `ansible-galaxy collection list` when apt-bundled copies exist.** It'll always report the collection as "still present" via the apt path. The right verifier is `os.Stat(install_path) == NotExist`. Recommendation: the workflow plan called for the list-based verify; future similar features should default to filesystem checks when the install path is controlled.

7. **"Install" UX on a Failed/Removed row should not require force=true.** The initial service throw made Failed rows un-installable without a `?force=true` query param the UI never sends. Fixed by allowing retry without force when status is Failed or Removed. Recommendation: spell out lifecycle transitions in the design doc — what UI affordance is shown for each status and what semantics the corresponding API call has.

## Fix files written
- `FIX-001` — pre-create `/usr/share/ansible/collections` before vorch start (systemd ReadWritePaths requirement)
- `FIX-002` — `apt install ansible` system-wide so root finds `ansible-galaxy`
- `FIX-003` — `Environment=HOME=/tmp` in the vorch systemd unit
- `FIX-004` — always pass `--force`; fix install_path parsing; verify-removal via filesystem absence

All four are in `Retro/`. Each carries the failure it solved + its outcome.

## Deviations from plan
- **Always `--force`** — plan only specified `--force` for Reinstall; lesson #4 above required it on every install. Cleaner than the originally-recommended split.
- **Verify-after-remove uses `os.Stat`, not `ansible-galaxy collection list`** — plan recommended the list call; switched to filesystem absence check per lesson #6.
- **Install accepts Failed/Removed state without force** — plan called for `?force=true` mandatory for any existing row; relaxed for Failed/Removed which is the operator's natural retry case (lesson #7).
- **`HOME=/tmp` env added to systemd unit** — not in the original plan, required for ansible-galaxy under `ProtectHome=yes`.
