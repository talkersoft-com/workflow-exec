# Task: Flip the flag

## Task ID
`0011-ANSIBLE-SHENANIGANS-TASK`

## Parent Orchestration
`Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md`

## Status
- [ ] In progress
- [ ] Complete
- [ ] Failed

## Objective
Production cutover (such as there is one — currently a single environment). Final Result + final Wishlist.

## Steps

1. **Confirm preconditions**
   - Phase 0010 Result is Success.
   - Worker is running: `systemctl is-active cloud-manager-worker` → active.
   - API is running.

2. **Flip the flag**
   ```bash
   curl -X PATCH http://localhost:5250/api/v1/admin/feature-flags/playbooks \
     -H 'Content-Type: application/json' \
     -d '{"enabled": true}'
   ```

3. **Wait for healthy=true**
   - Watch `psql -c "SELECT healthy, last_probed_at FROM membership.feature_flags WHERE key='playbooks'"` every 30s.
   - Within probe interval (5 min) healthy should flip to true.
   - If not: do not proceed. Phase 0006's health probe regressed.

4. **UI verification**
   - Load the web app fresh (clear cache / hard reload).
   - Confirm `/playbooks` link in nav.
   - Confirm VmDetailPage shows Playbooks section.

5. **Write the final Result**
   - Reference all prior phase Results.
   - Note the smoke VM is still up for inspection.
   - Note any phase that required improvisations (so the next reader of this workflow can find the pitfalls quickly).

6. **Write the final Wishlist**
   - This is the *whole workflow* reflection, not just this phase.
   - Look across all per-phase Wishlists; what kept coming up?
   - MCPs / docs / access that would have cut total wall-clock; what compounded; what was a one-off.
   - This file informs the operator's next tooling investment.

## Acceptance Criteria
- Flag enabled=true, healthy=true in DB.
- UI shows playbook sections.
- Final Result + Wishlist written.

## Test
`Test/0011-ANSIBLE-SHENANIGANS-TEST.md`
