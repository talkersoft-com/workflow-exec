# Task: End-to-end smoke

## Task ID
`0006-DISKSIZE-DISCO-TASK`

## Objective
Prove the whole feature works: create a VM via the UI (or MCP) with `disk_size_gb=10`, watch it provision, SSH in, confirm the disk is actually 10 GB. Leave the VM up for operator inspection.

## Steps
1. Via web UI OR `vm_create` MCP: create `disksize-test-1` with `disk_size_gb=10`, memory=2048, on the existing host.
2. Wait for `orchestrationStatus=2`, `machineStatus=0` (Running).
3. SSH in (using the VM's vault-stored SSH key like the ansible-shenanigans workflow did):
   ```bash
   lsblk
   df -h /
   ```
4. Confirm the root filesystem is ~10 GB (within margin for filesystem overhead).
5. Bonus: install something that previously wouldn't fit on the 2.4 GB test VM — postgres-16 (the workload that motivated this feature). Should fit easily in 10 GB.

## Acceptance Criteria
- New VM provisions cleanly with `disk_size_gb=10`.
- `df -h /` on the VM shows ~9.5G+ available (filesystem reserves a small percentage).
- Postgres-16 installs without disk pressure errors.

## Test
`Test/0006-DISKSIZE-DISCO-TEST.md`

## Cleanup
Do NOT destroy `disksize-test-1`. Operator will inspect via the dashboard.
