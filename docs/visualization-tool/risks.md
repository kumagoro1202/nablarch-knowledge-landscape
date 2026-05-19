# Nablarch Class Visualization Tool — Risk Analysis

**Version**: 1.0  
**Created**: 2026-05-20

---

## Risk Summary Table

| ID | Risk | Probability | Impact | Phase | Mitigation |
|----|------|------------|--------|-------|-----------|
| R01 | Initial render too slow (2,681 nodes) | High | High | Phase 1 | Pre-computed layout, LOD, Canvas renderer |
| R02 | Layout becomes inconsistent between versions | Medium | Medium | Phase 2 | Pre-computed layout anchoring; diff view |
| R03 | Analysis pipeline failure (timeout / network) | Medium | Medium | Phase 1+ | Incremental clone, retry, status tracking |
| R04 | `pom.properties` absent → GAV extraction fails | Low | Medium | Phase 1 | Filename fallback; manual override file |
| R05 | Hairball graph — all edges visible at once | High | High | Phase 3 | Default filter (EXTENDS+IMPLEMENTS only); edge limit guard |
| R06 | Edge count explosion (USES relationships) | High | High | Phase 3 | Lazy loading; per-class USES loading; edge count cap |
| R07 | WebGL / Canvas incompatibility on Safari | Low | Low | Phase 1 | Tested on Chrome/Firefox first; Safari best-effort |
| R08 | Cytoscape.js performance ceiling hit at scale | Medium | High | Phase 1 | Benchmark gate; fallback to Sigma.js/G6 if needed |
| R09 | New Nablarch version breaks ClassGraph scan | Low | Medium | Phase 2+ | Pinned ClassGraph version; version compatibility test |

---

## R01 — Initial Render Performance (2,681 Nodes)

**Risk**: Cytoscape.js with 2,681 nodes and 8,000+ edges may take more than 10 seconds to render, degrading user experience.

**Probability**: High  
**Impact**: High — renders the tool unusable if not addressed

**Background**: Cytoscape.js documentation explicitly warns that performance degrades with graph size increases, especially for edges. The 2,681 node count exceeds the cmd_474 initial estimate (500–2,000) by ~30%.

**Mitigation strategies**:

1. **Pre-computed layout (mandatory)**: Run fcose layout once during analysis; save `x`/`y` coordinates to `classes.json`. On subsequent loads, use `layout: 'preset'` to skip the expensive force-directed computation entirely. Typical render time with preset layout: 1–3 seconds.

2. **Canvas renderer options**:
   ```js
   cytoscape({
     textureOnViewport: true,   // Canvas snapshot during pan/zoom — prevents re-render
     hideEdgesOnViewport: true, // Hide edges during interaction — major perf gain
     motionBlur: true,          // Reduce perceived jank
     pixelRatio: 1              // Avoid DPI scaling overhead
   });
   ```

3. **Level-of-detail (LOD) for initial view**: At zoom < 0.3, render package-group compound nodes instead of individual class nodes. The ~92 repositories map to roughly 50–70 unique package prefixes — a manageable number for the overview.

4. **Edge lazy loading**: Load `relations.json` only after the initial node-only render completes. The user sees nodes immediately; edges appear 1–2 seconds later.

5. **WebGL fallback**: If Cytoscape.js canvas performance is insufficient after all optimizations, switch to Sigma.js (WebGL renderer). The JSON schema is designed to be backend-agnostic, so the data layer does not change.

**Phase 1 gate**: Measure actual render time after initial implementation. If > 10 seconds after applying steps 1–4, escalate to Sigma.js before proceeding to Phase 2.

---

## R02 — Layout Inconsistency Between Versions

**Risk**: When switching from version A to version B, the layout coordinates change significantly (because new classes were added or removed), making it difficult to compare the two versions visually.

**Probability**: Medium (Nablarch changes are incremental but class additions do occur)  
**Impact**: Medium — degrades the version comparison experience

**Mitigation strategies**:

1. **Stable layout anchoring**: When a new version is analyzed, re-use layout coordinates from the previous version for classes that exist in both versions. Only newly added classes receive fresh fcose coordinates.

2. **Diff mode (Phase 3+)**: Color-code nodes/edges that are new, removed, or changed between two selected versions. This makes the layout change meaningful rather than disorienting.

3. **Separate layout per version**: Each version stores its own `x`/`y` in `classes.json`. Switching versions loads the pre-computed layout for that version, which is always stable.

---

## R03 — Analysis Pipeline Failure

**Risk**: The analysis pipeline fails mid-run due to network timeout (GitHub API rate limit, slow clone), Maven Central unavailability, or ClassGraph scan error.

**Probability**: Medium (GitHub API limits apply; 92 repos × shallow clone is non-trivial)  
**Impact**: Medium — analysis must be retried; partial output is discarded

**Mitigation strategies**:

1. **Incremental clone**: Reuse existing shallow clones from previous runs. Only re-clone if the version tag differs.

2. **`meta.json` progress tracking**: Write status at each pipeline step (`queued → cloning → analyzing → generating → done`). On retry, skip steps whose output already exists.

3. **GitHub API rate limit**: Use `gh auth login` to authenticate; authenticated requests have 5,000 req/hour limit (vs. 60 for anonymous).

