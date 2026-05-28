# Improvise: Phase 3 stale-binary debugging

## Phase
`0003-DISKSIZE-DISCO`

## What happened
After deploying the new vorch-service binary with the qemu-img resize logic, every create-vm started failing with an empty PublicId — `Creating VM  with OS=linux...` (note empty space). The very SAME source code had worked with a debug-enabled build minutes earlier.

## Root cause
`go build .` in the vorch-service module wasn't always overwriting the local `./vorch-service` binary file. The build exited 0 but the on-disk binary remained from an earlier compile. When I `cp`'d that binary to `/kvm-automator/vorch-service`, I was deploying the wrong artifact.

Confirmed via md5sum: deployed binary at `/kvm-automator/vorch-service` had a different md5 than `/tmp/vorch-service-fresh` (which I built with `go build -o /tmp/...`).

## Fix
Force a fresh build to a known-clean path:
```bash
go build -o /tmp/vorch-service-fresh .
md5sum /tmp/vorch-service-fresh /kvm-automator/vorch-service  # verify they differ
sudo cp /tmp/vorch-service-fresh /kvm-automator/vorch-service
```

Or use `go build -a` (force rebuild everything).

## Verification after fix
- `disksize-test-1` (vm_7NMR71EVMF) provisioned with `disksizegb=10`
- `qemu-img info` showed virtual size 10 GiB
- `lsblk` showed 10G `vda`, partition expanded to 9.9G
- `df -h /` showed 9.6G filesystem with 8.4G available
- postgres-10 installed cleanly with no disk pressure

## Followup (also logged in Wishlist)
Treat `go build` output with skepticism in deployment scripts. Always verify the deployed binary's identity via md5 OR explicitly build to a tempfile + diff against the deployed copy. A `make deploy` target that does this should replace the manual cp dance.
