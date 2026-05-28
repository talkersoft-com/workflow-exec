# Wishlist: disksize-disco (workflow-level reflection)

## Scope
Cross-phase reflection. The workflow shipped in ~7 phases; this is the friction worth flagging.

## Top three (most-leverage)

### 1. Trust `go build` less; verify the binary
Phase 3's biggest time sink was a stale binary problem. `go build .` produced a binary that LOOKED fresh (timestamps matched, the build command exited 0) but the deployed artifact didn't reflect my source changes. Without `strings` to grep for debug markers, I would've kept "fixing" the wrong code. A tiny "binary identity" check (`go build -o /tmp/build-$$ . && md5sum /tmp/build-$$ /deployed/binary && diff`) at the end of every deploy would have caught this immediately.

### 2. `branches.md` paid off immediately — keep the convention
Task 0000 caught divergent local main on cloud-manager-mcp before any code touched. The cherry-pick to preserve the un-pushed `public_id` fix took 60 seconds with the right awareness; the same divergence would have caused a multi-way merge conflict at PR-review time. **The convention works; don't drop it.**

### 3. Surface "where does this field also live?" at design time
Every property that flows web → API → AMQP → vorch-service → vorch-lib touches 6-11 files across 4-5 repos. A static analysis or doc generator that produces the "memory's plumbing punch list" output (the agent-trace report I used for this workflow) on demand would have collapsed Phase 2-5 planning to a glance. Even a `make trace-field memory` Makefile target would be useful.

## Other things that came up

### vorch-service has zero structured logging
Service logs go to `/kvm-automator/provision.log` (file) AND systemd journal — but the file gets all the application output and the journal only gets startup/lifecycle. This split makes debugging painful (you have to know to `tail -f /kvm-automator/provision.log`, not `journalctl -u vorch_service -f`). Could be: route stdout to journal consistently, drop the file.

### Empty publicid bug was scary because it was silent
yaml.v2 unmarshalling silently dropped PublicId/DisplayName when the message struct was somehow misconfigured (root cause: stale binary, not the struct). A debug print confirmed the live state was actually right. **Wishlist:** any unmarshal that produces an obviously-empty required field should warn or fail-fast in vorch-service.

### vorch-lib bumps need a vorch-cli check
I modified `InitializeHost` signature in vorch-lib — broke vorch-bootstrap's caller that uses an older signature (which already didn't compile, but if it did this would silently break it). When publishing a new vorch-lib tag, ALL callers in the workspace should re-build clean. Could be a single CI check.

### Test VM disk choice should be a project default
The prior workflow (ansible-shenanigans) failed Phase 0010 because the test VM was 2.4 GB. The new default of 10 GB (which this workflow shipped) is a sane minimum for any "test workload that does real install work". Consider making the API's default 10 GB instead of relying on the column default + controller fallback.

## Things to copy to the next workflow
- The branches.md table format is solid; copy verbatim
- "Mirror an existing field's plumbing" is the simplest mental model for an end-to-end feature add — write the punch list of files-to-touch in the init.md as part of planning
- Task 0000 as the always-first task that says "branch, don't touch code"

## Things that were just fine
- Mirroring the `memory` field's path made every decision automatic
- EF Core migration with `defaultValue: 10` auto-backfilled existing rows without explicit SQL
- AutoMapper picked up the new field with no profile changes (identical name + type)
- The systemd unit's `ReadWritePaths` was already broad enough; no sandbox tweaks needed for this feature
