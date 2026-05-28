# Test: vorch-service handler

## Test ID
`0003-DISKSIZE-DISCO-TEST`

## Test Cases

### TC-001: qemu-img resize fired
- Provision a VM with `disk_size_gb: 10`
- Immediately after provision (BEFORE first boot completes): on the libvirt host, `qemu-img info <vm-disk-path>` shows `virtual size: 10 GiB`
- **Pass**: 10 GiB
- **Fail**: original image size → handler skipped resize step

### TC-002: partition grew on boot
- Wait for VM to boot + cloud-init complete (~30-60s)
- SSH in: `lsblk` shows a ~10G root partition
- `df -h /` shows filesystem near 10G
- **Pass**: both
- **Fail**: lsblk shows 10G but df shows original — cloud-init growpart fired but resize2fs/xfs_growfs didn't

### TC-003: AMQP message contains the field
- Tail the AMQP queue OR vorch-service logs during provision; confirm the decoded message struct has the disk_size_gb value
- **Pass**: visible in logs
- **Fail**: field never made it across — check yaml tag name + struct visibility

## Scoring
All three. On pass, check the box for Task 0003.
