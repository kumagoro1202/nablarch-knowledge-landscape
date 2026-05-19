# Nablarch Class Visualization Tool — System Architecture

**Version**: 1.0  
**Created**: 2026-05-20  
**Technology basis**: Prior technology selection research — Rank 1: Cytoscape.js × ClassGraph

---

## 1. Architecture Overview

The system is divided into two independent components with a clear contract between them:

```
┌─────────────────────────────────────────────────────────────┐
│  Analysis Pipeline (Backend / CLI)                          │
│                                                             │
│  Input:  Nablarch version tag (e.g. v5.4.0)                 │
│  Steps:  Clone repos → ClassGraph scan → JSON generation    │
│  Output: data/versions/{version}/ (4 JSON files)            │
└───────────────────────────┬─────────────────────────────────┘
                            │ JSON files (local filesystem)
┌───────────────────────────▼─────────────────────────────────┐
│  Rendering Frontend (Browser)                               │
│                                                             │
│  Input:  data/versions/{version}/*.json                     │
│  Steps:  Load JSON → Cytoscape.js render → Exploration UI   │
│  Output: Interactive graph in browser                       │
└─────────────────────────────────────────────────────────────┘
```

**Key design principle**: The analysis pipeline and rendering frontend share no runtime dependency. The pipeline is a one-time batch job; the frontend is a static web application that reads pre-computed data.

---

## 2. Analysis Pipeline

### 2.1 Inputs

- **Nablarch version**: A release tag (e.g., `v5.4.0`) or a commit SHA from the `nablarch` GitHub organization.
- **Repository list**: Derived at runtime via `gh repo list nablarch --limit 200` (or cached in `data/repo-list.json`).

### 2.2 Processing Steps

```
Step 1: Resolve repository list
        gh repo list nablarch → filter repos with src/main/java

Step 2: Clone / update repositories at the target version
        git clone --depth 1 --branch {version} {repo_url} {work_dir}/{repo}
        (shallow clone; reuse existing clone if version matches)

Step 3: Collect JAR artifacts
        mvn dependency:copy-dependencies -DoutputDirectory={work_dir}/jars
        (or: download from Maven Central via coordinates in pom.xml)

Step 4: ClassGraph scan
        ClassGraph().overrideClasspath(jarList)
                    .enableAllInfo()
                    .scan()
        → Extract: ClassInfo list with superclass, interfaces, fields, methods

Step 5: GAV resolution
        For each JAR: read META-INF/maven/*/pom.properties
        → groupId, artifactId, version
        Fallback: parse JAR filename (e.g. nablarch-fw-web-6.0.jar)

Step 6: JSON generation
        → classes.json   (node data)
        → relations.json (edge data)
        → artifacts.json (artifact metadata + color assignments)
        → meta.json      (analysis metadata)

Step 7: Update index
        → data/versions/index.json (append new version entry)
```

### 2.3 Implementation Options

The pipeline can be implemented as:

| Option | Technology | Notes |
|--------|-----------|-------|
| CLI script | Java (Maven plugin) or Bash + `mvn` | Simplest; runs locally on demand |
| REST API endpoint | Node.js or Python (Flask/FastAPI) | Required for Option B/C deployment |
| GitHub Actions workflow | YAML-defined CI job | Useful for scheduled analysis of new releases |

For the PoC phase, a **CLI script** is sufficient. The REST API wrapper is added in Phase 2.

### 2.4 Estimated Processing Time

Based on 92 repositories and 2,681 type declarations:

| Step | Estimated Duration |
|------|--------------------|
| Repo clone (92 × shallow) | 5–10 min (network dependent) |
| ClassGraph scan (all JARs) | 30–60 sec |
| JSON generation | < 10 sec |
| **Total** | **~10–15 min** |

Subsequent runs reuse existing clones and skip unchanged repositories, reducing time to < 2 minutes for incremental updates.

---

## 3. Rendering Frontend

### 3.1 Technology Stack

| Component | Technology | Rationale |
|-----------|-----------|----------|
| Graph rendering | **Cytoscape.js** (MIT) | Full REQ coverage, 2–3 day learning curve, rich plugin ecosystem |
| Layout engine | **cytoscape-fcose** plugin | Handles 1,000–3,000 nodes; force-directed with compound node support |
| Search / highlight | **cytoscape-view-utilities** plugin | `highlightNeighbors()` and `zoomToSelected()` out of the box |
| UI framework | **Vanilla JS** (Phase 1–3) / React (Phase 4+) | Minimizes dependencies for PoC; React only if team collaboration requires it |
| Build tool | **Vite** | Fast HMR, ES modules, minimal config |
| Data loading | `fetch()` + JSON | Static files; no backend required for frontend-only deployments |

### 3.2 Frontend Component Structure

```
src/
├── index.html           # Single-page app shell
├── main.js              # Entry point: load data, initialize Cytoscape
├── graph/
│   ├── init.js          # Cytoscape instance setup, layout config
│   ├── style.js         # Node/edge style rules (color by artifact, edge types)
│   ├── layout.js        # Layout selection and pre-computed coordinate fallback
│   └── events.js        # Click, hover, zoom event handlers
├── ui/
│   ├── search.js        # Search box with debounce + highlight
│   ├── filters.js       # Relationship type and artifact filter controls
│   ├── expand.js        # N-level expansion controls (+1/−1/all/reset)
│   ├── version.js       # Version selector and analysis trigger
│   └── legend.js        # Artifact color legend
└── data/
    └── loader.js        # Load and validate JSON from data/versions/{version}/
```

