# Lessons — ansible-p7-editor

## FIX files
- `FIX-001.md` — `AnsibleRoleDetailPage` crashed on real data: client types used
  `file_path` while the API ships camelCase `filePath` (conventions §11).
  Pre-existing; first exposed by this workflow's smoke because the page had
  never been opened against live role files. Fixed client-side (rename across
  api.ts / slice / RoleFileTree / page), redeployed, smoke green.

## What would have helped
1. **A wire-contract check between client response types and the API** — the
   `file_path` mismatch survived since the page shipped because nothing
   exercised `api.ansibleRoles.listFiles` against the real backend. Even one
   integration smoke per page at its shipping phase would have caught it.
2. **Knowing lezer-yaml's actual tolerance up front.** The plan budgeted for
   "missing document start" errors, but lezer-yaml parses uniformly-indented
   chunks and even unclosed quotes without error nodes. The real fragment
   problems are dedent-below-first-line and silent quote absorption — probing
   the parser before designing fragmentMode reshaped the work (synthetic
   nesting-chain prefix with constant offset instead of per-line re-indent
   position maps).
3. **Overlay mounts are invisible to `Tree.iterate`/`cursor()`** — only
   `resolveInner` and `highlightTree` enter them. Cost one test-debug cycle;
   worth remembering for P8, which will walk the template tree for decoration
   targets.
4. **CM6 in jsdom needs two shims** (`Range.getClientRects`,
   `ResizeObserver`) and React Testing Library auto-cleanup is NOT active
   without vitest globals — stale `.cm-editor` nodes from earlier tests made
   five tests fail confusingly until `afterEach(cleanup)` was added.
5. **Scoped `@skip` blocks in Lezer grammars don't reach rules defined
   outside the block** — `Filter { Pipe FilterName }` silently failed on the
   space after the pipe until the rule moved inside the skip block.
6. **The bundle guard earned its keep immediately**: it tripped (correctly) on
   each phase's intentional growth, forcing the budget updates to be explicit
   instead of silent drift.
