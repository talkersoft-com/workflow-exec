# Test: Web form

## Test ID
`0005-DISKSIZE-DISCO-TEST`

## Test Cases

### TC-001: bundle includes the field reference
```bash
grep -o "diskSizeGb\|Disk Size" /opt/cloud-manager-web/assets/index-*.js | sort -u
```
**Pass**: both strings present.

### TC-002: page renders the field (visual — operator)
- Open `/create` in the browser
- **Pass**: Disk Size input visible near Memory, default 10, can type a different value
- **Fail**: field absent or misplaced

### TC-003: submit creates a VM with the chosen size
- Fill the form with `name=disksize-web-test`, `diskSizeGb=10`, submit
- DB: `SELECT disk_size_gb FROM vm.virtual_machines WHERE name='disksize-web-test'` → `10`

### TC-004: client validation rejects bad values
- Try to set the input to `1` or `9000` → field should reject or button should disable

## Scoring
All four. On pass, check the box for Task 0005.
