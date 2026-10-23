---
name: design-review
description: Run a software design audit of the current repository by running every available design-review family skill and aggregating the findings into one prioritized report. Use when the user asks for a design review, design audit or design health check of a codebase. Read-only and advisory.
---

# Design review: aggregator

Audit the repository in the working directory for software design issues. You never modify code. Findings are advisory.

This skill does no detection itself. It runs the family skills and combines their output.

## 1. Load the rules

Read `${CLAUDE_PLUGIN_ROOT}/design-issues/conventions.md` (severity rubric, ownership rule, context order). If `${CLAUDE_PLUGIN_ROOT}` is not substituted, locate the plugin directory (the one containing `.claude-plugin/plugin.json` and `design-issues/`).

## 2. Find the families

The catalogue has one file per family in `${CLAUDE_PLUGIN_ROOT}/design-issues/families/`. A family is **built** when a skill named `design-review-<family>` exists in `${CLAUDE_PLUGIN_ROOT}/skills/`. Otherwise it is **not built**.

Currently built: `design-review-coupling` (family 01). Detect the rest from the folder, do not rely on this list.

## 3. Run each built family

Invoke each built family skill with the Skill tool, one after another. Each one reads its own rules and builds its own context, so do not restate or override them. Tell each one it is being called by an aggregator, so it must not offer to write its own report file.

Keep each family's findings exactly as reported. Do not re-judge, merge or drop them. The ownership rule still applies: one issue, one owning family.

## 4. Output

Markdown in the conversation. Offer to also write it to `design-review/report.md`; write it only if the user agrees.

**Header:**

1. **Scope and strategy** - how files were chosen and the languages detected, as reported by the families.
2. **Family coverage** - for every family in the catalogue: assessed, or not built yet. Name the not-built families explicitly, so the reader knows what this report does not cover.
3. **Per-issue status** - merged from the families' headers (found, clean, not assessable).
4. **Assumptions** - the union of the families' assumptions, de-duplicated, so the team can correct them in `design-context.md`.

**Findings:** all families' findings grouped by severity (High, Medium, Low), most severe first. Keep each finding's ID, evidence, confidence and ruled-out cases as the family reported them.

Do not suggest refactorings. Precision over coverage: never add a finding no family reported.
