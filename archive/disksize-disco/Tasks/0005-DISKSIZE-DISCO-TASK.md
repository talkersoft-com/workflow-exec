# Task: Web form field + types + api client

## Task ID
`0005-DISKSIZE-DISCO-TASK`

## Repo
`cloud-manager-web` (on branch `disksize-disco`)

## Objective
Add a Disk Size field to the Create VM page; plumb through the types and API client.

## Steps
1. `src/types/index.ts` — add `diskSizeGb: number;` to `VirtualMachine`.
2. `src/services/api.ts` — update `vms.create` to accept + forward `diskSizeGb`:
   ```ts
   create: (hostId, data: { ..., diskSizeGb: number }) => request(`/api/v1/virtualmachine/create/${hostId}`, {
     method: "POST",
     body: JSON.stringify({ vmi: data.imageId, virtualMachine: { name, description, memory, diskSizeGb: data.diskSizeGb } }),
   })
   ```
3. `src/pages/CreateVmPage.tsx` — add a Disk Size input near the existing Memory input:
   - Local state: `const [diskSizeGb, setDiskSizeGb] = useState(10);`
   - Form field: `<input type="number" min="5" max="500" value={diskSizeGb} onChange={...}/>` with a small "GB" label
   - On submit, pass `diskSizeGb` to `api.vms.create`
4. Match the existing Memory selector's styling so the form looks coherent. If there's a `MemorySelector` component (`src/components/MemorySelector/`), consider adding a `DiskSizeSelector` parallel to it, OR just inline the input — keep the existing convention.
5. Build + deploy: `npm run build` → `sudo python3 scripts/web/install-web-app.py` (run from cloud-manager-api dir).

## Acceptance Criteria
- Create VM page shows a Disk Size field with a sensible default (10 GB).
- Submitting creates a VM with the chosen size; backend persists it.
- min/max validation prevents nonsense values client-side (and server-side from Phase 2 catches it as backup).

## Test
`Test/0005-DISKSIZE-DISCO-TEST.md`
