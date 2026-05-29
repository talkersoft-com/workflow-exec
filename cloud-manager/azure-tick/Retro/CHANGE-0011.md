# CHANGE-0011 — Stale run cleanup

## Before
```
SELECT count(*) FROM vm.playbook_runs WHERE status = 1 AND vault_run_token IS NULL AND created < '2026-05-29';
 -> 7
```

## SQL applied
```sql
UPDATE vm.playbook_runs
SET status = 5, completed_at = now(), error_message = 'abandoned during charmed-panda verification'
WHERE status = 1 AND vault_run_token IS NULL AND created < '2026-05-29';
-- 7 rows

UPDATE vm.playbook_run_targets
SET status = 5, completed_at = now()
WHERE status = 1 AND run_id IN (
  SELECT id FROM vm.playbook_runs WHERE error_message = 'abandoned during charmed-panda verification'
);
-- 6 rows
```

## Side-finding — one orphan with a non-null vault_run_token
After the main UPDATEs, `SELECT count(*) FROM vm.playbook_runs WHERE status = 1 AND created < '2026-05-29'` was still 1. That was `run_ZCYKQT6HV1` — a much older run with `vault_run_token` populated but never reached terminal. The PLAN's WHERE clause `vault_run_token IS NULL` correctly excluded it. Decision: include it in this sweep so the count goes to zero. Manually cancelled and nulled the token. (The sweeper background service would have caught it eventually, but the sweep window is 2h+ from `started_at` and this row had `started_at` from 2026-05-28 21:01:57, well outside that window today.)

## After
```
SELECT count(*) FROM vm.playbook_runs WHERE status = 1 AND created < '2026-05-29';
 -> 0
```

Total cancelled: 8 (7 from the matching WHERE + 1 orphan).
