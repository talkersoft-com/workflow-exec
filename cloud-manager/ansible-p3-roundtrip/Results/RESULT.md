# Result: cloud-manager/ansible-p3-roundtrip

## Outcome
**Complete.** All seven phases executed and tested. A ruamel.yaml round-trip engine runs as a
Python sidecar (`roundtrip-service/`, FastAPI + ruamel + uv/ruff, own Dockerfile + compose,
loopback-only on 127.0.0.1:5260) beside the API. `POST /decompose` returns a structural map
with exact per-node source spans (token-level start AND end positions) + style metadata;
`POST /recompose` splices structured edits into the original text. A `ydoc` record per
`PlaybookRevision`/`RoleFileRevision` stores the source checksum + decomposition map, written
in the same transaction as the revision after a write-time `recompose(decompose(x)) == x`
proof — mismatch fails the save loudly (400), sidecar-down falls back to pre-P3 raw-save
behaviour. The P2-deferred PATCH play/task endpoints build an edit list anchored on the ydoc
map, splice via the sidecar, and save through the EXISTING revision-cutting path. P2's
decomposer prefers the sidecar span map; YamlDotNet marks remain the fallback. Golden-file
corpus green (69 pytest tests), raw-text save path byte-identical, API deployed, migration
applied. No porch/vorch/mcp/web changes.

## Branch
`cushiony-veranda` (created by Task 0000 via hv_next from origin/main; recorded in deck.md)

