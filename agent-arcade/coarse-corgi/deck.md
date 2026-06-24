# Deck: agent-arcade (execution)

## Deck name
`agent-arcade`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `talkersoft-com/agent-arcade` | The rebuild target — all implementation lands here. |

Reference only (NOT modified): `talkersoft-com/agent-arcade-studio` (read-only 1:1 parity
source). Out of scope: `talkersoft-com/agent-arcade-api`. The `planning/*` repos hold the
plan + this scaffold. Repos not listed are on the feature branch but skipped by `hv_integrate`.

## Branch
`TBD` — the execution branch is created at run time in Task 0000 and recorded here.

## Initialize (Task 0000)
```
hv_status  deck: "agent-arcade"
hv_init    deck: "agent-arcade"
```
If already provisioned on a prior feature branch with all PRs merged:
```
hv_status  deck: "agent-arcade"
hv_next    deck: "agent-arcade"
```

## Ship (per phase)
```
hv_integrate  deck: "agent-arcade"
         message: "<commit message>"
         title:   "<PR title>"
```

## Release (after parity pass, Phase 0007)
```
hv_release  deck: "agent-arcade"
       title: "Release: Agent Arcade rebuild"
```

## Build / test commands
- **agent-arcade (Electron + Go bridges) changed** →
  `cd /Users/talker/workspace/agent-arcade/agent-arcade && npm run build`
  (vendors WezTerm, builds the CGO Go bridges, and runs the esbuild JS stage — must succeed).
  Sanity-check edited renderer/main JS with `node --check <file>`.
