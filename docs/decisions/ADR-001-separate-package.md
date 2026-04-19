# ADR-001: dxe-server as a Standalone Package

- **Date**: 2026-04-04
- **Status**: Accepted
- **Deciders**: DGE session (initial design)

---

## Context

DxE-suite manages the installation of DGE, DDE, DVE, and DRE as a monorepo. An option was considered to add `npx dxe ui` as a subcommand of DxE-suite itself.

A second option was to build visualization as a DxE-suite plugin, requiring DxE-suite to be installed.

---

## Decision

Create `@unlaxer/dxe-server` as an independent npm package. Do not integrate it into DxE-suite.

---

## Rationale

| Reason | Detail |
|---|---|
| Separation of concerns | DxE-suite is the kernel — install management and hook enforcement. Adding a visualization server would blur its responsibility. |
| Optional installation | Many users may never need a browser UI. A standalone package keeps the DxE-suite install footprint minimal. |
| Future extensibility | A standalone server can evolve toward team-shared hosting, CI reporting, or scheduled digests without coupling to DxE-suite's release cycle. |
| Single project vs. multi-project | Multi-project cross-aggregation requires a server that runs outside any single project directory. A DxE-suite subcommand tied to one project's install cannot serve this use case. |

---

## Consequences

- `dxe-server` releases independently of DxE-suite
- Users must install `dxe-server` separately (`npx dxe-server start` or global install)
- DxE-suite does not depend on dxe-server; dxe-server may depend on DxE-suite types/data formats
- The `bin/dxe-server.js` entry point must be implemented before the package is usable (currently missing)
