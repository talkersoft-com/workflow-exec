# Deck: cloud-manager (exec copy)

## Deck name
`cloud-manager`

## Branch
`minty-rooster`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `cloud-manager-mcp` | 3 new tools in `src/tools/vms.ts` + wrapper hardening in `src/runner.ts`; registered in `src/index.ts`; `npm run build` clean |

All other deck repos sit on `minty-rooster` but are skipped by `hv_ship`.

## Ship call
```
hv_ship  deck: "cloud-manager"
         message: "mcp-coverage-tier-1: cloud_vm_update + ip_address_update + events"
         title:   "mcp coverage tier 1: 3 new VM tools (update, ip-address, events)"
```
