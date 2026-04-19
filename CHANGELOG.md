# Changelog

All notable changes to dxe-server are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).
This project uses [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Planned
- `bin/dxe-server.js` — CLI entry point (currently missing)
- DDE document-completion visualization API
- Weekly digest report generation
- Resolve Critical Gap #2 (multi-project value proposition)
- Resolve Critical Gap #3 (user profile and usage frequency)
- ADR-002 scope decision (Option A / B / C)

## [0.1.0] - 2026-04-04

### Added
- Initial scaffold: `package.json` with `@unlaxer/dxe-server` name, version, and `bin` declaration
- `README.md` — initial positioning in DxE series
- `design-materials/intake/initial-design.md` — DGE session output: architecture decisions, Gap list, open questions

### Known Issues
- `bin/dxe-server.js` declared in `package.json` but not yet created — `npx dxe-server start` will fail
- Critical Gap #2 unresolved: business case for API server vs. CLI file output not established
- Critical Gap #3 unresolved: user profile and usage frequency undefined
- DVE (DxE-suite) already implements state-detector, `/api/scan`, and Preact+Cytoscape Web UI — scope overlap not resolved
