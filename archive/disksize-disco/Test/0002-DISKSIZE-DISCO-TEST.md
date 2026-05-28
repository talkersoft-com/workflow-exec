# Test: API DTO + AMQP + Controller

## Test ID
`0002-DISKSIZE-DISCO-TEST`

## Test Cases

### TC-001: valid disk_size_gb persists
- POST `/api/v1/virtualmachine/create/{hostId}` with `{ vmi, virtualMachine: { name, memory, diskSizeGb: 10 } }` against a real host
- Expect 200/202; query DB: `SELECT disk_size_gb FROM vm.virtual_machines WHERE name=...` → `10`
- Cleanup: orchestration_destroy the VM (or leave for the end-to-end smoke in Phase 6)

### TC-002: too small returns 400
- POST with `diskSizeGb: 1` → expect HTTP 400 with message containing `5-500`

### TC-003: too large returns 400
- POST with `diskSizeGb: 9000` → expect HTTP 400

### TC-004: missing field defaults via DB column default
- POST without `diskSizeGb` → behavior is "default 10 from DB" OR "controller rejects missing field" — pick one and assert it consistently. Recommend defaulting (operator can omit if they don't care).

## Scoring
All four. On pass, check the box for Task 0002.
