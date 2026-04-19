[日本語版はこちら / Japanese](README-ja.md)

# dxe-server

Visualization server for the DxE toolkit suite — **DDE state visualization + weekly reports**.

> **Status: v0.1.0 scaffold — design in progress.**
> `bin/dxe-server.js` does not yet exist. The CLI entry point declared in `package.json` is a placeholder only.

---

## The Problem

DxE-suite (DGE / DDE / DVE / DRE) accumulates state across projects:

- DRE tracks install state (FRESH / INSTALLED / OUTDATED / CUSTOMIZED) but only shows it in the terminal
- DGE accumulates Gap sessions as `.md` files — readable by humans, not aggregatable
- **DDE document-completion state has no visualization at all**

DVE (Decision Visualization Engine, inside DxE-suite) already covers the DGE and DRE visualization story. See [docs/relationship-with-dve.md](docs/relationship-with-dve.md) for the full overlap analysis.

**Critical Gap #2 and #3 are unresolved** — the exact scope of dxe-server is still undecided. See [docs/architecture.md](docs/architecture.md).

---

## Intended Scope (under discussion)

| Feature | Status |
|---|---|
| DDE document-completion visualization | Target scope |
| Weekly digest reports (DDE progress) | Target scope |
| DRE state visualization | Overlaps with DVE — see Gap analysis |
| DGE Gap lifecycle visualization | Implemented by DVE — likely out of scope |
| Multi-project scan (`/api/scan`) | Implemented by DVE — likely out of scope |

---

## Position in the DxE Series

```
DGE-toolkit   → find design gaps (dialogue drama)
DDE-toolkit   → find and fill doc gaps
DVE           → visualize decision graph (Session→Gap→DD→Spec)
DRE-toolkit   → distribute and enforce rules/skills
DxE-suite     → bundles all four as one installable suite
─────────────────────────────────────────────────────
dxe-server    → DDE visualization + weekly reports (optional, standalone)
```

dxe-server works without DxE-suite installed. It is an **optional add-on**, not a required component.

---

## Intended Usage (when implemented)

```
npx dxe-server start
```

Starts a local server and opens the browser. No background daemon — on-demand only.

---

## Current State

This repository is a **scaffold**. No runnable code exists yet.

Known gaps blocking implementation:

- **Critical Gap #2** — The core value of an API server is multi-project cross-aggregation. For a single project, file output from DDE CLI is sufficient. The business case for a server must be established first.
- **Critical Gap #3** — Who uses this, when, and how often is undefined. Usage frequency determines the architecture: daily dashboard → persistent daemon; weekly check → on-demand `npx`; one-off → `dxe status` CLI output is enough.

See [design-materials/intake/initial-design.md](design-materials/intake/initial-design.md) for the full DGE session that produced these gaps.

---

## Documentation

| Document | Description |
|---|---|
| [docs/architecture.md](docs/architecture.md) | Component responsibilities, boundary with DVE, future DDE scope |
| [docs/relationship-with-dve.md](docs/relationship-with-dve.md) | DVE overlap analysis — what DVE already does, differentiation candidates |
| [docs/decisions/ADR-001-separate-package.md](docs/decisions/ADR-001-separate-package.md) | Decision: standalone package, not merged into DxE-suite |
| [docs/decisions/ADR-002-scope-options.md](docs/decisions/ADR-002-scope-options.md) | Open decision: Option A (absorb into DVE) / B (DDE-only) / C (coexist) |
| [CHANGELOG.md](CHANGELOG.md) | Version history |

---

## Contributing

This project is in early design phase. Open an issue to discuss before sending a PR.

**License**: MIT
