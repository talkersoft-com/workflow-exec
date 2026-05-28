# Task: SignalR run log streaming

## Task ID
`0008-ANSIBLE-SHENANIGANS-TASK`

## Parent Orchestration
`Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md`

## Status
- [ ] In progress
- [ ] Complete
- [ ] Failed

## Objective
Open the run page during a live run → stdout streams in near-realtime.

## Steps

1. **Worker → RabbitMQ**: in `RunHandler`, the ansible-runner `event_handler` callback (called per line/event) publishes `{run_id, line}` to a queue `playbook-run-logs`. Also writes to the on-disk log (unchanged from Phase 0006). Use a single long-lived `pika` channel, batched if perf becomes an issue.

2. **API consumer**: extend `ConsumerService` or add `RunLogConsumerService` (cleaner). Listens to `playbook-run-logs`. For each message: `_hubContext.Clients.Group($"RunLog_{run_id}").SendAsync("LogLine", line)`.

3. **NotificationHub**: add `JoinRunLog(string runId) → Groups.AddToGroupAsync(Context.ConnectionId, $"RunLog_{runId}")` and `LeaveRunLog(string runId) → ...`. Authenticated when auth lands; open for now.

4. **Web RunPage**:
   - On mount: `connection.invoke("JoinRunLog", runId)`, register `connection.on("LogLine", line => append)`.
   - On unmount: `LeaveRunLog`. Disconnect cleanly.
   - Render log lines in an `<pre>` with monospace font and auto-scroll to bottom (with a "follow" toggle so user can scroll up without it jumping back).

5. **Backfill**: when the page opens, first fetch the full log via `GET /run/{id}/log` to seed the buffer. Then start subscribing. Tag the "live stream starts here" boundary in the UI so the user knows where the cutover is.

## Acceptance Criteria
- During an in-progress run, opening the page shows lines appearing within ~500ms of the worker producing them.
- After completion, refreshing the page shows the full log from the file (no missing lines around the live/persisted boundary).
- Disconnecting the browser → no leaked SignalR connections after 30s (`signalr` connection count goes back down).

## Test
`Test/0008-ANSIBLE-SHENANIGANS-TEST.md`
