# Improvise: Phase 0003 TC-007 expectation mismatch

## Phase
`0003-ANSIBLE-SHENANIGANS`

## What the plan said
TC-007: "insert a dummy `vm.playbooks` row via psql with minimal columns; confirm `public_id` is auto-populated with `pb_<10 char Crockford>`".

## What's actually true
`public_id` is assigned by an **EF Core SaveChangesInterceptor** (`CloudManager.Entities/PublicId/PublicIdInterceptor.cs`), not a Postgres trigger. A raw psql INSERT has no path through .NET, so `public_id` stays NULL.

Confirmed by reading `ApplyPublicIdConfiguration` in `CloudManagerDbContext` — it only declares the column + unique index; no trigger DDL anywhere in the project (`grep tgname pg_trigger` returns 0 rows for public_id triggers).

Existing tables (e.g. `membership.feature_flags`) behave the same way — the seed I added in Phase 0002 only worked because I built the public_id explicitly in the migration SQL.

## Decision
Test TC-007 with an **explicit** public_id, which proves the column accepts the format + unique constraint works. Defer the interceptor end-to-end smoke until Phase 0004 ships a real `POST /api/v1/playbooks` endpoint and the route can be exercised over HTTP.

## Verification
```sql
INSERT INTO vm.playbooks (..., public_id, ...) VALUES (..., 'pb_TESTING001', ...);  -- INSERT 0 1
SELECT public_id FROM vm.playbooks WHERE name='__smoke_test__';                       -- pb_TESTING001
DELETE FROM vm.playbooks WHERE name='__smoke_test__';                                  -- DELETE 1
```
Cleanup mandatory per the test doc; completed.

## Followup
Phase 0004 acceptance: when the playbook create endpoint lands, re-run TC-007's *spirit* — `curl -XPOST /api/v1/playbooks` and assert the returned `id` matches `^pb_[0-9A-HJKMNP-TV-Z]{10}$`. That's the real interceptor test.
