# Test: End-to-end smoke

## Test ID
`0006-DISKSIZE-DISCO-TEST`

## Test Cases

### TC-001: VM provisions with the right disk size
- After create, `vm_get(disksize-test-1)` shows `diskSizeGb=10` and `orchestrationStatus=2`

### TC-002: disk actually 10 GB on the VM
```bash
# extract SSH key from Vault, ssh in
ssh cloudmanager@<vm-ip> 'lsblk -bno SIZE / | head -1'
```
**Pass**: returns a value ≈ 10737418240 (10 * 1024^3) ± reasonable margin.

### TC-003: filesystem grew to match
```bash
ssh cloudmanager@<vm-ip> "df -BG --output=size /"
```
**Pass**: shows ~10G (filesystem reserve takes a slice, so 9G is acceptable; <8G means resize2fs didn't fire).

### TC-004: postgres install fits
```bash
ssh cloudmanager@<vm-ip> 'sudo apt-get update && sudo apt-get install -y postgresql'
ssh cloudmanager@<vm-ip> 'systemctl is-active postgresql && df -h /'
```
**Pass**: install completes; postgresql active; root not 100%.

## Scoring
All four. On pass, check the box for Task 0006 — this is the last task; write the Result and Wishlist.

## On Pass
Check the final box in `Orchestrate/0001-DISKSIZE-DISCO-ORCH.md`, then write `Results/0001-DISKSIZE-DISCO-RESULT.md` and `SelfImprove/0001-DISKSIZE-DISCO-WISHLIST-001.md`. Done.
