# Task: API DTO + AMQP message + Controller

## Task ID
`0002-DISKSIZE-DISCO-TASK`

## Repo
`cloud-manager-api` (on branch `disksize-disco`)

## Objective
Plumb `disk_size_gb` through the API: DTO ↔ entity (AutoMapper) + AMQP outbound message + Controller request → DTO mapping. Validate min/max in the controller.

## Pattern
Mirror `Memory` in every file. Same shape, same conventions.

## Steps
1. `src/Models/CloudManager.DTO/Models/VirtualMachine.cs`: add `public int DiskSizeGb { get; set; }`.
2. `src/Models/CloudManager.Messages/VmCommandData.cs`: add `public int DiskSizeGb { get; set; }`.
3. AutoMapper profile: confirm Memory maps automatically (same name + type). DiskSizeGb should also map automatically — no explicit profile change needed.
4. `src/CloudManager.API/Controllers/VirtualMachineController.cs` Create endpoint:
   - Read from request: `DiskSizeGb = request.VirtualMachine.DiskSizeGb`
   - Pass to AMQP message: `DiskSizeGb = virtualMachine.DiskSizeGb`
   - Validate at the controller boundary (these are the only place this lives):
     ```csharp
     const int MIN_GB = 5;
     const int MAX_GB = 500;
     if (request.VirtualMachine.DiskSizeGb < MIN_GB || request.VirtualMachine.DiskSizeGb > MAX_GB)
         return BadRequest(new { message = $"disk_size_gb must be {MIN_GB}-{MAX_GB}" });
     ```
5. Rebuild + redeploy API: `sudo python3 scripts/api/install-api-service.py`

## Acceptance Criteria
- `POST /api/v1/virtualmachine/create/{hostId}` with a `diskSizeGb` body field returns 200/202 and persists the value.
- Same call with `diskSizeGb: 1` or `diskSizeGb: 9000` returns 400 with the validation message.
- AMQP message received by vorch-service contains `disk_size_gb` (verify in Phase 3).

## Test
`Test/0002-DISKSIZE-DISCO-TEST.md`

## On Failure
- AutoMapper missing field: add explicit `.ForMember(d => d.DiskSizeGb, ...)` if needed.
- Validation message not surfacing: confirm the controller returns BEFORE mapping/persisting.
