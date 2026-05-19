# Nablarch Class Visualization Tool — Requirements Definition

**Version**: 1.0  
**Created**: 2026-05-20  
**Based on**: Prior technology selection research and class count measurement results

---

## 1. Background

The Nablarch framework is distributed across 118 repositories (92 with production Java source). As of 2026-05-19, it contains **2,681 type declarations** (classes, interfaces, enums, annotations, records) across 92 repositories. Understanding class relationships — inheritance, implementation, and usage — is essential for onboarding, architecture reviews, and code reading.

A single static diagram is not practical at this scale. This tool provides an **interactive exploration view** that allows the user to start from a high-level overview and drill into specific relationships on demand.

---

## 2. User Stories

| ID | Story |
|----|-------|
| US-01 | As a developer, I want to see all Nablarch classes in one canvas so that I can get an overview of the architecture. |
| US-02 | As a developer, I want to select a class and expand its related classes step by step so that I can trace a specific inheritance chain without visual noise. |
| US-03 | As a developer, I want to filter by relationship type so that I can focus on inheritance vs. usage patterns separately. |
| US-04 | As a developer, I want to filter by artifact (JAR) so that I can focus on a specific module like `nablarch-fw-web`. |
| US-05 | As a developer, I want to search for a class by name and have it highlighted and centered so that I can quickly find it. |
| US-06 | As a developer, I want to select the Nablarch version I'm interested in so that I can see class relationships for that version. |
| US-07 | As a developer, I want to trigger an analysis for a new version so that I can see up-to-date class data without manual steps. |

---

## 3. Functional Requirements

### 3.1 Exploration View

#### 3.1.1 Initial Overview

- **REQ-V01**: On load, display all classes in a single canvas with a package-group or artifact-group simplified layout.
- **REQ-V02**: At overview scale, individual class labels may be hidden or abbreviated to maintain readability with 2,681+ nodes.
- **REQ-V03**: Each node is color-coded by artifact (JAR). A color legend is displayed persistently.
- **REQ-V04**: The initial layout must render and become interactive within **10 seconds** on a modern laptop.

#### 3.1.2 Class Selection and N-Level Expansion

- **REQ-V05**: Clicking a node selects it as the "focal class" and expands its directly related classes (N=1).
- **REQ-V06**: Expansion controls:
  - **+1 level**: expand one additional hop from current focal class
  - **−1 level**: collapse the outermost ring
  - **Expand all**: show all transitively reachable classes from the focal class
  - **Reset**: return to the initial overview
- **REQ-V07**: Non-expanded nodes are not removed from the canvas; they are dimmed or hidden based on user preference.

#### 3.1.3 Relationship Type Filtering

- **REQ-V08**: The user can filter displayed edges by relationship type. At least four types are supported:
  1. **Inheritance / Implementation** (`extends` / `implements`)
  2. **Usage** (`import` / field reference / method call)
  3. **Containment** (inner class / nested class)
  4. **Dependency** (constructor argument type / field type)
- **REQ-V09**: Multiple relationship types can be active simultaneously.
- **REQ-V10**: When a filter is changed, the graph updates without a full reload.

#### 3.1.4 Artifact Filtering

- **REQ-V11**: The user can select one or more artifacts (e.g., `nablarch-fw-batch`, `nablarch-fw-web`, `nablarch-core`) to limit the displayed nodes.
- **REQ-V12**: Artifact selection is a multi-select control. "All" is the default.
- **REQ-V13**: The filter list is derived from the analysis data, not hardcoded.

#### 3.1.5 Package Hierarchy Filtering

- **REQ-V14**: The user can filter nodes by package prefix (e.g., `nablarch.fw.*`).
- **REQ-V15**: A text input supporting partial match or glob patterns is acceptable.

#### 3.1.6 Search and Highlight

- **REQ-V16**: A search box allows entry of a class name or package name (partial match supported).
- **REQ-V17**: Matching nodes are highlighted and the camera automatically centers on the first match.
- **REQ-V18**: If multiple matches exist, the user can cycle through them.
- **REQ-V19**: Search input is debounced (≥ 200ms) to avoid excessive re-renders.

#### 3.1.7 Navigation

- **REQ-V20**: Mouse wheel zoom is supported with configurable min/max zoom levels.
- **REQ-V21**: Canvas pan is supported via click-and-drag on empty space.
- **REQ-V22**: Node drag is supported to allow manual rearrangement.

### 3.2 Version Management

#### 3.2.1 Version Selection

- **REQ-M01**: The user can select the Nablarch version to view from a dropdown of analyzed versions.
- **REQ-M02**: "Version" is defined as a Nablarch release tag (e.g., `v5.4.0`) or a Git commit SHA.
- **REQ-M03**: The UI displays the currently selected version and its analysis timestamp.

#### 3.2.2 Analysis Execution

- **REQ-M04**: A button labeled "Analyze this version" triggers the analysis pipeline for the specified version.
- **REQ-M05**: While analysis is running, a progress indicator shows the current state:
  - States: `Queued` → `Cloning` → `Analyzing` → `Generating JSON` → `Done` / `Failed`
  - Estimated duration is displayed (target: under 15 minutes for full analysis).
- **REQ-M06**: If analysis fails, an error message and partial log excerpt are shown.
- **REQ-M07**: On completion, the new version becomes available in the version selector.

#### 3.2.3 Version Switching

- **REQ-M08**: The user can switch between analyzed versions without re-triggering analysis.
- **REQ-M09**: Version switch reloads graph data but preserves UI state (zoom level, active filters) where possible.

#### 3.2.4 Analysis Result Persistence

- **REQ-M10**: Analysis output is stored under `data/versions/{version}/` as JSON files (see Data Schema).
- **REQ-M11**: Stored results persist across browser sessions and application restarts.
- **REQ-M12**: A list of analyzed versions with their metadata (analysis date, class count, duration) is maintained in `data/versions/index.json`.

---

## 4. Non-Functional Requirements

| ID | Category | Requirement |
|----|----------|-------------|
| NFR-01 | Performance | Initial render of 2,681 nodes must complete within 10 seconds on a modern laptop (Chrome/Firefox). |
| NFR-02 | Performance | Relationship filter toggle must respond within 1 second. |
| NFR-03 | Performance | Class search result must appear within 500ms. |
| NFR-04 | Scalability | The design must support up to 5,000 nodes without architectural changes (anticipating class growth). |
| NFR-05 | Browser support | Chrome (latest), Firefox (latest), Edge (latest). Safari is best-effort. |
| NFR-06 | Offline | The rendering frontend must function without network access after initial load (data is local JSON). |
| NFR-07 | Portability | The tool must run on macOS, Windows (WSL2), and Linux without OS-specific changes. |

---

## 5. Out of Scope

- Real-time code analysis (analysis is always a pre-step, results are persisted as JSON)
- Modification of Nablarch source code
- Method-level or field-level dependency visualization (class-level only in this version)
- Integration with IDE tools
- Mobile / smartphone support (desktop and tablet only)

---

## 6. Glossary

| Term | Definition |
|------|-----------|
| **Node** | A single type declaration (class, interface, enum, annotation, or record) |
| **Edge** | A directed relationship between two nodes |
| **Artifact** | A Maven JAR artifact (e.g., `nablarch-fw-web`) |
| **Version** | A Nablarch release tag or commit SHA used as the analysis target |
| **Focal class** | The node currently selected as the center of N-level expansion |
| **Overview mode** | The initial canvas state showing all nodes without expansion filtering |
