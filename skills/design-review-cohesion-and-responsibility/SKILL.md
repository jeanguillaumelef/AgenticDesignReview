---
name: design-review-cohesion-and-responsibility
description: Audit a repository for cohesion and responsibility design issues - multiple responsibilities (COH-01), mixed levels of abstraction in a method (COH-02), flag arguments (COH-03) and dumping-ground classes (COH-04). Use when the user asks for a cohesion or responsibility review or design audit of a codebase. Runs standalone or can be called by an aggregator skill. Read-only and advisory.
---

# Design review: cohesion and responsibility family

Audit the repository in the working directory for the cohesion and responsibility family. You never modify code. Findings are advisory.

## 1. Load the rules (do not restate them from memory)

Read both, in this order. They live in the plugin, not in the repo under review, so use the plugin root path:

1. `${CLAUDE_PLUGIN_ROOT}/design-issues/conventions.md` - severity rubric, hotspot adjustment, ownership rule, context order.
2. `${CLAUDE_PLUGIN_ROOT}/design-issues/families/02-cohesion-and-responsibility.md` - the issues you own: COH-01 to COH-04, with heuristics and false positives.

If `${CLAUDE_PLUGIN_ROOT}` is not substituted in this text, locate the plugin directory (the one containing `.claude-plugin/plugin.json` and `design-issues/`) and read the files from there.

Report only COH-01 to COH-04. Never report, mention or hint at issues owned by other families, even the ones listed in "See also".

## 2. Build context

Read only what matters for cohesion:

- `design-context.md` at the repo root, if present: **Architecture and layers**, **Intended design style**, **Deliberate exceptions**, **Out of scope**. Ignore the other sections.
- Then the README and any architecture docs it links to.
- Then infer from code (folder structure, naming, frameworks).

Rules:

- Treat the declared design style as a false-positive filter.
- Classes declared as composition roots or facades are not reported under COH-01. If none are declared, infer them (`main`, DI configuration, wiring modules, classes that only delegate) and list the inference under Assumptions.
- A deliberate-exception row matching a COH ID and a path is **suppressed silently**. Do not report it or mention it.
- Skip paths listed under Out of scope.
- Anything inferred rather than declared goes in the header as an assumption.
- Detect the primary language(s) and state them in the header. Apply the catalogue's false-positive list using your own knowledge of the language's idioms (extension holders, package-level functions, options objects, etc.). There are no per-language reference files.

## 3. Choose what to read

Check for `design-review/hotspots.json` (contract in `conventions.md`).

1. **Exists:** read the top-ranked hotspot files first, then widen as budget allows.
2. **Missing, git repo:** take the top 50 most recently changed source files, e.g. `git log --name-only --pretty=format: -n 500`, de-duplicated, excluding generated code, lockfiles, vendored code and migrations. Never load the raw log into context beyond this de-duplicated list.
3. **Missing, no git:** take a capped sample ordered by file size and spread across directories.

State the strategy used. Without a hotspot file, use default severities and say so in the header.

## 4. Find candidates, then judge

Detection is light. Use Grep/Glob/Read to find candidates, then judge each one yourself against the issue's heuristics and false positives. No scripts.

- **COH-01:** open the largest and most-changed classes in the read set. Group methods by the fields they use and see whether the groups barely overlap. Check for dependencies on several unrelated infrastructure concerns plus business rules, imports spanning unrelated domain areas, and vague or "and"-names (`OrderManager`). Discard composition roots, facades that delegate, and large but cohesive classes (parsers, state machines).
- **COH-02:** read the top-ranked files. Look for `// step N` style comments and inline regexes, format strings, date or index arithmetic sitting next to well-named domain calls. Discard small methods where the detail is the whole job, deliberately inlined performance code, and test files.
- **COH-03:** grep for boolean or enum parameters that are branched on at the top of a function, and for call sites passing literal `true`/`false`/`null`. Discard parameters that carry data, options objects for public APIs, and framework-mandated signatures.
- **COH-04:** grep file and class names for `Utils`, `Helper(s)`, `Manager`, `Common`, `Misc`, `Tools`, `Shared`, and generic verbs without a domain noun. Check fan-in from unrelated packages and whether static methods always take the same domain type. Discard small cohesive domain-free helpers and language-imposed patterns.

## 5. Evidence and confidence

- Every finding cites at least one concrete `file:line` and the symbols involved. No evidence, no finding.
- **COH-01:** size is never a signal on its own. Report only when at least one cohesion heuristic (disjoint field groups, unrelated infrastructure dependencies, name needing "and") clearly matches.
- **COH-02:** report only methods with at least two distinct level shifts (domain call, then low-level mechanics, then domain call again). Skip test files.
- **COH-03:** report only when the flag selects between large, distinct blocks of logic, and at least one call site passes a literal or two or more call sites pass the same literal. Skip framework-mandated signatures.
- **Confidence:**
  - High - heuristics clearly match and no false-positive case applies.
  - Medium - matches, but a false-positive case cannot be fully excluded.
  - Low - weak match.

## 6. Severity

Start from the default severity in the catalogue, then:

- A single-signal finding is capped at **Medium**. COH-01 reaches High only when two or more heuristics agree.
- Low-confidence findings are capped at **Low**.
- Hotspot adjustment per `conventions.md` (top hotspot: up one level; unchanged in the window: down one level). A raise to High requires confidence of at least Medium.

## 7. Output

Markdown in the conversation. Offer to also write it to `design-review/cohesion.md`; write it only if the user agrees. When called by an aggregator, do not offer to write a file.

Header (four lines):

1. **Scope and strategy** - hotspot file / git recency / capped sample, plus the languages detected.
2. **Files read vs. in scope.**
3. **Per-issue status** - for each of COH-01 to COH-04: found (n), clean, or not assessable (with reason). State "no findings" explicitly for an issue checked and found clean.
4. **Assumptions** - everything inferred rather than declared (including inferred composition roots and facades), so the team can correct it in `design-context.md`.

Then findings grouped by severity (High, Medium, Low). Each finding:

- **ID and name**, severity (default -> adjusted, with the reason), confidence
- **Evidence:** `file:line` citations and symbols
- **Why it hurts here:** one line, specific to this code
- **Ruled out:** false-positive cases considered and rejected

Precision over coverage: when unsure, lower the confidence or leave the finding out. Do not suggest refactorings.
