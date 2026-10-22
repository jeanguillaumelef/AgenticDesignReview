# Conventions

Rules shared by every family skill. Each skill reads this file before its own family file.

## Purpose and audience

- The skills audit an **existing repository** for a **senior engineering team**.
- Findings are **advisory**. They are not a gate and never block anything.
- Scope: object-oriented-centred, language-agnostic design, from statement level up to module/package level. Architecture-level checks against a declared target architecture are out of scope for now.
- Only issues that need **judgement** are in the catalogue. Issues a linter or static analyser detects reliably are parked with the reason "better caught by tooling".

## Severity rubric (based on change cost)

| Severity | Meaning |
|---|---|
| **High** | Makes common changes expensive or risky, or breaks a structural boundary (layering, dependency direction). |
| **Medium** | Makes change harder locally; the cost grows if the code is touched often. |
| **Low** | Readability or local design quality; worth fixing opportunistically. |

Each issue has a **default severity**. The finding's final severity may be adjusted by history data and context, as below.

## Severity adjustment from history

If the hotspot file produced by family 07 is present:

- Finding in a **top hotspot** file → raise one level (max High).
- Finding in a file **not changed** during the analysis window → lower one level (min Low).
- Otherwise → default severity.

If the hotspot file is absent, use default severities and say so in the report header. Family 07 findings are never adjusted (they are the hotspot data).

## History analysis rules (family 07)

- The history script **streams and aggregates** `git log` output; the raw log never enters the model's context. Output is a fixed-size summary (top N hotspots, top N co-change pairs/clusters) regardless of history length.
- **Window:** last 12 months by default; overridable in `design-context.md`. State the window used in the report.
- **Excluded commits:** mechanical ones only — rename-only and whitespace-only changes (detectable with git rename detection and whitespace-insensitive diffs).
- **Excluded paths:** generated code, lockfiles, vendored code, migrations, plus anything listed in `design-context.md`.
- **Large commits are kept.** In co-change analysis, each file pair is weighted by commit size (a pair in a 2-file commit counts more than a pair in a 200-file commit), so a single giant commit does not dominate.
- Report the **share of very large commits** in the report header as context about delivery habits, not as a design finding.
- Follow renames so a file's history isn't split.

## Ownership rule

- Every issue has **exactly one owning family**.
- Each family file has a "See also" section listing neighbouring issues owned elsewhere. A skill **must not** report an issue it doesn't own, even if it notices it.
- When two issues describe the same code from two angles (e.g. data class ENC-01 and feature envy COU-02), each owner reports its own angle and may reference the other's ID.

## Project context

Each skill builds context in this order:

1. `design-context.md` at the repository root, if present (see `design-context.template.md`).
2. The README (and any architecture docs it links to), if present.
3. Inference from the code (folder structure, naming, frameworks).

Whatever was inferred rather than declared is **stated as an assumption** at the top of the findings, so the team can correct it in `design-context.md` for the next run. Deliberate exceptions listed in `design-context.md` are not reported.

## Entry format

Each kept issue in a family file has:

- **ID and name**
- **Aliases**
- **Definition** — one line or short paragraph
- **Why it hurts** — the change it makes expensive
- **Detection heuristics** — what to look for in code (or history)
- **False positives** — legitimate cases not to flag
- **Default severity**
- **Source**

Refactoring suggestions are deliberately not included yet.

Parked items have: name, definition, reason parked. Reasons come from a fixed set:

- Better caught by tooling
- Needs runtime information
- Not a code-design issue
- Low value for effort
- Covered by another entry
