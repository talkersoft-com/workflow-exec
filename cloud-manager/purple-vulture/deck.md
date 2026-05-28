# Deck: cloud-manager

## Deck name
`cloud-manager`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `cloud-manager-web` | Edits to `AnsiblePlaybookEditorPage.tsx`, `PlaybookDetailPage.tsx`, `PlaybooksPage.tsx`, and `RoleFileEditor.tsx` (new `onDirtyChange` prop only). |
| `workflow-exec` | Always — workflow scaffold + Results/Lessons at ship time. |

Repos not listed will be on the feature branch but skipped by `hv_ship`.

## Branch
`purple-vulture` (planning), execution branch recorded in Task 0000

## Initialize (Task 0000)
```
hv_status  deck: "cloud-manager"
hv_init    deck: "cloud-manager"
```
If already provisioned on a prior feature branch with all PRs merged:
```
hv_status  deck: "cloud-manager"
hv_next    deck: "cloud-manager"
```

## Ship
```
hv_ship  deck: "cloud-manager"
         message: "feat(web): make Ansible playbook editor discoverable and dismissible"
         title:   "feat(web): make Ansible playbook editor discoverable and dismissible"
```

## Operational notes for this workflow
- The workflow runs ON `ubuntu-server.talkersoft.com` — the same host that runs the deployed cloud-manager stack. The deployed web app is at `https://ubuntu-server.talkersoft.com` (port 3000 behind nginx).
- The deployed API is at `https://ubuntu-server.talkersoft.com/api` (cloud-manager-api on port 5250); it's not touched by this workflow but is the API the deployed editor PATCHes to.
- Deploy script for web (Phase 3): `cloud-manager-api/scripts/web/install-web-app.py`. Always `sudo rm -rf cloud-manager-web/dist` before re-deploying to avoid prior-deploy permission errors.
- Playwright MCP is the verification harness for Phase 4 — drive the deployed UI directly, capture network requests + screenshots.
- A real playbook (`pg` / `pb_EMEJCH08GG`) exists in `vm.playbooks` for the verification flow; no test data needs to be created.
