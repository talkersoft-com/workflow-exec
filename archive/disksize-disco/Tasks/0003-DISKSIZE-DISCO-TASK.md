# Task: vorch-service handler (qemu-img resize + cloud-init growpart)

## Task ID
`0003-DISKSIZE-DISCO-TASK`

## Repo
`vorch-service` (on branch `disksize-disco`)

## Objective
On VM provision, vorch-service receives `disk_size_gb` in the AMQP message, runs `qemu-img resize <disk> {N}G` against the cloned VM image BEFORE first boot, and ensures the partition grows on first boot.

## Pattern
Find where `Memory` is consumed in the handler (`vorch-service/handlers/vm_handlers.go:46` per the trace from init.md). Mirror its shape.

## Steps
1. Add `DiskSizeGb int \`yaml:"disk_size_gb"\`` to `models/command_messages.go` (the struct that decodes the AMQP YAML).
2. In `handlers/vm_handlers.go`, after the image clone but BEFORE `virsh start`:
   ```go
   if msg.VM.DiskSizeGb > 0 {
       cmd := exec.Command("qemu-img", "resize", clonedDiskPath, fmt.Sprintf("%dG", msg.VM.DiskSizeGb))
       if out, err := cmd.CombinedOutput(); err != nil {
           return fmt.Errorf("qemu-img resize failed: %s", out)
       }
   }
   ```
3. The image's cloud-init / cloud-config must already invoke `growpart` + `resize2fs` (or xfs_growfs) on first boot. Verify this with one cloud-init snippet check on the template image. If the image doesn't do this, add a `growpart` directive to the cloud-init userdata vorch-service injects (look for existing userdata templating; that's where to extend).
4. Rebuild + redeploy vorch-service per its install scripts.

## Acceptance Criteria
- Provisioning a VM with `disk_size_gb: 10` results in `qemu-img info` showing a 10G disk after clone, BEFORE boot.
- After boot, SSH in: `lsblk` shows ~10G on the root device; `df -h /` shows the grown filesystem.

## Test
`Test/0003-DISKSIZE-DISCO-TEST.md`

## On Failure
- qemu-img resize works but the partition doesn't grow: cloud-init growpart isn't enabled. Confirm `growpart: {mode: auto, devices: ['/']}` is in the cloud-init userdata, or add it.
- Image is qcow2 with snapshots: resize warns about snapshot-locking. Solve by flattening before resize, OR fail-fast in the handler with a useful error.
- Resize requested < current image size: don't shrink. Reject in the handler or in the API controller (preferred: API controller already validates, so the handler can trust).
