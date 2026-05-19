# Nablarch Class Visualization Tool — Implementation Plan

**Version**: 1.0  
**Created**: 2026-05-20  
**Prerequisite**: Technology selection (cmd_474), Class count measurement (cmd_476), Requirements (requirements.md), Architecture (architecture.md)

---

## Overview

The implementation is divided into four phases. Each phase produces a working, demonstrable artifact. Phases 1–3 are essential; Phase 4 is optional depending on team collaboration needs.

| Phase | Name | Target Duration | Deployment |
|-------|------|----------------|------------|
| 1 | PoC — Single Version, Core Visualization | 1–2 weeks | Option A (Static) |
| 2 | Version Management | 1–2 weeks | Option A or B |
| 3 | Full Exploration UI | 1–2 weeks | Option A or B |
| 4 | Team Distribution | Optional | Option C (Docker) |

---

## Phase 1: PoC — Single Version, Core Visualization

**Goal**: Prove that 2,681 Nablarch nodes can be rendered interactively in a browser using Cytoscape.js, with artifact color-coding and basic search.

### Scope

- Analysis pipeline: ClassGraph scan of all 92 Nablarch repositories; extract `EXTENDS` and `IMPLEMENTS` relationships only.
- Frontend: Single-page Cytoscape.js app; load pre-generated JSON; render nodes and edges.
- Layout: fcose layout (compute once; save coordinates to `classes.json`).
- Features:
  - All nodes visible on initial load (overview mode)
  - Artifact color-coding with legend
  - Class name search → highlight → auto-center
  - Mouse wheel zoom and canvas pan
  - Node click → show class detail panel (FQCN, artifact, type, modifiers)

### Not in Phase 1

- N-level expansion controls
- Relationship type filtering
- Version management UI
- Analysis trigger from UI

### Deliverables

- `tools/analyzer/` — Java CLI (Maven project) using ClassGraph
- `data/versions/v{initial}/` — JSON output for one Nablarch version
- `src/` — Cytoscape.js frontend (Vanilla JS + Vite)
- `index.html` — entry point for static serving

### Estimated Effort (1 developer)

| Task | Days |
|------|------|
| Java ClassGraph setup, JAR download, GAV extraction | 2 |
| JSON serialization (classes + relations + artifacts + meta) | 1 |
| Cytoscape.js setup + fcose layout + style rules | 2 |
| Search box + highlight + auto-center | 1 |
| Artifact legend + color palette | 1 |
| Performance tuning (texture, hide edges, LOD) | 1 |
| **Total** | **~8 days** |

### Verification Criteria

- [ ] Browser loads without error (Chrome/Firefox)
- [ ] All 2,681 nodes visible after initial render
- [ ] Initial render completes within 10 seconds
- [ ] Each artifact renders in a distinct color; legend is accurate
- [ ] Search for "HttpRequestHandler" highlights the correct node and centers the camera
- [ ] Zoom in/out and pan work smoothly

---

## Phase 2: Version Management

**Goal**: Allow the user to select from multiple analyzed Nablarch versions and trigger a new analysis from the UI.

### Scope

- Analysis pipeline extended: accept version tag as CLI argument; write `meta.json` with `status` field updated at each step.
- Frontend: version selector dropdown; "Analyze new version" button with progress display.
- Deployment: Option A (static + local CLI) or Option B (add lightweight REST server).
- `data/versions/index.json` maintained automatically after each analysis.

### Features

- Version dropdown populated from `data/versions/index.json`
- Version switch: reload JSON without page refresh; preserve zoom level and active filters
- Analysis trigger (Option A): display CLI command in a modal
- Analysis trigger (Option B): POST to `/api/analyze`; poll `/api/status/{version}` every 5 seconds; show progress steps and estimated time
- If analysis fails: show error state in UI; display last 20 lines of log

### Not in Phase 2

- N-level expansion
- `USES`, `CONTAINS`, `DEPENDS` relationship types (still `EXTENDS`/`IMPLEMENTS` only)

### Deliverables

- `tools/analyzer/` updated to write progress to `meta.json`
- `src/ui/version.js` — version selector + analysis trigger component
- (Option B only) `server/index.js` — Node.js REST API wrapper

### Estimated Effort

| Task | Days |
|------|------|
| Update pipeline to write `meta.json` with progress | 1 |
| `data/versions/index.json` maintenance | 0.5 |
| Version selector UI | 1 |
| Version switch (reload JSON, preserve state) | 1 |
| Analysis trigger + progress display (Option A: CLI modal) | 1 |
| (Option B) Node.js server + progress polling | 2 |
| **Total (Option A)** | **~4.5 days** |
| **Total (Option B)** | **~6.5 days** |

### Verification Criteria

- [ ] Two analyzed versions exist; switching between them updates the graph
- [ ] Version selector shows analysis date and class count for each version
- [ ] Analysis trigger (Option A): CLI command is displayed and is correct
- [ ] (Option B) Triggering analysis shows progress steps in UI; completion adds version to selector

---

## Phase 3: Full Exploration UI

**Goal**: Implement the complete N-level expansion, all four relationship type filters, artifact filter, and package hierarchy filter as specified in `requirements.md`.

