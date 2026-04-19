[日本語版はこちら / Japanese](architecture-ja.md)

# dxe-server — Architecture

> **Status**: Design phase. No runnable code exists. `bin/dxe-server.js` is missing.

---

## Component Responsibilities

dxe-server is an **optional visualization add-on** for the DxE toolkit suite. Its intended responsibilities are:

| Responsibility | Owner |
|---|---|
| DDE document-completion state visualization | **dxe-server** (target) |
| Weekly digest reports (DDE progress over time) | **dxe-server** (target) |
| Decision graph (Session → Gap → DD → Spec) | **DVE** (implemented) |
| DRE install state (FRESH/INSTALLED/OUTDATED/CUSTOMIZED) | **DVE** (implemented via `state-detector.ts`) |
| Multi-project scan (`/api/scan`) | **DVE** (implemented) |
| Preact + Cytoscape Web UI | **DVE** (implemented) |
| Rules/skills distribution and enforcement | **DRE** |
| Design gap extraction | **DGE** |

The critical question — whether dxe-server should cover DRE state and multi-project scan at all — is **unresolved**. See [ADR-002](decisions/ADR-002-scope-options.md).

---

## Intended Architecture (when implemented)

```
┌─────────────────────────────────────────┐
│  npx dxe-server start                   │
│                                         │
│  Node.js server (port TBD)              │
│  ├── GET  /api/dde/status               │  DDE completion state per project
│  ├── GET  /api/dde/weekly-report        │  weekly progress digest
│  └── GET  /                             │  browser UI (static SPA)
│                                         │
│  Data sources:                          │
│  ├── DDE output files (project dirs)    │
│  └── project registry (config file)    │
└─────────────────────────────────────────┘
```

**Not yet decided**: whether this server persists (daemon) or is ephemeral (on-demand `npx`).

---

## Boundary with DVE

DVE (Decision Visualization Engine, part of DxE-suite) is the primary visualization tool for the DxE suite. It runs at `localhost:4174` and provides:

- A Preact + Cytoscape graph of all Sessions, Gaps, DDs, and Specs
- The `/api/scan` endpoint that discovers DxE-enabled projects on disk
- `state-detector.ts` — detects DRE install state per project
- Annotation write-back API

**dxe-server must not duplicate DVE's existing capabilities.**

The non-overlapping opportunity is **DDE visualization** — DVE tracks that a project `hasDDE: true` (boolean) but does not visualize DDE's internal completion state: which pages are complete, which terms are linked, which gaps remain open.

---

## Future DDE Visualization Scope

DDE produces the following data that has no current visualization:

| DDE artifact | Location | Currently visible |
|---|---|---|
| Term extraction results | `dde/terms/` | No |
| Generated articles | `dde/articles/` | No |
| Auto-linked doc pages | source docs | Partially (diff only) |
| Completion percentage | — (no file yet) | No |
| Missing glossary links | — (no file yet) | No |

The DDE visualization story is the **primary differentiator** for dxe-server.

---

## Open Issues

1. **Critical Gap #2** — Multi-project cross-aggregation is the core value of a server. A single-project user gets the same value from `dde status` CLI output. The server's business case must be established before implementation begins.

2. **Critical Gap #3** — Who uses this, when, and at what frequency?
   - Daily: needs a persistent daemon + change notifications
   - Weekly: on-demand `npx dxe-server start` is sufficient
   - One-off: `dxe status` CLI output replaces the server entirely

3. **ADR-002 unresolved** — Three options remain open. See [ADR-002](decisions/ADR-002-scope-options.md).

---

## Related Documents

- [relationship-with-dve.md](relationship-with-dve.md) — detailed DVE overlap analysis
- [decisions/ADR-001-separate-package.md](decisions/ADR-001-separate-package.md) — why dxe-server is a standalone package
- [decisions/ADR-002-scope-options.md](decisions/ADR-002-scope-options.md) — open scope decision
- [../design-materials/intake/initial-design.md](../design-materials/intake/initial-design.md) — original DGE session
