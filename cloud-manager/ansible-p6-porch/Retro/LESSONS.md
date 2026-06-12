# Lessons — cloud-manager/ansible-p6-porch

Four FIX files this run: FIX-001 (EF design-time factory targets live DB), FIX-002
(`--cmdline=--check` single-token argparse), FIX-003 (build-app.sh vs cmd/ layout), FIX-004
(deploy script ssh-to-self). Details in each file; what would have helped:

1. **`dotnet ef` in cloud-manager-api ALWAYS hits live clouddb unless `--connection` is
   passed.** `CloudManagerDbContextFactory` hardcodes the live connection and ignores the
   runtime env vars (`DBHOST`/`DATABASE`/…). Every scratch/test apply must pass
   `--connection` explicitly; only the Task-0006 intentional live apply may use the default.
   A guardrail (e.g. factory refusing `clouddb` without an explicit opt-in env) would remove
   this whole failure class.

2. **Forwarding `--`-prefixed values to argparse CLIs needs `--opt=value` form.** The plan's
   `--cmdline "--check"` is shell-quoting notation, not argv structure; passing it as two exec
   argv elements makes argparse report "expected one argument". Caught only by the real-VM
   e2e — unit tests on argv shape alone would have blessed the broken form.

3. **Deploy fragments rot when nothing exercises them.** `build-app.sh` still built the
   pre-split module root (no Go files) and `deploy-app.sh` assumes a remote workstation
   (scp/ssh to self fails on the server itself). `go build ./...` in CI proves the code
   builds, not that the deploy path works. Either run the deploy script in CI against a
   throwaway dir or keep a one-line smoke (`build-app.sh && ls bin/...`) in the task list.

4. **Shared-RabbitMQ testing needs the live consumer stopped first.** Local API + local porch
   against a scratch DB works well, but live porch on the same queues would consume test
   messages and nack-loop on runs that only exist in the scratch DB (porch 404-nacks with
   requeue=true). Stop porch.service for the window, purge test queues, restart, verify
   `consumers: 1` and empty queues after. A future improvement: porch could ack-and-drop on
   run-fetch 404 (the run row is authoritative; a 404 run can never become processable).

5. **`vm_events` needed a nullable VM for lint events.** Contracts §6 placed
   `playbook_lint_completed` in `vm_events`, but lint runs have no VM context. Relaxing
   `virtual_machine_id` to nullable (additive, timelines unaffected) plus a
   `RecordGlobalEventAsync` was the smallest contract-conforming change; worth stating in the
   contracts file next time an event type has no natural aggregate row.

6. **Stale Queued runs are not in-flight work.** The restart-caution check found 3 Queued
   runs days old with empty queues — orphans from before, not active work. "Queued + empty
   queue + old started_at" is the signature; only Running rows (or non-empty queues) should
   block a restart.
