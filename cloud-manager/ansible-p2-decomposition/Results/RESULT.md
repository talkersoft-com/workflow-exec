# Result: cloud-manager/ansible-p2-decomposition

## Outcome
**Complete.** All seven phases executed and tested. Stored playbooks and role files are
decomposed into addressable records (`play` `ptask` `hdlr` `vblk`) rebuilt atomically per
revision, with a typed `EntityEdge` (`edge`) graph (`depends_on`, `includes`, `consumed_by`,
`targets`, `defines`, `overrides`), role-dependency cycle detection (400), flag-gated
usedBy/graph/read APIs, an idempotent decompose-all backfill (run against the live DB —
0 head revisions left pending), and four new MCP tools in `graph.ts`. Existing
playbook/role/run flows verified byte-identical post-deploy. The YAML blob remains the source
of truth; no code path recomposes records to YAML (structured-edit PATCH deferred to after P3).

> **Operator action required:** Restart cloud-manager-mcp via `/mcp` in Claude Code to pick
> up the new dist.

## Branch
`clawed-mainmast` (created by Task 0000 via hv_next from origin/main; recorded in deck.md)

## PR table
| Repo | PR |
|------|----|
| cloud-manager-api | TBD (filled after hv_ship) |
| cloud-manager-mcp | TBD (filled after hv_ship) |
| workflow-exec | TBD (filled after hv_ship) |

## Phase summary
| Phase | What landed | Test |
|-------|-------------|------|
| 0000 | hv_status (18/18 clean) + hv_next → `clawed-mainmast`; branch recorded in deck.md | PASS |
| 0001 | `PlaybookPlay`/`PlaybookTask`/`PlaybookHandler`/`VarBlock` entities in schema `ansible`, prefixes `play` `ptask` `hdlr` `vblk`, `DecompositionState` marker on both revision entities, migration `AddPlaybookDecompositionRecords` | PASS (6/6) |
| 0002 | `IPlaybookDecomposer` + YamlDotNet implementation; one-transaction delete-and-recreate per revision; SourceSpan from parser marks; hooks in `PlaybookService.UpdateAsync` + `RoleFileService.UpdateAsync` (the only two revision-cutting paths; restore re-enters update); malformed YAML never blocks the save — revision marked `failed` and surfaced in revision DTOs | PASS (20/20 live checks) |
| 0003 | `EntityEdge` + migration `AddEntityEdges` (unique from/to/edgeType, check-constrained edge types); edges rebuilt inside the decomposer transaction; `roles:` + pgr attach → `includes`; meta/main.yml deps → `depends_on` with DFS cycle check (400) in the save path; Jinja2 scan → `consumed_by` with `var:<name>` pseudo-ids; hosts-pattern → `targets` against P1 inventory records; var blocks → `defines`/`overrides` | PASS (10/10 live checks) |
| 0004 | `GET api/v1/entity/{id}/usedBy` + `…/graph?depth=` (default 2, clamp 1–5) + play/task/handler/varBlock read endpoints + `POST api/v1/playbook/decompose-all` (idempotent, `?force=true` override), all `[RequireFeatureFlag("ansible-studio")]`; no PATCH/PUT on record entities | PASS (15/15 live checks) |
| 0005 | MCP `graph.ts`: `cloud_entity_used_by`, `cloud_entity_graph`, `cloud_playbook_play_list`, `cloud_playbook_task_list`; registered in `hv.cloud` profile; dist rebuilt; regression baseline captured; backfill coverage verified by SQL (0 pending heads, 0 succeeded-but-empty playbook heads) | PASS |
| 0006 | Migration applied (no-op — already current), api + mcp deployed, decompose-all re-run over HTTP, post-deploy regression diff, this file + LESSONS, hv_ship | PASS |

## Deploy + smoke
| Target | Result | Smoke |
|--------|--------|-------|
| clouddb migration | `dotnet ef database update` clean; 5/5 new `ansible` tables present | SQL information_schema check |
| api | publish + service restart succeeded | up after 4s, attempt 4/10, HTTP 200 |
| mcp | `npm ci` + `tsc` dist rebuild succeeded | 4 graph tools present in `dist/tools/graph.js` (operator `/mcp` restart pending) |
| health | `GET api/v1/Health/check` | HTTP 200 `{"status":"Healthy"}` |
| backfill (HTTP) | `POST api/v1/playbook/decompose-all` | 200 `{processed:0, skipped:2, failed:0}` — idempotent, all heads already covered |

## Post-deploy regression (byte-identical check)
Captured from the pre-deploy (main-code) API and diffed against the deployed build on the same
data: playbook list/get (×2), globalRole list, run list — **byte-identical**. Revision list
differs **only** by the additive `decompositionState` field (and `decompositionError` on the
full revision DTO), which Task 0002 explicitly requires surfacing. `ansible-studio` disabled →
new endpoints return the flag-gated response; enabled → they serve; flag restored to disabled
(its pre-existing state). Zero diffs in cloud-manager-web, vorch-lib, vorch-service.

## Deviations / notes
- **Failure events (Task 0004 step 4):** contracts §1/§6 allow no new P2 event entity and no
  existing event aggregate (vm/inventory/blueprint) fits a playbook revision. Failures are
  recorded durably on the revision row (`decomposition_state` + `decomposition_error`,
  API-visible) plus structured logging — verified by TC-007. A `playbook_events` table was
  deliberately NOT added; if a real event stream is wanted, amend the contracts in a future PR.
- **Record parents extended for role decomposition (PLAN OQ2 default "both"):** `ptask` and
  `hdlr` accept a `role_file_revision_id` parent (exactly-one-parent check constraints);
  `vblk` carries nullable revision anchors so rebuilds and DB cascades stay consistent.
- **EntityEdge internal columns:** `source_playbook_revision_id` / `source_role_file_revision_id`
  (not on the wire) let a rebuild replace exactly its own derivations; mutation-path edges
  (pgr attach, depends_on) leave them null. pgr detach re-decomposes the head revision so a
  YAML-derived `includes` edge survives detach.
- **depends_on edges** are maintained in the role-file save path (create/update/delete of
  meta/main.yml), not the decomposer — the cycle check must reject the save with 400 before
  commit, and creates don't cut revisions.
- **Backfill scope:** revisions only exist after the first edit (create stores content on the
  parent row); playbooks/role files never edited have no revision and therefore no records —
  inherent to the repo's revision model, not a backfill gap.
- The missing input `master-plans/ANSIBLE-EPIC-CONTRACTS.md` was recovered from
  `origin/enjoyable-tegu` (commit 3ab4e0e) in workflow-plans — see LESSONS.

## Explicit deferrals (coverage map)
- Structured-edit PATCH on play/task raw attrs → after P3 (round-trip fidelity first).
- `var:<name>` pseudo-ids re-key to `vdef_…` in P4's migration.
- Edge cleanup on playbook/role hard-delete paths (deletes are currently blocked by FK
  restrictions when revisions/refs exist; revision-anchored edges cascade automatically).
