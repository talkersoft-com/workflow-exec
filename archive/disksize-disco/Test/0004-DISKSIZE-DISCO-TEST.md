# Test: MCP vm_create

## Test ID
`0004-DISKSIZE-DISCO-TEST`

## Test Cases

### TC-001: schema shows the new param
- Inspect the MCP tool definitions (the operator's MCP client shows them, OR via the MCP server's discovery endpoint)
- **Pass**: `vm_create` advertises `disk_size_gb: number (5-500, default 10)`

### TC-002: end-to-end via MCP
- `vm_create(host_X, "disksize-mcp-test", 2048, vmi_Y, disk_size_gb=10)`
- DB shows the new VM with `disk_size_gb=10`
- VM boots with the 10G disk (validates Phase 3 too)
- Cleanup: orchestration_destroy when done (or leave for Phase 6 if it's the smoke VM)

### TC-003: default applied on omit
- `vm_create(...)` without `disk_size_gb`
- DB shows `disk_size_gb=10` (the schema default)

## Scoring
All three. On pass, check the box for Task 0004.