## PR table
| Repo | PR | Status |
|------|----|--------|
| cloud-manager-api | [#30](https://github.com/talkersoft-com/cloud-manager-api/pull/30) | merged |
| workflow-exec | [#57](https://github.com/talkersoft-com/workflow-exec/pull/57) | merged |

(No PR for workflow-plans — the P3 plan folder was already on main. No PR for
cloud-manager-mcp, cloud-manager-web, vorch-lib, or vorch-service — deck guardrail held.
PR-table fill committed on `serene-drydock`: the deck auto-merges and auto-transitions on
ship, so the merged `cushiony-veranda` branch could not be amended.)

## Phase summary
| Phase | What landed | Test |
|-------|-------------|------|
| 0000 | hv_status (18/18 clean) + hv_next → `cushiony-veranda`; branch recorded in deck.md; P2 artifacts confirmed on main | PASS (4/4) |
| 0001 | `roundtrip-service/` sidecar: splice-based recompose engine (exact byte-range edits — untouched bytes preserved by construction), token-level span map with self-verification, FastAPI decompose/recompose/healthz, 409 checksum conflict, Dockerfile + compose (loopback-only), 9-file golden corpus, 69 pytest tests, uv + ruff | PASS (6/6) |
| 0002 | `YamlDocument` entity (`ydoc`, schema `ansible`, contracts §1) + migration `AddYamlDocuments` (FKs to both revision tables, one-owner check, unique per revision); `IRoundtripClient`; write-time fidelity proof in PlaybookService/RoleFileService (same-transaction revision + ydoc, all-or-nothing); proxy routes `api/v1/roundtrip/decompose\|recompose` flag-gated `ansible-studio` | PASS (6/6) |
| 0003 | `PATCH api/v1/playbook/{pid}/play/{playId}` + `…/task/{ptaskId}` (P2-deferred): set/delete of top-level keys → edit list anchored on ydoc map → sidecar splice → saved via PlaybookService.UpdateAsync (no parallel writer); stale record or span drift → 409, refetch semantics; handler tasks addressable; role-file tasks correctly rejected from playbook routes | PASS (7/7) |
| 0004 | Decomposer prefers ydoc span map (walked in decomposer traversal order, 1:1 alignment, count-mismatch aborts override); sidecar spans flow into `ptask.SourceSpan` (multi-line ends verified vs YamlDotNet point-marks); pre-P3/no-ydoc revisions decompose via YamlDotNet fallback; records still never recomposed to YAML | PASS (5/5) |
| 0005 | Sidecar-down: raw saves fall back (200, no ydoc, decomposition Succeeded), structured edits + proxy → 503; raw-save bytes verified identical (md5); unchanged re-save cuts no revision; flag OFF hides all new routes, existing flows (list/get/materialize, role-file CRUD) unchanged; corpus re-run green; deployment docs in `roundtrip-service/README.md` | PASS (6/6) |
| 0006 | Migration no-op (applied in phase 2), sidecar healthy, api deployed, smoke green, this file + LESSONS, hv_ship | PASS |

## Deploy + smoke
| Target | Result | Smoke |
|--------|--------|-------|
| clouddb migration | `dotnet ef database update` — no-op, already current; `ansible.yaml_documents` verified via `\d` | columns/FKs/indexes match contracts §1 |
| roundtrip sidecar | `docker compose up -d --build` — image built, container `cloud-manager-roundtrip` running | `GET /healthz` 200 |
| api | publish + service restart succeeded | up after 4s (attempt 3/10), HTTP 200 |
| health | `cloud_health_check` (MCP, deployed API) | `{"status":"Healthy"}` |
| roundtrip proxy | `POST api/v1/roundtrip/decompose` through deployed API, flag temporarily on | HTTP 200, span map returned; flag restored to disabled (pre-existing state) |

## Key design decision (recorded for P7–P9)
ruamel re-emits indentation from global settings and pins comments to absolute columns, so a
whole-document re-dump can NEVER reproduce the standard Ansible style byte-identically
(top-level dash at col 0 + indented nested sequences conflict in one emitter config; proven
empirically before building). The engine therefore **splices**: each edit is an exact
byte-range operation on the original text — untouched bytes are preserved *by construction*.
The write-time proof is correspondingly two-part: the sidecar self-verifies every scalar span
against the source (`fidelityOk`/`spanErrors`), and recompose-with-zero-edits must return the
exact bytes. Guard rails: checksum drift → 409; edits inside flow collections → 422 (set the
nearest block-level ancestor); post-splice re-parse compared against structurally-applied
edits → any divergence → 500, output withheld.

## Deviations / notes
- **Sidecar-down semantics** (task 0002 vs 0005): task 0002 alone implied any sidecar failure
  could block saves; task 0005 step 4 pins the intended split, which is what shipped — raw
  saves fall back (no ydoc, warning logged), structured edits and proxy routes fail 503,
  fidelity MISMATCH always fails the save (400). Unparseable YAML never blocks a save and
  gets no ydoc (P2 stance; decomposer marks the revision failed).
- **Structured-edit granularity**: `set`/`delete` operate on top-level keys of the play/task
  mapping. Setting a mapping value (e.g. whole module args) replaces that block with a fresh
  render — quote style inside the *replaced* value may normalize; everything outside the
  edited span is byte-identical. Scalar sets are surgical (single line, EOL comments kept).
- **Play PATCH forbids** `tasks`/`pre_tasks`/`post_tasks`/`handlers` (400) — task lists are
  edited via the task endpoints or the raw editor, not replaced wholesale.
- **`API_BIND_URL` env override added** to Program.cs (default unchanged:
  `http://ubuntu-server.talkersoft.com:5250`) — implements P2 retro lesson 4; every phase
  test ran a true second instance on 127.0.0.1:5299 against the live DB pre-deploy.
- **Docker installed on the host** (docker.io + docker-compose-v2 via apt) — the host had no
  container runtime; the sidecar decision requires one. uv installed user-level.
- **Pre-existing bug observed (NOT P3)**: `DELETE api/v1/playbook/{id}` 500s when revisions
  exist (`playbook_revisions.playbook_id` FK is non-cascading and DeleteAsync doesn't remove
  revisions). Test data was cleaned via SQL. Worth a follow-up fix outside this series.
- `Microsoft.Extensions.Http` added to CloudManager.Data.Services (for the typed
  `IRoundtripClient`); version matches the Vault client's (10.0.8).