### Scope

- Analysis pipeline: add `USES`, `CONTAINS`, and `DEPENDS` relationship extraction (ClassGraph field/method analysis).
- Frontend: full exploration controls; lazy-load `relations.json` after initial node render.

### Features

- **N-level expansion**: focal class selection; +1/−1/expand all/reset controls
- **Relationship type filter**: checkbox group for EXTENDS/IMPLEMENTS/USES/CONTAINS/DEPENDS; real-time edge update
- **Artifact filter**: multi-select for artifacts; node show/hide with count indicator
- **Package hierarchy filter**: text input with partial match; filter by `nablarch.fw.*` etc.
- **Overview optimization**: at low zoom, collapse nodes to package-group compound nodes (LOD)
- **Edge count limit guard**: if active filter produces > 5,000 edges, show warning before rendering

### Technical Notes

- `relations.json` may be large (estimated 8,000–15,000 edges with all types). Use lazy loading: load file only after user first activates a filter toggle.
- N-level expansion: maintain a "visible nodes" set in memory; Cytoscape `batch()` calls for bulk show/hide to avoid layout recalculation.
- Package-group LOD: use Cytoscape compound nodes; group children by package prefix. Toggle LOD based on `cy.zoom()` change event.

### Deliverables

- `tools/analyzer/` updated with field/method dependency extraction
- `src/ui/expand.js`, `src/ui/filters.js` — expansion and filter components
- Updated `data-schema.md` with USES/CONTAINS/DEPENDS edge examples
- Performance benchmark report (FPS measurements at 2,681 nodes with all edges visible)

### Estimated Effort

| Task | Days |
|------|------|
| ClassGraph field/method extraction (USES/CONTAINS/DEPENDS) | 2 |
| N-level expansion logic + UI controls | 3 |
| Relationship type filter (checkbox + edge update) | 2 |
| Artifact filter + package filter | 2 |
| LOD compound node implementation | 2 |
| Lazy loading for relations.json | 1 |
| Performance benchmark | 1 |
| **Total** | **~13 days** |

### Verification Criteria

- [ ] Selecting a class and pressing "+1 level" shows only direct neighbors
- [ ] Pressing "−1 level" collapses the outermost ring
- [ ] "Expand all" shows all transitively reachable nodes from focal class
- [ ] "Reset" returns to overview mode
- [ ] Disabling "EXTENDS" filter removes all EXTENDS edges from view
- [ ] Selecting only `nablarch-core` artifact hides all nodes from other artifacts
- [ ] Package filter `nablarch.fw.web` shows only nodes in that package prefix
- [ ] At low zoom, nodes collapse to package groups; zooming in expands them

---

## Phase 4: Team Distribution (Optional)

**Goal**: Package the tool for sharing with a team or for self-hosted deployment.

### Scope

- Docker image with analysis pipeline (Java) + frontend server (Node.js/Nginx)
- Docker Compose file with volume mount for `data/` and workspace
- README with setup instructions
- CI/CD: optional GitHub Actions workflow for scheduled analysis of new Nablarch releases

### Features

- `docker compose up` starts the full stack
- `data/` persists across container restarts via bind mount
- GitHub Actions workflow: triggered on Nablarch release event → runs analysis → commits JSON to repo

### Deliverables

- `Dockerfile` + `docker-compose.yml`
- `docs/deployment.md` — setup and operation guide
- `.github/workflows/analyze-new-release.yml` (optional)

### Estimated Effort

| Task | Days |
|------|------|
| Dockerfile (multi-stage: Java builder + Node server) | 1.5 |
| Docker Compose + volume configuration | 0.5 |
| Deployment documentation | 1 |
| GitHub Actions workflow (optional) | 1 |
| **Total** | **~4 days** |

### Verification Criteria

- [ ] `docker compose up` starts without error on a clean machine
- [ ] Analyzing a version from the UI completes successfully in the container
- [ ] `data/` contents survive `docker compose down` + `docker compose up`
- [ ] (Optional) GitHub Actions workflow triggers on Nablarch release and updates `data/`

---

## Dependencies and Prerequisites

| Prerequisite | Required For | Notes |
|-------------|-------------|-------|
| Java 17+ | Phase 1 (ClassGraph analysis) | Required for ClassGraph 4.8.x |
| Maven 3.8+ | Phase 1 (JAR download) | Used to resolve Nablarch JARs from Maven Central |
| Node.js 20+ | Phase 1 (Vite build) | Frontend build tooling |
| `gh` CLI | Phase 1 (repo list) | `gh repo list nablarch --limit 200` |
| Docker | Phase 4 only | Not needed for Phases 1–3 |
| Nablarch Maven Central artifacts | Phase 1 | Requires network access or local Maven cache |

---

## Risk Checkpoints

| After Phase | Check |
|-------------|-------|
| Phase 1 | Render time < 10s for 2,681 nodes. If not, evaluate Sigma.js or G6 as alternatives before Phase 2. |
| Phase 2 | Version switch latency < 3s. If JSON loading is slow, consider splitting `classes.json` by artifact. |
| Phase 3 | Edge count with all relationship types. If > 20,000, add default filter that shows only EXTENDS+IMPLEMENTS. |
