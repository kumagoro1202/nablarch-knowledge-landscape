# Nablarch Class Visualization Tool — Data Schema

**Version**: 1.0  
**Created**: 2026-05-20

---

## Overview

All analysis output is stored under `data/versions/{version}/` as four JSON files:

```
data/versions/
├── index.json            ← Version index (updated after each analysis)
└── v5.4.0/
    ├── classes.json      ← Node data (one entry per type declaration)
    ├── relations.json    ← Edge data (one entry per relationship)
    ├── artifacts.json    ← Artifact metadata and color assignments
    └── meta.json         ← Analysis metadata and status
```

---

## 1. `classes.json`

Describes each type declaration (class, interface, enum, annotation, record).

### Schema

```json
{
  "nodes": [
    {
      "id":          "string",   // Fully qualified class name (FQCN) — unique key
      "fqcn":        "string",   // Same as id; explicit field for clarity
      "simpleName":  "string",   // Short class name without package
      "artifactId":  "string",   // Maven artifactId of the containing JAR
      "package":     "string",   // Package name (e.g., "nablarch.fw.web")
      "type":        "string",   // One of: CLASS | INTERFACE | ENUM | ANNOTATION | RECORD
      "modifiers":   ["string"], // e.g., ["public", "abstract"]
      "x":           number,     // Pre-computed layout X coordinate (null if not yet computed)
      "y":           number      // Pre-computed layout Y coordinate (null if not yet computed)
    }
  ]
}
```

### Sample

```json
{
  "nodes": [
    {
      "id":          "nablarch.fw.web.HttpRequestHandler",
      "fqcn":        "nablarch.fw.web.HttpRequestHandler",
      "simpleName":  "HttpRequestHandler",
      "artifactId":  "nablarch-fw-web",
      "package":     "nablarch.fw.web",
      "type":        "INTERFACE",
      "modifiers":   ["public"],
      "x":           142.5,
      "y":           -88.3
    },
    {
      "id":          "nablarch.fw.web.servlet.ServletExecutionContext",
      "fqcn":        "nablarch.fw.web.servlet.ServletExecutionContext",
      "simpleName":  "ServletExecutionContext",
      "artifactId":  "nablarch-fw-web",
      "package":     "nablarch.fw.web.servlet",
      "type":        "CLASS",
      "modifiers":   ["public"],
      "x":           210.0,
      "y":           -45.1
    },
    {
      "id":          "nablarch.fw.ExecutionContext",
      "fqcn":        "nablarch.fw.ExecutionContext",
      "simpleName":  "ExecutionContext",
      "artifactId":  "nablarch-core",
      "package":     "nablarch.fw",
      "type":        "CLASS",
      "modifiers":   ["public"],
      "x":           180.0,
      "y":           0.0
    }
  ]
}
```

### Notes

- `id` and `fqcn` are identical. Both fields exist so that consumers can use either convention.
- `x` and `y` are `null` after the first analysis run. They are populated after the first layout computation in the frontend (the result is saved back to the file for subsequent loads).
- `modifiers` may be empty (`[]`) for package-private types.

---

## 2. `relations.json`

Describes directed relationships between type declarations.

### Schema

```json
{
  "edges": [
    {
      "from":          "string",  // FQCN of the source node
      "to":            "string",  // FQCN of the target node
      "relation_type": "string",  // One of: EXTENDS | IMPLEMENTS | USES | CONTAINS | DEPENDS
      "detail":        "string"   // Optional: additional detail (e.g., method name for USES)
    }
  ]
}
```

### Relation Types

| Value | Meaning | Example |
|-------|---------|---------|
| `EXTENDS` | Class extends another class | `FooImpl extends FooBase` |
| `IMPLEMENTS` | Class implements an interface | `FooImpl implements FooInterface` |
| `USES` | Import / field reference / method call | `BarService` references `BazRepository` |
| `CONTAINS` | Inner class or nested class | `Outer` contains `Outer.Inner` |
| `DEPENDS` | Constructor argument type or field type | `Service(Repository repo)` → `DEPENDS` on `Repository` |

### Sample

```json
{
  "edges": [
    {
      "from":          "nablarch.fw.web.servlet.ServletExecutionContext",
      "to":            "nablarch.fw.ExecutionContext",
      "relation_type": "EXTENDS",
      "detail":        ""
    },
    {
      "from":          "nablarch.fw.web.servlet.ServletExecutionContext",
      "to":            "nablarch.fw.web.HttpRequestHandler",
      "relation_type": "IMPLEMENTS",
      "detail":        ""
    },
    {
      "from":          "nablarch.fw.web.action.WebFrontController",
      "to":            "nablarch.fw.web.servlet.ServletExecutionContext",
      "relation_type": "DEPENDS",
      "detail":        "field: executionContext"
    }
  ]
}
```

### Notes