### 3.3 Performance Strategy for 2,681 Nodes

| Technique | Detail |
|-----------|--------|
| **Pre-computed layout** | Run fcose once, save node coordinates to `classes.json`. On load, use `layout: 'preset'` to skip expensive calculation. |
| **Texture on viewport** | Enable `textureOnViewport: true` in Cytoscape options during pan/zoom. |
| **Hide edges on viewport** | Enable `hideEdgesOnViewport: true` during interaction; restore on idle. |
| **Level-of-detail (LOD)** | At zoom < threshold, render package-group compound nodes instead of individual class nodes. |
| **Lazy relationship loading** | Load `relations.json` only after initial render. Initial view shows nodes only; edges load in background. |

### 3.4 Version Management UI

```
┌────────────────────────────────────────────┐
│ Version: [v5.4.0 ▼]  [Analyze new version] │
│ Analyzed: 2026-05-19  Classes: 2,681        │
└────────────────────────────────────────────┘
```

- Version selector is populated from `data/versions/index.json`.
- "Analyze new version" opens a modal: input box for version tag → calls the analysis API (Option B/C) or shows CLI command (Option A).
- Progress is polled from `data/versions/{version}/meta.json` (status field).

---

## 4. Deployment Options

### Option A: Static File Distribution (Recommended for PoC and personal use)

```
nablarch-knowledge-landscape/
├── index.html         (frontend entry point)
├── assets/            (Vite build output: JS, CSS)
└── data/
    └── versions/
        ├── index.json
        └── v5.4.0/
            ├── classes.json
            ├── relations.json
            ├── artifacts.json
            └── meta.json
```

- Analysis is run **locally** as a CLI command before serving.
- Frontend is served via `npx serve .` or any static file server (Python `http.server`, Nginx, etc.).
- GitHub Pages deployment: push `data/` and built assets to the repo; analysis output is committed to the repo.
- **No server process required at runtime.**
- Analysis trigger UI is replaced with a "Run analysis locally" instruction.

**Pros**: Zero infrastructure cost, fully offline after initial data generation, trivial to share (zip and send).  
**Cons**: Analysis must be run manually; version updates require local CLI execution and re-deploy.

---

### Option B: Backend Server

```
┌──────────────┐        REST API         ┌────────────────────┐
│   Browser    │ ←───────────────────── │  Backend Server    │
│  (Vite SPA)  │                        │  Node.js / Python  │
└──────────────┘                        │  ├── /api/versions  │
                                        │  ├── /api/analyze   │
                                        │  └── /api/status    │
                                        └─────────┬──────────┘
                                                  │ spawns
                                        ┌─────────▼──────────┐
                                        │  Analysis CLI      │
                                        │  (Java / Bash)     │
                                        └────────────────────┘
```

- Analysis is triggered via the UI: POST `/api/analyze?version=v5.4.0` → server spawns the CLI process.
- Progress is polled: GET `/api/status/{version}` → returns current step and log tail.
- JSON data is served from the same server: GET `/data/versions/v5.4.0/classes.json`.
- **Requires a persistent server process.**

**Pros**: Full UI-driven workflow; no local CLI needed.  
**Cons**: Requires server setup (Node.js/Python environment, process management).

---

### Option C: Docker Container

```
docker run -p 8080:8080 -v ./data:/app/data nablarch-viz:latest
```

- The Docker image bundles the backend server (Option B) and the frontend.
- `data/` is mounted as a volume for persistence across container restarts.
- Docker Compose variant includes a volume for analysis workspace.

**Pros**: Single-command startup; reproducible environment; easy team distribution.  
**Cons**: Requires Docker on the host machine; image build step adds setup time.

---

### Recommendation

| Scenario | Recommended Option |
|----------|--------------------|
| Personal PoC (Phase 1–2) | **Option A** — zero infrastructure, fastest to start |
| Single developer, want UI-driven analysis | **Option B** — lightweight server on localhost |
| Team sharing / consistent environment | **Option C** — Docker Compose with mounted data volume |

**For the PoC (Phase 1), Option A is the recommended starting point.** The architecture is designed so that Option B/C can be added later without changing the frontend or data schema.

---

## 5. Data Flow Summary

```
[Nablarch GitHub repos]
        ↓ git clone (shallow, per version)
[Local working directory: /tmp/nablarch-viz-work/]
        ↓ Maven build → JAR collection
[JAR files: nablarch-fw-web-6.0.jar, nablarch-core-6.0.jar, ...]
        ↓ ClassGraph scan + pom.properties GAV extraction
[Java objects: ClassInfo, with superclass/interfaces/fields]
        ↓ JSON serialization (custom adapter)
[data/versions/v5.4.0/classes.json + relations.json + artifacts.json + meta.json]
        ↓ fetch() in browser
[Cytoscape.js graph: nodes + edges]
        ↓ user interaction
[Filtered subgraph: N-level expansion, relationship type filter, artifact filter]
```
