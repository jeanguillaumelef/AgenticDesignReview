# Family 07 — Change patterns (history-based)

Design problems that are visible in *how the code changes over time*, measured from git history. This family produces the hotspot file that other families may use for severity adjustment (see `../conventions.md` — History analysis).

Findings in this family are **not** themselves adjusted by hotspot data; they *are* the hotspot data.

---

### CHG-01 — Hotspots

- **Aliases:** churn × complexity hotspots
- **Definition:** Files (or functions) that are both complex and changed frequently within the analysis window.
- **Why it hurts:** This is where design problems actually cost money: complex code that is rarely touched is cheap; complex code changed every week taxes every change.
- **Detection heuristics:**
  - Rank files by change frequency (commits touching the file in the window) combined with a complexity proxy (lines of code or indentation-based complexity).
  - Report the top N with their change count, size and trend (growing, stable, shrinking complexity).
  - For the top few, read the file and name the likely design reason (cross-referencing issues from other families).
- **False positives:**
  - Files that change often by nature and are simple: changelogs, route tables, feature-flag lists, dependency manifests, translation files — exclude by path or flag as "expected churn".
  - Files renamed or moved in the window (follow renames).
- **Default severity:** High (for the top hotspots; others are listed as context)
- **Source:** Tornhill, *Your Code as a Crime Scene* (2015; 2nd ed. 2024); Tornhill, *Software Design X-Rays* (2018).

---

### CHG-02 — Change coupling without a code link

- **Aliases:** logical coupling, temporal coupling (history sense)
- **Definition:** Two files or modules that repeatedly change in the same commits even though there is no direct code dependency between them.
- **Why it hurts:** It reveals hidden dependencies — duplicated knowledge, implicit contracts, parallel structures — that a code reading alone won't show and that developers must remember.
- **Detection heuristics:**
  - Compute co-change pairs weighted by commit size (see conventions) and keep pairs above a minimum shared-commit count.
  - Check for a direct import/reference between the pair; report the ones *without* one as the strongest signal, those *with* one as context.
  - Pairs crossing module, service or layer boundaries rank higher than pairs inside one folder.
  - Name the likely cause where visible (shared enum mirrored, duplicated rule, schema + mapper).
- **False positives:**
  - Production file and its test file.
  - Interface and implementation, code and its migration/schema when deliberately paired.
  - Pairs only coupled through very large commits (the size weighting should already reduce them).
- **Default severity:** High
- **Source:** Gall, Hajek & Jazayeri, *Detection of Logical Coupling Based on Product Release History* (1998); Tornhill, *Software Design X-Rays*.

---

### CHG-03 — Shotgun surgery

- **Aliases:** scattered change
- **Definition:** A single kind of change (a new field, a new case, a policy tweak) routinely requires edits across many files or modules.
- **Why it hurts:** Each change is expensive and error-prone; the concept being changed has no single home.
- **Detection heuristics:**
  - Commits (excluding mechanical ones) that touch many files across several modules, recurring with a similar footprint.
  - Clusters of files that co-change as a group of three or more (extends CHG-02 to groups).
  - Commit messages describing one small concept change ("add status X", "support currency Y") with a wide footprint.
  - For each cluster, name the concept that is scattered.
- **False positives:**
  - Deliberate cross-cutting changes (dependency upgrades, renames, API version bumps) — these should be excluded as mechanical or recognised from the message.
  - Large commits caused by delivery habits rather than design: report the share of large commits as *context* in the report header, not as shotgun surgery, unless the same concept recurs.
- **Default severity:** High
- **Source:** Fowler, *Refactoring* — Shotgun Surgery.

---

### CHG-04 — Divergent change

- **Aliases:** one file, many reasons to change
- **Definition:** A single file is modified for many unrelated reasons over time.
- **Why it hurts:** It is the historical evidence of a multiple-responsibility class: unrelated work collides in the same file, causing conflicts and regressions.
- **Detection heuristics:**
  - For high-churn files, group commit messages (and co-changed files) into themes; several distinct themes = divergent change.
  - Many different authors or teams touching the file for different features.
  - Cross-reference COH-01 findings: report the history evidence here; COH-01 reports the code structure.
- **False positives:**
  - Composition roots, route registries, central config files — expected to change for many reasons.
  - Poor commit messages may hide the themes; state low confidence when messages are uninformative.
- **Default severity:** Medium
- **Source:** Fowler, *Refactoring* — Divergent Change.

---

## Parked

| Name | Definition | Reason parked |
|---|---|---|
| Knowledge concentration / bus factor | Code mostly written or changed by one person | Not a code-design issue |
| Code age / stale areas | Code untouched for a long time | Not a code-design issue (used only as a severity input) |

## See also (owned elsewhere — do not report here)

- Code-level multiple responsibilities → COH-01
- Code-level duplicated rules and parallel structures → DUP-01, DUP-02
