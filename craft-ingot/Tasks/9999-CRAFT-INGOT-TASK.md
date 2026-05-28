# Task: Open PRs and merge

## Task ID
`9999-CRAFT-INGOT-TASK` (always the last task of every workflow; pick `9999` regardless of how many phases your workflow has)

## Parent Orchestration
`Orchestrate/0001-CRAFT-INGOT-ORCH.md`

## Objective
For every repo in `branches.md`, open a PR from the workflow's branch into the repo's default branch, then merge. After all PRs are merged, redeploy every affected service from the new main.

## Steps

1. **Open PRs** for every branch in `branches.md`:
   ```bash
   for d in <repos from branches.md>; do
     cd $d
     gh pr create \
       --base $(git remote show origin | grep "HEAD branch" | awk '{print $NF}') \
       --head <workflow-name> \
       --title "<workflow-name>: <one-line summary>" \
       --body "<short description; link to ../Results/0001-...md>"
   done
   ```

2. **If a sibling workflow merged first**, rebase before merging:
   ```bash
   git fetch origin
   git checkout <workflow-name>
   git rebase origin/main   # or origin/master if that's the default
   # resolve any conflicts; usually rare if the workflows touch different code paths
   git push --force-with-lease origin <workflow-name>
   ```

3. **Merge** every PR. Use `--merge` (preserves the branch history) unless the project's convention is `--squash` or `--rebase`:
   ```bash
   gh pr merge <pr#> --merge --delete-branch=false
   ```

4. **Pull main** in every repo and **redeploy** the affected services:
   ```bash
   for d in <repos>; do
     cd $d && git checkout main && git pull --ff-only
   done
   # then re-run each service's install script (API, web, worker, MCP, etc.)
   ```

5. **Update the Result** with the merge commit short SHAs so future readers can find what landed where.

## Acceptance Criteria
- Every PR in `branches.md` shows state `MERGED`.
- The default branch of every repo includes the workflow's commits.
- All affected services are running the merged code (verify by hitting any new endpoint or grepping the deployed bundle for a new symbol).
- The Result file lists the merge commit SHAs.

## Test
`Test/9999-CRAFT-INGOT-TEST.md`

## On Failure
- PR conflicts: rebase + re-push; do NOT merge with conflicts.
- A required reviewer wasn't added: the workflow stops here, awaiting human approval. Document in an Improvise; resume when approved.
- A repo's branch protection blocks the merge: surface to the operator, document, don't bypass.

## Why this exists
Code on an unmerged branch is invisible to the next workflow. If you skip this, you'll find yourself rebuilding the next workflow against a main that doesn't have your prior work — and the operator won't see your feature in the UI until someone (probably them) merges the branch by hand.
