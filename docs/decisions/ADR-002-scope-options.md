# ADR-002: Scope — Absorb into DVE / DDE-only / Coexist

- **Date**: 2026-04-19
- **Status**: OPEN — decision pending
- **Deciders**: TBD

---

## Context

DVE (Decision Visualization Engine, part of DxE-suite v4.2.0) already implements:

- `state-detector.ts` — DRE FRESH / INSTALLED / OUTDATED / CUSTOMIZED detection
- `POST /api/scan` — multi-project discovery
- Preact + Cytoscape Web UI for the full Session → Gap → DD → Spec graph

The original dxe-server design (see [design-materials/intake/initial-design.md](../../design-materials/intake/initial-design.md)) planned to duplicate these features. That creates a direct conflict.

Two unresolved critical gaps block the scope decision:

- **Critical Gap #2**: The value of an API server is multi-project aggregation. For a single project, DDE CLI file output is sufficient. The server's business case is unproven.
- **Critical Gap #3**: User profile and usage frequency are undefined. Daily use → daemon; weekly use → on-demand `npx`; one-off → CLI output replaces the server.

---

## Options

### Option A — Absorb dxe-server into DVE

**What**: Add a DDE visualization view to DVE's existing Preact app. Deprecate the dxe-server repository or archive it.

**Pros**:
- One server, one UI — no duplication
- DVE already has the Preact + Cytoscape infrastructure
- Eliminates the maintenance burden of a second server

**Cons**:
- DVE's graph model (node/edge) is not naturally suited to completion-rate metrics
- Requires changes to DxE-suite (DVE is part of the monorepo)
- DDE visualization would be subordinate to the DGE-centric graph model

**When to choose**: If DDE visualization is simple enough to fit as a side panel or overlay in DVE's existing UI.

---

### Option B — DDE-only dxe-server (recommended starting point)

**What**: Implement dxe-server with exactly one scope: **DDE document-completion visualization and weekly digest reports**. Explicitly delegate DRE, DGE, and scan features to DVE.

**Pros**:
- No feature overlap with DVE
- Clear, defensible scope
- Weekly digest (Markdown or HTML) is a genuinely new output format DVE does not produce
- Can be headless (no browser required) — different UX from DVE

**Cons**:
- Narrow scope may limit adoption
- Gap #2 and Gap #3 still apply: single-project users may not need a server
- Requires DDE to produce machine-readable completion data (format TBD)

**When to choose**: If the goal is a lightweight, focused tool that complements DVE without competing.

---

### Option C — Coexist as a different UX model

**What**: dxe-server becomes a **report generator** rather than an interactive server. `npx dxe-server report` produces a static Markdown or HTML file summarizing DDE progress across registered projects.

**Pros**:
- No persistent server → no Gap #3 (usage frequency) problem
- Static report is CI-friendly (can be committed, emailed, posted to Slack)
- Completely non-overlapping with DVE (DVE is interactive only)
- Solves the weekly digest use case directly

**Cons**:
- The repository name `dxe-server` is misleading for a report generator
- Reduced scope: no real-time dashboard
- Still requires Gap #2 answer: is multi-project aggregation the value, or single-project summary?

**When to choose**: If the primary use case is async reporting (weekly email digest, CI artifact) rather than an interactive browser UI.

---

## Recommendation

**Start with Option B** (DDE-only), **evaluate Option C** (report generator) as the initial deliverable.

Rationale: Option C resolves Gap #3 immediately (no usage-frequency dilemma for a one-shot report command) and produces a concrete artifact (Markdown/HTML report) that validates whether multi-project aggregation actually matters before committing to a persistent server.

Option A should be revisited after Option B/C proves DDE visualization value.

---

## Decision

**[ PENDING ]**

This ADR must be resolved before implementation begins. Update the Status field and fill in the Deciders once the decision is made.

---

## Consequences of Option B (if chosen)

- `bin/dxe-server.js` implements a minimal Node.js server or report generator, DDE-only
- README and architecture docs updated to clearly state DVE handles DRE/DGE/scan
- DDE must produce a machine-readable completion summary (new feature in DDE, not in dxe-server)
- A new DGE session recommended to resolve Gap #2 and #3 before coding starts
