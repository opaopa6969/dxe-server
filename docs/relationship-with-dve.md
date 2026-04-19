[日本語版はこちら / Japanese](relationship-with-dve-ja.md)

# Relationship with DVE

DVE (Decision Visualization Engine) is part of DxE-suite (`@unlaxer/dve-toolkit`, `v4.2.0`). This document maps what DVE already implements, where it stops, and where dxe-server might add value.

---

## What DVE Already Implements

DVE is a **complete visualization stack** for the DxE decision graph. It is not a stub.

### Backend (`dve/kit/server/api.ts`, `localhost:4174`)

| Endpoint | Function |
|---|---|
| `POST /api/scan` | Scans a directory tree for DxE-enabled projects (detects DGE, DDE, DRE, DVE presence) |
| `POST /api/register` | Registers discovered projects into the graph |
| `POST /api/annotations` | Writes annotation files back to the project |
| `GET /api/graph` | Returns the full decision graph as JSON |
| `GET /api/health` | Health check |

### State Detection (`dve/kit/parser/state-detector.ts`)

DVE implements full DRE install state detection:

- `FRESH` — `.claude/` does not exist
- `INSTALLED` — DRE version file present, matches kit version
- `OUTDATED` — local version older than kit version
- `CUSTOMIZED` — any `.claude/` file differs from kit source

This covers every DRE state that the original dxe-server design planned to visualize.

### Frontend (`dve/app/src/`, Preact + Cytoscape)

- Interactive graph: Session → Gap → DD → Spec as clickable nodes
- `ScanView.tsx` — project discovery UI that calls `/api/scan`
- Node clustering, filtering, annotation overlay
- Served by Vite at `localhost:4173` (development) / `localhost:4174` (preview + API on same port)

---

## What DVE Does NOT Cover

| Gap | Detail |
|---|---|
| **DDE internal completion state** | DVE records `hasDDE: boolean` per project. It does not visualize which DDE pages are complete, which terms are extracted, which glossary links are missing. |
| **DDE time-series progress** | No history of DDE completion percentage over time |
| **Weekly digest reports** | No scheduled or on-demand narrative report format |
| **DDE-centric view** | DVE is organized around the decision graph (DGE-centric). DDE is a peripheral attribute, not a first-class view. |

These four gaps are the **only non-overlapping territory** for dxe-server.

---

## Honest Overlap Assessment

If dxe-server were to implement its original full scope (DRE state view + DGE Gap lifecycle + multi-project scan), it would **duplicate DVE's core functionality**. That is not a viable path.

The realistic paths are:

1. **Absorb into DVE** — Add DDE visualization as a new view in DVE's Preact app. No new server needed. (ADR-002 Option A)
2. **DDE-only dxe-server** — Implement only the DDE visualization gap. Explicitly defer to DVE for everything else. (ADR-002 Option B)
3. **Coexist** — dxe-server becomes a lightweight companion with a different UX model (e.g., static report generation, no persistent server). (ADR-002 Option C)

---

## Differentiation Candidates

If dxe-server pursues Option B or C, the differentiators that DVE cannot easily absorb are:

| Candidate | Reason DVE cannot easily absorb |
|---|---|
| DDE completion percentage dashboard | DVE's graph model is node/edge; completion rates are aggregate metrics |
| Weekly Markdown digest report | DVE is an interactive UI; static report export is outside its UX model |
| `dde status` terminal output (no browser) | DVE requires a browser; headless status is a different use case |
| Cross-project DDE trend (completion over time) | DVE graph is current-state only; time-series is a new data model |

---

## Conclusion

DVE is the answer for DGE/DRE/decision-graph visualization. **dxe-server's only defensible scope is DDE visualization and DDE-centric reporting.**

Any implementation of dxe-server should:
1. Explicitly state it does not replace or compete with DVE
2. Link users to DVE for decision-graph features
3. Focus exclusively on DDE completion state and weekly report generation

See [ADR-002](decisions/ADR-002-scope-options.md) for the pending scope decision.
