# Execution Workflow — Pseudocode

High-level orchestration pattern for every workflow in this directory.
MCP tool calls are shown as `CALL tool_name(params)`.
Read this alongside `instructions.md` before starting any workflow.

---

```
WORKFLOW_START
  ├── read: init.md
  ├── read: deck.md
  ├── read: Orchestrate/NNNN-*-ORCH.md
  └── read: any referenced design docs (DATA-MODEL.md, etc.)


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE 0 — INITIALIZE WORKSPACE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  CALL hv_status(deck)
    → inspect: is every repo clean? which branch is each on?

  IF all repos clean AND on default branch (main):
    CALL hv_init(deck)
      → provisions missing repos
      → creates feature branch across ALL repos (name auto-generated)

  ELIF all repos clean AND on a feature branch AND PRs are merged:
    CALL hv_next(deck)
      → fetches origin/main into every repo
      → creates new feature branch across ALL repos (name auto-generated)

  ELIF any repo is dirty:
    STOP — investigate dirty repo before proceeding
    IF dirty repo has a merged PR and uncommitted changes (deadlock):
      CALL hv_stash(deck)
      CALL hv_next(deck)
      CALL hv_unstash(deck)

  CALL hv_status(deck)
    → confirm: every repo is on the new feature branch, all clean

  record generated branch name → write to deck.md
  check box: Task 0000 in orchestration file


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE N — WORK LOOP  (repeat for each unchecked task)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  FOR each unchecked task in Orchestrate/NNNN-*-ORCH.md:

    read: Tasks/NNNN-*-TASK.md

    implement changes in the target repo(s)
      (agent edits files directly — workspace is already on feature branch)

    run: Test/NNNN-*-TEST.md
      → execute each TC-* check

      IF test passes:
        check box in orchestration file
        CONTINUE to next task

      IF test fails:
        write: Improvise/NNNN-*-IMPROVISE-NNN.md
          → document: which TC failed, root cause, fix approach
        apply fix
        re-run test
        IF still failing: STOP and surface to operator


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE N+1 — DATABASE MIGRATION  (if workflow touches DB schema)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  CALL db_test_connection(workspaceRoot, dbKeyIdentifier: "clouddb_local")
    → confirm local DB is reachable

  run: migration generation script (e.g. add-migration.py <MigrationName>)
    → review generated Up() / Down() before applying

  run: deploy script against local DB
    → verify: new tables / columns exist

  CALL db_test_connection(workspaceRoot, dbKeyIdentifier: "clouddb")
    → confirm remote DB reachable (SSH tunnel must be active)

  run: deploy script against remote DB
    → verify: schema matches local

  CALL db_get_table_schema(workspaceRoot, dbKeyIdentifier, targetDatabase, schema, tables)
    → spot-check key tables for expected columns


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE N+2 — WRITE RESULT + SELFIMPROVE  ← BEFORE shipping
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  ⚠️  Write these files BEFORE calling hv_ship.
      hv_ship commits everything it finds — if these files are written
      after the ship call they will be on the next branch, not in the PR.

  write: Results/NNNN-*-RESULT.md
    ├── outcome (SHIPPED / PARTIAL / FAILED)
    ├── phase-by-phase summary
    ├── generated branch name
    ├── PR URLs → leave as "TBD — filled from hv_ship output"
    └── DB verification result (already done in prior phase)

  write: SelfImprove/NNNN-*-WISHLIST-NNN.md
    └── what tooling / access / docs would have cut wall-clock time


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE N+3 — SHIP
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  verify build is clean before shipping
    (dotnet build / npm run build / go build — 0 errors)

  CALL hv_ship(deck, message, title, body)
    → per repo with changes:
        uncommitted changes → git add -A + commit (includes Result + Wishlist)
        committed + unpushed → push only
        nothing to do → skip
    → opens PR for every repo ahead of origin/default
    → auto-merges if auto_merge: true in config
    → output contains the PR URLs — record them

  CALL hv_list_pulls(deck)
    → confirm PRs are open (or merged if auto_merge is on)

  CALL hv_status(deck)
    → confirm all repos clean, all commits pushed


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE N+4 — VERIFY DEPLOYMENT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  FOR each verification target (local DB, remote DB, running service, etc.):

    CALL db_execute_sql_query(...)   ← confirm schema / data
    OR
    probe the live service endpoint  ← confirm it responds as expected

  IF verification fails:
    investigate: was the migration applied? did the deploy script run?
    re-apply if needed — do NOT call hv_ship again for this


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
WORKFLOW_COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  CALL hv_status(deck)
    → final confirmation: workspace is clean, ready for next workflow
```

---

## MCP Tool Reference

| Phase | Tool | Purpose |
|---|---|---|
| Initialize | `hv_status(deck)` | Check workspace state before any action |
| Initialize | `hv_init(deck)` | Provision repos + create feature branch |
| Initialize | `hv_next(deck)` | Transition to new branch after prior ship |
| Initialize | `hv_stash(deck)` | Escape deadlock: stash merged-but-dirty repos |
| Initialize | `hv_unstash(deck)` | Restore stashed changes onto new branch |
| Work | `hv_status(deck)` | Spot-check workspace state at any point |
| DB | `db_test_connection(root, key)` | Verify DB is reachable before migration |
| DB | `db_list_tables(root, key, db, schema)` | Confirm tables exist after migration |
| DB | `db_get_table_schema(root, key, db, schema, tables)` | Inspect column-level schema |
| DB | `db_execute_sql_query(root, key, db, query)` | Spot-check data or schema details |
| Ship | `hv_ship(deck, message, title, body)` | Commit + push + open PRs across all repos |
| Ship | `hv_list_pulls(deck)` | Confirm PRs opened / merged |
| Ship | `hv_status(deck)` | Confirm all pushed, workspace clean post-ship |

## Decision Tree — Phase 0

```
hv_status
    │
    ├── all clean + on default branch ──────► hv_init   → record branch name
    │
    ├── all clean + on feature branch
    │       └── PRs merged? ────── yes ─────► hv_next   → record branch name
    │                      └───── no  ──────► hv_ship first, then hv_next
    │
    └── any repo dirty
            └── has merged PR + uncommitted? ► hv_stash → hv_next → hv_unstash
            └── otherwise ──────────────────► STOP, fix manually
```
