# Workflow: disksize-disco

## What this is
Add a **user-specifiable disk size** field to VM creation in Cloud Manager. Today the user picks memory at create time; disk size is baked into the image and the resulting VM gets stuck with whatever the image template provides (currently 2.4 GB — too small for real workloads like postgres).

End state: user enters a disk size in GB on the Create VM page → backend stores it → vorch-service resizes the qcow2 + grows the partition during provision → the new VM boots with the requested disk.

## Inputs to read before any task
- `branches.md` — every repo this workflow modifies; **branch all of them before Task 0001**
- `Orchestrate/0001-DISKSIZE-DISCO-ORCH.md` — the master orchestration with phases + `/loop` directive
- `../instructions.md` — workflow conventions
- `../craft-ingot/` — canonical example for every file shape

## Pattern to mirror
The existing `memory` field is plumbed end-to-end through the same surfaces this feature needs. **Copy memory's shape exactly** — the architecture is already proven, deviation is risk.

Memory's path (for reference):
- Entity: `VirtualMachine.Memory` (int, MB)
- DTO + AMQP message: `Memory` int
- Controller: maps request → DTO → AMQP message
- vorch-service Go handler: passes `Memory` to libvirt provisioning
- Web: form field with default 2048
- MCP: `vm_create` tool param

## Naming choices we've already made
- **Field name**: `DiskSizeGb` (C#) / `diskSizeGb` (TS/JSON) / `disk_size_gb` (db, yaml)
- **Units**: gigabytes (operator-friendly, no MB ↔ GB conversion confusion)
- **Default**: the source image's native size (don't shrink; only allow growing)
- **Lower bound**: image size (you can't shrink an already-built image cleanly without re-mounting)
- **Upper bound**: pick something conservative (e.g. 500 GB) to avoid fat-finger 5000 → host runs out of disk

## Out of scope (do NOT do this in this workflow)
- Resizing existing VMs after creation — additional feature, deserves its own workflow
- Per-host disk quota enforcement — needs a host capacity model that doesn't exist yet
- Anything involving thin provisioning / LVM / ZFS

## Test fixture
- Use MCP `vm_create` to create `disksize-test-1` with `disk_size_gb: 10` on the existing host
- Verify with SSH: `lsblk` shows ~10 GB on the root device, `df -h /` shows the grown filesystem
- Leave the test VM up at end-of-workflow per the operator's standing instruction