4. **Timeout handling**: Set a 15-minute overall timeout for the analysis job. If exceeded, set `status: "failed"` with an error message.

5. **Partial retry**: If cloning fails for specific repositories (e.g., archived or empty), log the failure and continue with the rest. Record skipped repos in `meta.json`.

---

## R04 — `pom.properties` Absent → GAV Extraction Fails

**Risk**: Nablarch JARs may not contain `META-INF/maven/*/pom.properties`, preventing ClassGraph from extracting `groupId`/`artifactId`/`version`.

**Probability**: Low (Maven standard JARs include this file by default)  
**Impact**: Medium — artifact-based color-coding fails; all nodes appear in the same color

**Mitigation**:

1. **Filename fallback**: Parse the JAR filename. `nablarch-fw-web-6.0.0.jar` → `artifactId=nablarch-fw-web`, `version=6.0.0`.

2. **Manual override file**: Maintain `tools/analyzer/artifact-overrides.json` that maps JAR filenames to GAV coordinates for edge cases.

3. **Verification step**: During Phase 1 implementation, enumerate all JAR files and verify `pom.properties` presence before committing to the approach.

---

## R05 — Hairball Graph: All Edges Visible Simultaneously

**Risk**: With 2,681 nodes and potentially 8,000–15,000 edges, displaying all edges at once produces an illegible "hairball" graph where individual relationships cannot be distinguished.

**Probability**: High (this is a well-known problem in class hierarchy visualization)  
**Impact**: High — the tool becomes visually useless in its default state

**Mitigation**:

1. **Default filter: EXTENDS + IMPLEMENTS only**: The initial view shows only inheritance and interface implementation relationships. USES, CONTAINS, and DEPENDS are opt-in (checkboxes off by default). This reduces the initial edge count to an estimated 500–2,000.

2. **N-level expansion**: In non-overview mode, only edges connected to the focal class's expanded neighborhood are shown. The rest are hidden.

3. **Minimum zoom edge hiding**: At zoom < 0.15 (full overview), hide all edges. The overview shows the node distribution only; edges become visible when zoomed in.

4. **Edge count guard**: Before rendering a filter change that would produce > 5,000 visible edges, display a warning: "This will show N edges. Performance may be slow. Continue?" with a confirm/cancel button.

---

## R06 — Edge Count Explosion (USES Relationships)

**Risk**: `USES` relationships (import references, field references, method calls) can produce tens of thousands of edges, far exceeding the manageable range for interactive rendering.

**Probability**: High (a single class may import 20+ other classes)  
**Impact**: High — JSON file size explosion; render freeze

**Mitigation**:

1. **Selective extraction**: In Phase 3, extract only **field type dependencies** (not method call dependencies) for `USES`. This limits edges to the most structurally significant relationships.

2. **Lazy loading per class**: Instead of loading all `USES` edges globally, load them only when a class is selected as the focal node (REST API or on-demand JSON chunk).

3. **Edge count cap**: Hard limit at 20,000 edges in `relations.json`. If the cap is exceeded during analysis, log which relationship types were dropped.

4. **Phase gating**: `USES`/`CONTAINS`/`DEPENDS` extraction is Phase 3 work. Phase 1 and 2 operate with only `EXTENDS`/`IMPLEMENTS`, which is known to be manageable.

---

## R07 — Safari Browser Compatibility

**Risk**: Safari (especially versions < 16) has known limitations with Canvas API features used by Cytoscape.js, and WebGL2 support was limited before Safari 15.

**Probability**: Low (the primary user is on a modern browser)  
**Impact**: Low — affects only Safari users

**Mitigation**: Test on Chrome and Firefox first. Safari is best-effort. Note explicitly in the README that Chrome/Firefox are the supported browsers.

---

## R08 — Cytoscape.js Performance Ceiling

**Risk**: Even with all Canvas optimizations applied, Cytoscape.js may not achieve the 10-second render target at 2,681 nodes on lower-spec hardware.

**Probability**: Medium  
**Impact**: High — may require switching to Sigma.js (3–5 day rework)

**Mitigation**:

1. **Benchmark in Phase 1**: Measure actual frame times on a reference machine (target: modern laptop, 8GB RAM, no GPU).

2. **Fallback plan prepared**: The data schema is frontend-agnostic. If Cytoscape.js fails the benchmark, migrate the rendering layer to Sigma.js. The JSON loading, filter logic, and version management code remain unchanged.

3. **Success criterion**: Render time < 10s AND pan/zoom > 30 FPS with 2,681 nodes. If either fails, escalate to Sigma.js.

---

## R09 — ClassGraph Incompatibility with Future Nablarch Versions

**Risk**: A future Nablarch version may use Java features (e.g., sealed classes, records with complex generics) that ClassGraph cannot parse correctly, producing incorrect or missing relationships.

**Probability**: Low (ClassGraph is actively maintained and tracks Java releases)  
**Impact**: Medium — analysis output is silently incomplete

**Mitigation**:

1. **Pinned ClassGraph version**: Use a fixed version in `pom.xml` (currently `4.8.184`). Update only after verifying compatibility.

2. **Class count validation**: After analysis, compare `meta.json.total_classes` against the `class-count.csv` from the count-classes tool. A > 10% discrepancy triggers a warning.

3. **Changelog monitoring**: Subscribe to ClassGraph GitHub releases to catch breaking changes early.
