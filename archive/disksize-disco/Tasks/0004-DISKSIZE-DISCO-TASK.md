# Task: MCP vm_create param

## Task ID
`0004-DISKSIZE-DISCO-TASK`

## Repo
`cloud-manager-mcp` (on branch `disksize-disco`)

## Objective
The `vm_create` MCP tool accepts `disk_size_gb` and forwards it to the API.

## Steps
1. `src/tools/vms.ts` — find the `vm_create` zod schema (line ~38 per the memory trace). Add:
   ```ts
   disk_size_gb: z.number().int().min(5).max(500)
     .default(10)
     .describe("Root disk size in GB (5-500)")
   ```
2. In the handler body, include `diskSizeGb: args.disk_size_gb` in the POST body's `virtualMachine` object.
3. Rebuild: `npm run build` in the mcp dir.
4. Restart the MCP server (operator may need to reconnect — note this in the Test file).

## Acceptance Criteria
- Calling `vm_create(... disk_size_gb=10)` via MCP creates a VM with `disk_size_gb=10` in the DB.
- Omitting the param uses the default `10`.

## Test
`Test/0004-DISKSIZE-DISCO-TEST.md`
