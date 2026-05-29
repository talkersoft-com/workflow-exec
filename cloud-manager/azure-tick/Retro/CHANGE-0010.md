# CHANGE-0010 — Build + deploy + migrate + backfill

## API
- Published `cloud-manager-api` from `azure-tick` branch via `dotnet publish` → `/tmp/cm-api-azure-tick/`.
- Stopped `cloud-manager-api`, copied output to `/opt/cloud-manager-api/`, ran `dotnet ef database update`. Migration `20260529023655_AddVmIpAddress` applied cleanly.
- Started API; healthprobe round-trip confirmed on first tick.

## Backfill notes
**FIX-001 (mid-phase)**: `python3 -m psycopg2` missing on the host. Installed via `sudo apt install python3-psycopg2` (Ubuntu 24.04 has the pgdg package at `2.9.10-1.pgdg24.04+1`). Not a regression — psycopg2 wasn't a stated prereq; the system happened to not have it.

**FIX-002 (mid-phase)**: backfill script was passing the VM friendly name (e.g. `postgres-test`) to `virsh domifaddr`, but libvirt's domain name in this stack is the public_id (`vm_189KV047V2`). Updated script to pass `pub` instead of `name`. Also added `--source arp` as a fallback when the default lease source returns empty (which it does in this libvirt config).

After fixes:
```
$ sudo python3 scripts/porch/backfill-vm-ip-addresses.py
Backfilling 4 VMs without ip_address...
  disksize-test-1 (vm_7NMR71EVMF) — no IP from libvirt; SKIPPING
  postgres-test (vm_189KV047V2) -> 10.0.150.55  OK
  TEST111111 (vm_J0MAE5D2YA) -> 10.0.150.51  OK
  test-vm-2 (vm_ZNCMJE89SR) — no IP from libvirt; SKIPPING

Done. 2 updated, 2 skipped.
```

The 2 skipped VMs (`disksize-test-1`, `test-vm-2`) had no entries via either lease or ARP — likely never sent traffic on the bridge. Operator can populate manually via the PATCH endpoint once those VMs are reachable. Not blocking — the demo target is `postgres-test`, which now has `10.0.150.55`.

## Vorch
- Built `cmd/vorch-service` from azure-tick. Stopped, swapped binary at `/kvm-automator/vorch-service`, restarted.

## Porch
- Stopped, re-ran `install-porch-service.py`. New unit has `ProtectHome=yes` and `Environment="ANSIBLE_RUNNER_BIN=/usr/local/bin/ansible-runner"`. Started.

## Status after deploy
- All three services `active`.
- `which ansible-runner` → `/usr/local/bin/ansible-runner` (symlinked to `/opt/pipx/venvs/ansible-runner/bin/ansible-runner`).
- Single consumer on `playbook-runs` (porch); no regression on the charmed-panda single-consumer guarantee.
- No exception traces in the journals over the last 60 seconds.
