---
name: design-review-coupling
description: Audit a repository for coupling design issues - Law of Demeter violations (COU-01), feature envy (COU-02), inappropriate intimacy (COU-03) and middle man (COU-04). Use when the user asks for a coupling review or design audit of a codebase. Runs standalone or can be called by an aggregator skill. Read-only and advisory.
---

# Design review: coupling family

Audit the repository in the working directory for the coupling family. You never modify code. Findings are advisory.

## 1. Load the rules (do not restate them from memory)

Read both, in this order. They live in the plugin, not in the repo under review, so use the plugin root path:

1. `${CLAUDE_PLUGIN_ROOT}/design-issues/conventions.md` - severity rubric, hotspot adjustment, ownership rule, context order.
2. `${CLAUDE_PLUGIN_ROOT}/design-issues/families/01-coupling.md` - the issues you own: COU-01 to COU-04, with heuristics and false positives.

If `${CLAUDE_PLUGIN_ROOT}` is not substituted in this text, locate the plugin directory (the one containing `.claude-plugin/plugin.json` and `design-issues/`) and read the files from there.

Report only COU-01 to COU-04. Never report, mention or hint at issues owned by other families, even the ones listed in "See also".

## 2. Build context

Read only what matters for coupling:

- `design-context.md` at the repo root, if present: **Architecture and layers**, **Intended design style**, **Deliberate exceptions**, **Out of scope**. Ignore the other sections.
- Then the README and any architecture docs it links to.
- Then infer from code (folder structure, naming, frameworks).

Rules:

- Treat the declared design style as a false-positive filter (e.g. functional style weakens COU-02; a mandated application-service layer weakens COU-04).
- A deliberate-exception row matching a COU ID and a path is **suppressed silently**. Do not report it or mention it.
- Skip paths listed under Out of scope.
- Anything inferred rather than declared goes in the header as an assumption.
- Detect the primary language(s) and state them in the header. Apply the catalogue's false-positive list using your own knowledge of the language's idioms (fluent builders, stream/LINQ pipelines, promise chains, etc.). There are no per-language reference files.

## 3. Choose what to read

Check for `design-review/hotspots.json` (contract in `conventions.md`).

1. **Exists:** read the top-ranked hotspot files first, then widen as budget allows.
2. **Missing, git repo:** take the top 50 most recently changed source files, e.g. `git log --name-only --pretty=format: -n 500`, de-duplicated, excluding generated code, lockfiles, vendored code and migrations. Never load the raw log into context beyond this de-duplicated list.
3. **Missing, no git:** take a capped sample ordered by file size and spread across directories.

State the strategy used. Without a hotspot file, use default severities and say so in the header.

## 4. Find candidates, then judge

Detection is light. Use Grep/Glob/Read to find candidates, then judge each one yourself against the issue's heuristics and false positives. No scripts.

- **COU-01:** grep for chained calls (`)\.\w+\(` repeated, getter chains), then read the context. Discard fluent builders, stream pipelines, DTO/JSON/config navigation, and framework idioms.
- **COU-02:** find methods dominated by getters on one foreign object. Discard mappers, presenters, serialisers, visitors, strategies.
- **COU-03:** look for reflection, internal/package-private access, bidirectional calls, direct manipulation of another class's collections, and subclass reliance on parent internals. Discard nested classes, aggregates with their entities, tree-node links.
- **COU-04:** find classes where more than half of the public methods are one-line delegations to one field. Discard facades, adapters, proxies, decorators, anti-corruption layers and architecture-mandated layers.

## 5. Evidence and confidence

- Every finding cites at least one concrete `file:line` and the symbols involved. No evidence, no finding.
- **COU-01:** report only if the same chain prefix repeats, or the chain crosses 3 or more domain objects. A lone 2-call chain is not reported.
- **Confidence:**
  - High - heuristics clearly match and no false-positive case applies.
  - Medium - matches, but a false-positive case cannot be fully excluded.
  - Low - weak match.

## 6. Severity

Start from the default severity in the catalogue, then:

- A single-signal finding is capped at **Medium**. COU-03 reaches High only when two or more heuristics agree.
- Low-confidence findings are capped at **Low**.
- Hotspot adjustment per `conventions.md` (top hotspot: up one level; unchanged in the window: down one level). A raise to High requires confidence of at least Medium.

## 7. Output

Markdown in the conversation. Offer to also write it to `design-review/coupling.md`; write it only if the user agrees.

Header (four lines):

1. **Scope and strategy** - hotspot file / git recency / capped sample, plus the languages detected.
2. **Files read vs. in scope.**
3. **Per-issue status** - for each of COU-01 to COU-04: found (n), clean, or not assessable (with reason, e.g. needs runtime information). State "no findings" explicitly for an issue checked and found clean.
4. **Assumptions** - everything inferred rather than declared, so the team can correct it in `design-context.md`.

Then findings grouped by severity (High, Medium, Low). Each finding:

- **ID and name**, severity (default -> adjusted, with the reason), confidence
- **Evidence:** `file:line` citations and symbols
- **Why it hurts here:** one line, specific to this code
- **Ruled out:** false-positive cases considered and rejected

Precision over coverage: when unsure, lower the confidence or leave the finding out. Do not suggest refactorings.
