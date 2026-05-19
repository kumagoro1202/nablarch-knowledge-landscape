# count-classes

Lightweight tool for counting Java type declarations across all repositories in the
[nablarch](https://github.com/nablarch) GitHub organization.

## Purpose

Provide a quick, reproducible measurement of the codebase scale (classes, interfaces, enums,
annotations, records) to support architectural analysis and visualization sizing.

## Dependencies

- `bash` (4.x+)
- `git`
- [`gh`](https://cli.github.com/) CLI (authenticated, `read:org` and `repo` scopes)
- `find`, `grep`, `awk`, `xargs` (POSIX userland)

No language runtimes (Python, Node, etc.) are required.

## Usage

```bash
# Default working directory: /tmp/nablarch-classes
bash run.sh

# Or pass an explicit working directory
bash run.sh /path/to/workdir
```

The script will:

1. Enumerate all public repositories under the `nablarch` GitHub organization (`gh repo list`).
2. Shallow-clone each repository into `<workdir>/repos/<name>` in parallel (max 4 jobs).
   Existing directories are skipped to allow incremental re-runs.
3. Aggregate type declaration counts under `src/main/java` (production) and `src/test/java`
   (test) using line-anchored grep patterns.
4. Emit three artifacts under `output/`:
   - `class-count.csv` — raw per-repository counts
   - `class-count.json` — JSON array (same data)
   - `REPORT.md` — human-readable Markdown summary, per-repo table, top 10, visualization estimates

## Counted Types

For each repository's `src/main/java` directory, the script counts top-level declarations
matching these patterns (regex, line head, capitalized identifier):

| Type | Pattern (simplified) |
|------|----------------------|
| class | `[modifiers] class Xxx` |
| interface | `[modifiers] interface Xxx` |
| enum | `[modifiers] enum Xxx` |
| annotation | `[modifiers] @interface Xxx` |
| record | `[modifiers] record Xxx` |

The same patterns are applied to `src/test/java` for the `total_test` column (counted as
"test classes" only, without per-kind breakdown).

## CSV Schema

```
repo,classes,interfaces,enums,annotations,records,total_main,java_files_main,total_test,java_files_test
```

## Limitations

- Inner / nested types are not counted (only top-level, line-anchored declarations).
- Anonymous classes are not counted.
- Repositories without `src/main/java` are silently skipped.
- The grep pattern may slightly under-count multi-line declarations that wrap modifiers across
  multiple lines.

## Re-running

The clone step is idempotent: existing repository directories are reused. To force a full
re-clone, remove the working directory first:

```bash
rm -rf /tmp/nablarch-classes
bash run.sh
```
