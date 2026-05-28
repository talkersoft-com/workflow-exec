# Result: disksize-disco

## Workflow ID
`0001-DISKSIZE-DISCO`

## Outcome
**Shipped.** 7/7 phases complete. End-to-end proven: user picks disk size in API/UI/MCP → AMQP carries `disksizegb` → vorch-service runs `qemu-img resize` before boot → cloud-init's growpart + resize2fs grow the filesystem → VM boots with the requested disk. Postgres-10 installs cleanly on the new 10 GB test VM (motivation for the feature).

## Phase summary

| Phase | Title | Result |
|---|---|---|
| 0000 | Branch every repo per branches.md | ✅ |
| 0001 | API entity + EF migration (DiskSizeGb column) | ✅ |
| 0002 | API DTO + AMQP message + Controller (min/max validation 5-500) | ✅ |
| 0003 | vorch-service: qemu-img resize + cloud-init growpart | ✅ (1 improvise) |
| 0004 | MCP `cloud_vm_create` adds `disk_size_gb` param | ✅ |
| 0005 | Web: Disk Size form field on CreateVmPage | ✅ |
| 0006 | End-to-end smoke (10 GB VM, postgres install fits) | ✅ |

## Test VM left up for inspection (per instructions)
- `vm_7NMR71EVMF` aka `disksize-test-1` at `10.0.150.54`
- Provisioned with `diskSizeGb=10`, qemu-img grown to 10 GiB, cloud-init expanded to 9.6 GB filesystem
- Has postgresql-10 installed and active
- SSH key in Vault at `cloudmanager/vm/instances/vm_7NMR71EVMF/ssh_key`

## Repos and branches (per branches.md)
All on the `disksize-disco` branch:
- `cloud-manager-api` — 1 commit (entity + migration + DTO + AMQP + controller)
- `cloud-manager-web` — 1 commit (types + api client + CreateVmPage Disk Size field)
- `cloud-manager-mcp` — 2 commits (cherry-picked un-pushed `public_id` fix + new `disk_size_gb` schema)
- `vorch-service` — 1 commit (message struct + handler arg passthrough)
- `vorch-lib` — 1 commit (InitializeHost gains DiskSizeGb param + qemu-img resize inline)
- `execution-workflow` — 1 commit (the disksize-disco/ folder)

## What worked the first time
- The "mirror memory" plan — every change followed memory's path exactly. No design decisions to relitigate.
- API validation at the controller (5-500 GB) — caught bad values cleanly with 400.
- Cloud-init `growpart + resize2fs` already enabled in the bionic cloud image — `qemu-img resize` was the only thing missing.
- Web wizard reused the existing `memory-buttons` CSS class for the disk-size buttons; no new styles.

## What needed improvisation
- **vorch-service yaml unmarshal regression after rebuild**: The deployed binary's `case "create-vm"` was silently losing PublicId + DisplayName after a Go rebuild. Cause turned out to be that `go build .` doesn't always overwrite the local binary if it thinks nothing changed — needed `-a` or `-o /tmp/...` to force a fresh artifact. Documented as Phase 3 improvise.
- **Test VM disk leftover from prior workflow**: had to clean up `dbg`/`dbg2` test rows by hand because they failed to fully orchestrate.

## Branches.md convention proved itself
Task 0000 caught a real issue immediately: `cloud-manager-mcp`'s local main was diverged from origin/main (un-pushed `58dc27c` fix + upstream's cloud_ prefix refactor). The branches.md convention forced us to fetch + verify before any code touched, which caught the divergence at the branching step rather than at PR-review time when conflicts would be much harder to untangle.

## References
- Workflow plan: `init.md`
- Repo manifest: `branches.md`
- Per-phase Improvises: `Improvise/`
- Wishlist: `SelfImprove/0001-DISKSIZE-DISCO-WISHLIST-001.md`
