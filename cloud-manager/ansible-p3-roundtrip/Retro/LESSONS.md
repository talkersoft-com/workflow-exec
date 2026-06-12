# Lessons: cloud-manager/ansible-p3-roundtrip

## FIX files
None — every task's test battery passed without a retried failure. (Four first-run pytest
failures in phase 0001 were fixed inside the same red→green loop before the battery was
declared run: two test-authored path bugs, a ruamel quirk where `lc` positions of
null-valued keys point at the NEXT token, and flow-collection spans missing the closing
bracket. No task-level retry occurred, so no FIX file per the improvisation policy.)

## What would have helped
1. **The plan's "byte-identical re-dump" assumption needed an empirical check before
   building.** ruamel rt-mode re-emits indentation from one global setting and pins comments
   to absolute columns — the standard Ansible style (top-level dash col 0, nested offset 2)
   is *unrepresentable* in a single emitter config, so `dump(load(x)) == x` is false for the
   most common real files. Twenty minutes of scratch experiments before writing the engine
   redirected the design to text splicing (strictly better: untouched bytes preserved by
   construction). Future plans that hinge on a library behaviour should name a phase-0 spike
   to verify it.
2. **Task 0002 and task 0005 disagreed on sidecar-down semantics** (0002: fidelity check on
   every revision cut, fail loudly; 0005 step 4: raw saves fall back). 0005's split is the
   right call and is what shipped, but the 0002 implementation initially failed raw saves on
   an unreachable sidecar and was reworked in phase 5. Task authors should state the
   unavailable-dependency behaviour in the task that introduces the dependency.
3. **The deploy assumed a compose stack that didn't exist.** The API runs as a systemd
   service and the host had no container runtime at all — docker.io + docker-compose-v2 were
   installed as part of phase 1. PLAN open question 1 ("same compose stack as the API")
   presupposed the API was containerized. Deck/init files should record the actual host
   runtime so the first containerized service in a deck plans its own prerequisites.
4. **P2 retro lesson 4 paid off immediately**: making the bind URL env-overridable
   (`API_BIND_URL`, default unchanged) let every phase run a true second API instance against
   the live DB pre-deploy — all HTTP-level TCs (flag gating, 409s, proxy) were verified
   before the production deploy rather than after, unlike P2.
5. **The decomposer's position counters are not YAML indexes.** Task positions run across
   pre_tasks/tasks/post_tasks; handler positions run revision-wide; only mapping nodes are
   counted. Mapping records back to document paths means replaying the decomposer's exact
   traversal order against the span map. P8 should reuse the ydoc map (as P3's spans now do)
   rather than re-deriving paths from positions.

## What went well
- The splice engine's three guard rails (checksum 409, flow-edit 422, post-splice semantic
  re-parse 500) caught every corruption mode the test corpus could provoke; the semantic
  re-parse also caught two engine bugs during development that span tests missed.
- Same-transaction `revision + ydoc` (client-assigned revision Guid) made the all-or-nothing
  TC trivial — the corrupted-sidecar simulation left zero half-written rows.
- The PLAN.md before/after example reproduced character-for-character on the live API
  (`hosts: webservers` → `hosts: all` with the EOL comment intact, one-line diff).
- P2's patterns (entity/EF/migration/controller conventions, flag gating, decomposer hook
  placement) made the C# layers mechanical to mirror; zero build errors across all phases.
