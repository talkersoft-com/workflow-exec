# Parity checklist — agent-arcade vs agent-arcade-studio

Legend: ✅ build + logic verified · 👁 needs live visual/round-trip acceptance · ⚠ carry-forward gap

## Studio (React + Zustand)
| Surface | Status |
|---|---|
| Agents list (cards, badges, group/session sub, drag-reorder, empty state) | ✅ / 👁 |
| Agent editor — General / Agent Settings / Dictation gating / Diagnostics | ✅ / 👁 |
| Avatar generation + regenerate | ✅ / 👁 |
| Header clone / delete (block + confirm) | ✅ / 👁 |
| Settings — General / Backend (multi-server) / Displays / Organization | ✅ / 👁 |
| First-run guided Tour | ✅ / 👁 |
| YAML round-trip (same schema, normalization) | ✅ (no schema/main change) / 👁 |
| `@`-macro authoring | N/A — reference Studio has none (macros are hand-edited YAML) |

## Arcade (vanilla + XState + mitt)
| Surface | Status |
|---|---|
| Agents rail (grouped carousel, system filter `f`, 1–9 jump, welcome orb) | ✅ / 👁 |
| Agent view + ⌘←/→ switching (nav guards) | ✅ / 👁 |
| **Durable per-agent draft** across navigation (headline fix) | ✅ Node-verified / 👁 |
| Dictation ⌘D — recording→pending→confirmed\|error on Go events | ✅ / 👁 round-trip (mic + backend) |
| recordingNavBehavior send / lock + Esc-discard | ✅ Node-verified / 👁 |
| Async-commit to the originating agent across navigation | ✅ Node-verified / 👁 |
| Live pane peek (`t`, getText scrape) | ✅ / 👁 cadence vs real pane |
| ^C interrupt | ✅ / 👁 |
| Sync mode ⌘F (keyEventToBytes) | ✅ 17/17 / 👁 |
| Workspace shell ⌘W (node-pty + xterm.js) | ✅ wiring + spawn-helper / 👁 actual spawn + render |
| `@`-macro picker select/text/flag + compose | ✅ 8/8 / 👁 |
| Optimistic-with-reconciliation (non-zero wez exit → error state) | ✅ / 👁 real exit code |
| Keyboard contract ⌘⌥A ⌘D ⌘F ⌘W ⌘←/→ ^C Esc | ✅ routing / 👁 |

## Carry-forward gaps (from Phase 0004 — present in DOM, not yet driven)
| Item | Status |
|---|---|
| warn-on-exit / exit-confirm overlay | ⚠ not wired |
| First-run tour driving (Arcade side) | ⚠ not wired |
| `?` help overlay | ⚠ not wired |
| recording indicator polish | ⚠ partial |

## Release artifact
| Check | Status |
|---|---|
| Full `npm run build` (js + WezTerm + Go bridges) | ✅ |
| `npm pack` (105 MB) | ✅ |
| Install into clean prefix + postinstall | ✅ |
| `spawn-helper` executable (⌘W path) | ✅ |
| Launches in real Electron without crash (Studio) | ✅ |
| Bin `agent-arcade` present | ✅ |

**Net:** every surface is built and logic-verified; the 👁 items need one live acceptance pass
(operator's machine + computer-use grant + Spark backend). The ⚠ items are small follow-ups.
