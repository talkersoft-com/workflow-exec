# Test: SignalR run log streaming

## Test ID
`0008-ANSIBLE-SHENANIGANS-TEST`

## Task Reference
`Tasks/0008-ANSIBLE-SHENANIGANS-TASK.md`

## When To Run
After API + worker + web redeploy.

## Test Cases

### TC-001: live append
- **Setup**: trigger an Apply on the postgres playbook; immediately open the run page in the browser
- **Check**: new lines appear in the log pane without manual refresh
- **Pass**: ≥5 lines visible within 30s
- **Fail**: blank page or all-at-end → SignalR connection not established (check browser console)

### TC-002: queue depth stays bounded
- **Check during run**: `rabbitmqctl list_queues | grep playbook-run-logs`
- **Pass**: depth fluctuates but doesn't grow unboundedly; ≤500 unprocessed at any sample
- **Fail**: monotonically growing → consumer slower than producer or blocked

### TC-003: full log after completion
- **Check post-run**: refresh the run page
- **Pass**: full log present; no missing chunks; PLAY RECAP at the end matches the file
- **Fail**: gaps near the live/persisted boundary — seed-then-subscribe ordering issue

### TC-004: clean disconnect
- **Check**: close the run page; wait 30s; check SignalR connection count via the hub dashboard or `journalctl` for "Connection closed"
- **Pass**: connection drops; no zombie group memberships
- **Fail**: connections leak — investigate `OnDisconnectedAsync` cleanup

### TC-005: two viewers see same stream
- **Check**: two browser tabs on the same run page during a run
- **Pass**: both receive each line; ordering identical
- **Fail**: divergence → group send not idempotent across consumers

## Scoring
All five. Tab-close discipline matters; don't leave SignalR connections open between phases.

## On Pass
Check the box for this task in `Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md` and continue to the next unchecked task. Workflow-level Result + Wishlist are written once at the very end, after every box is checked.