- Edges are directed: `from` → `to`.
- The `detail` field is optional and may be an empty string. It is intended for debugging and tooltip display, not for filtering logic.
- `USES` relationships may produce a large number of edges. In the PoC phase, only `EXTENDS` and `IMPLEMENTS` are extracted (Phase 1); `USES`, `CONTAINS`, and `DEPENDS` are added in Phase 3.

---

## 3. `artifacts.json`

Describes each Maven artifact and its visual representation.

### Schema

```json
{
  "artifacts": [
    {
      "artifactId":  "string",  // Maven artifactId
      "groupId":     "string",  // Maven groupId
      "version":     "string",  // Maven version of this analysis
      "repository":  "string",  // GitHub repository name (e.g., "nablarch/nablarch-fw-web")
      "colorHex":    "string"   // Hex color for nodes belonging to this artifact (e.g., "#4E79A7")
    }
  ]
}
```

### Sample

```json
{
  "artifacts": [
    {
      "artifactId":  "nablarch-fw-web",
      "groupId":     "com.nablarch.framework",
      "version":     "6.0.0",
      "repository":  "nablarch/nablarch-fw-web",
      "colorHex":    "#4E79A7"
    },
    {
      "artifactId":  "nablarch-core",
      "groupId":     "com.nablarch.framework",
      "version":     "6.0.0",
      "repository":  "nablarch/nablarch-core",
      "colorHex":    "#F28E2B"
    },
    {
      "artifactId":  "nablarch-fw-batch",
      "groupId":     "com.nablarch.framework",
      "version":     "6.0.0",
      "repository":  "nablarch/nablarch-fw-batch",
      "colorHex":    "#E15759"
    }
  ]
}
```

### Notes

- Color assignment is deterministic: colors are assigned from the Tableau 10 palette in alphabetical artifact order. This ensures stable colors across re-runs.
- If `pom.properties` is not found in a JAR, `groupId` is set to `"unknown"` and `version` is parsed from the JAR filename.

---

## 4. `meta.json`

Stores analysis metadata and status for the version manager UI.

### Schema

```json
{
  "nablarch_version":  "string",    // Version tag or commit SHA used as analysis target
  "analyzed_at":       "string",    // ISO 8601 timestamp (e.g., "2026-05-20T10:30:00Z")
  "commit_sha":        "string",    // Git commit SHA of the main nablarch repo at analysis time
  "total_classes":     number,      // Total node count in classes.json
  "total_relations":   number,      // Total edge count in relations.json
  "total_artifacts":   number,      // Total artifact count in artifacts.json
  "duration_seconds":  number,      // Wall-clock time for the analysis pipeline
  "tool_version":      "string",    // Version of this visualization tool
  "status":            "string",    // One of: queued | cloning | analyzing | generating | done | failed
  "error_message":     "string"     // Non-empty only when status = "failed"
}
```

### Sample (analysis complete)

```json
{
  "nablarch_version":  "v5.4.0",
  "analyzed_at":       "2026-05-20T10:30:00Z",
  "commit_sha":        "a1b2c3d4e5f6...",
  "total_classes":     2681,
  "total_relations":   8432,
  "total_artifacts":   92,
  "duration_seconds":  487,
  "tool_version":      "1.0.0",
  "status":            "done",
  "error_message":     ""
}
```

### Sample (analysis in progress)

```json
{
  "nablarch_version":  "v5.5.0",
  "analyzed_at":       "",
  "commit_sha":        "",
  "total_classes":     0,
  "total_relations":   0,
  "total_artifacts":   0,
  "duration_seconds":  0,
  "tool_version":      "1.0.0",
  "status":            "cloning",
  "error_message":     ""
}
```

---

## 5. `data/versions/index.json`

Top-level index of all analyzed versions. Updated atomically after each successful analysis.

### Schema

```json
{
  "versions": [
    {
      "version":       "string",  // Version tag
      "analyzed_at":   "string",  // ISO 8601 timestamp
      "total_classes": number,    // For display in the version selector dropdown
      "status":        "string"   // done | failed
    }
  ]
}
```

### Sample

```json
{
  "versions": [
    {
      "version":       "v5.4.0",
      "analyzed_at":   "2026-05-20T10:30:00Z",
      "total_classes": 2681,
      "status":        "done"
    },
    {
      "version":       "v5.3.0",
      "analyzed_at":   "2026-04-15T08:00:00Z",
      "total_classes": 2598,
      "status":        "done"
    }
  ]
}
```

---

## 6. Design Constraints

1. All JSON files use **UTF-8 encoding** without BOM.
2. All timestamps use **ISO 8601 format** with UTC timezone (`Z` suffix).
3. The `id` field in `classes.json` and the `from`/`to` fields in `relations.json` use the **FQCN** as the primary key. This ensures uniqueness across artifacts.
4. `colorHex` values in `artifacts.json` are **7-character hex strings** (e.g., `"#4E79A7"`), not CSS named colors.
5. The schema is designed to be **append-only** for new versions: existing version directories are never overwritten.
