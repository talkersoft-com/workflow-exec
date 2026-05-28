# Lessons: cloud-manager/cosmic-strudel (neon-addax workflow)

## What would have helped

1. **The DATA-MODEL-DELTAS spec was wrong about needing the unique indexes.** Inspecting `CloudManagerDbContext.OnModelCreating` showed the `RoleFile (role_id, file_path)` and `PlaybookRoleFile (playbook_role_id, file_path)` unique indexes were already declared — they just happened to be in sync with the live schema too. The delta could have called this out by checking the DbContext rather than only inferring from `DATA-MODEL.md`. Net effect: the migration shrank to just two columns. Future: a quick `grep HasIndex.*IsUnique` of the DbContext before declaring "needs an index" would catch this.

2. **HTTP smoke tests need a runtime story.** Most Task tests assumed "the API is running locally on the configured port." It wasn't. I deferred those checks to `Retro/FIX-001.md` and used `dotnet build` + DB schema verification via MCP as my pass criteria. A better workflow scaffold would either (a) include a Task 0001a "start the API and wait for healthy" with explicit env vars, (b) provide an integration test project that exercises the endpoints in-process via `WebApplicationFactory`, or (c) explicitly mark HTTP tests as "manual operator verification" so the workflow doesn't pretend they're automated.

3. **The argument-specs merge semantics were under-specified.** API-PLAN said "re-derive Playbook.argument_schema" but didn't say how to combine multiple roles' schemas. I implemented union-of-properties (last-wins on conflict) + union-of-required, which is reasonable but not explicitly the spec. A real production version probably wants namespacing (e.g., `<role_name>.<option_name>`) to avoid silent collisions. Worth refining in a follow-on plan.

4. **`Edit` tool sometimes failed with "File has not been read yet"** even after recent edits to the same file. Had to re-`Read` to satisfy the cache. Cost a handful of round trips. Not blocking, just friction.

5. **The "AddAnsibleConstraintsAndTargetTimestamps" name is now slightly misleading** since only the timestamps actually appeared in the migration body. Could rename to `AddPlaybookRunTargetTimestamps`. Not worth a follow-on PR to fix; flag for renaming if/when a related migration lands.

## Fix files written
- FIX-001: HTTP smoke tests deferred across Tasks 0002–0010 (API not running; `dotnet build` + MCP schema checks substituted)
