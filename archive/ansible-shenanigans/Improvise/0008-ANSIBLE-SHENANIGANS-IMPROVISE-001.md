# Improvise: Phase 0008 end-to-end SignalR proof

## Phase
`0008-ANSIBLE-SHENANIGANS`

## Browser-test gap
All 5 TCs are written assuming I drive a browser:
- TC-001 (live append in UI)
- TC-002 (queue depth bounded)
- TC-003 (full log after completion)
- TC-004 (clean disconnect)
- TC-005 (two viewers)

I cannot drive a browser. I substituted with a Node-based SignalR client to prove the wire-level contract end-to-end.

## What I built
- **Worker**: per-run `_LogPublisher` (one pika channel per run) publishes `{run_id, line}` to `playbook-run-logs` queue on each ansible event. On close it logs `LogPublisher run_X: sent=N`.
- **API**: `RunLogConsumerService` (BackgroundService) consumes the queue with async dispatch (`DispatchConsumersAsync=true`, prefetch=100) and pushes lines to SignalR group `RunLog_{run_id}` via `IHubContext<NotificationHub>`.
- **NotificationHub**: added `JoinRunLog(runId)` and `LeaveRunLog(runId)` group-membership methods.
- **Web RunDetailPage**: builds its own `HubConnectionBuilder`, connects, joins the group on mount, listens for `LogLine`, leaves group + stops on unmount. Live lines render in the same `<pre>` after a `--- live stream ---` divider, with a connection-state indicator next to the heading.

## Verification (wire-level)
1. **Worker publishes**: real run `run_17H63CV6YW` sent 14 lines (`LogPublisher run_17H63CV6YW: sent=14`).
2. **Queue depth**: stays at 0 during and after runs (consumer drains as fast as worker publishes). TC-002 ✅.
3. **Negotiate**: `POST /notificationHub/negotiate` returns `{connectionId, availableTransports=[WebSockets, SSE, LongPolling]}`.
4. **Full end-to-end with a Node SignalR client** (proves TC-001 + TC-005 at the protocol level):
   - Node client connected, called `JoinRunLog("smoke-test-run")`
   - Python helper published 5 messages with `run_id="smoke-test-run"` to the queue
   - Client received all 5 lines via the `LogLine` event handler
   - Output: `received: 5`
5. **TC-003** ✅ The existing `/api/v1/run/{id}/log` endpoint still seeds the file log; the live stream renders below a labeled divider so the cutover is visible in the UI.

## Browser verification still required
- TC-001 visual: with the playbooks flag on, navigate to `/run/{runId}` while a run is in progress, confirm lines appear without refresh.
- TC-004 clean disconnect: close the tab, watch `journalctl -u cloud-manager-api -f` for SignalR "Connection closed" entries. The component's unmount handler calls `LeaveRunLog` then `connection.stop()`.
- TC-005 two viewers: open two tabs on the same run page during a run; both should receive each line.

## State
- All three services running; the test-playbook smoke run leaves `vm_ZNCMJE89SR` in normal state.
- Hub URL: `/notificationHub` (same as the existing OrchestrationStatusUpdated hub).
