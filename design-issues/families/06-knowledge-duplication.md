# Family 06 — Knowledge duplication

The same piece of *knowledge* — a business rule, a mapping, a domain value — expressed in more than one place. This is DRY in its original sense, not textual copy-paste.

Read `../conventions.md` first: severity rubric, hotspot adjustment, ownership rule.

---

### DUP-01 — Same business rule in several places

- **Aliases:** duplicated policy, scattered rule
- **Definition:** One business decision (eligibility, pricing, a validation rule, a status transition) is implemented independently in two or more places, possibly with different code that must produce the same answer.
- **Why it hurts:** When the rule changes, someone has to know every copy exists; missed copies cause inconsistent behaviour that's hard to trace.
- **Detection heuristics:**
  - The same condition over the same domain fields appearing in several services, handlers or front-end and back-end code (`age >= 18 && country == ...`).
  - Similar calculations with different variable names (e.g. VAT, discount or fee computed in more than one module).
  - The same status-transition checks repeated in multiple handlers.
  - Comments like "keep in sync with …" or "same as in …".
- **False positives:**
  - Validation repeated across layers on purpose (client-side for UX, server-side for safety) — flag only if the *rule* differs between copies.
  - Coincidental similarity: same code shape, different reasons to change (merging them would couple unrelated concepts).
- **Default severity:** High
- **Source:** Hunt & Thomas, *The Pragmatic Programmer* (1999) — DRY ("every piece of knowledge must have a single, authoritative representation").

---

### DUP-02 — Parallel structures that must be kept in sync

- **Aliases:** shadow enums, mirrored mappings, shotgun-ready lists
- **Definition:** A set of values or cases is defined once but mirrored by hand elsewhere: an enum duplicated in mapping tables, switch statements, database seed values, front-end constants, translation keys or config.
- **Why it hurts:** Adding or renaming a case requires updating several places with no compiler help; omissions surface as runtime errors or silent fallbacks.
- **Detection heuristics:**
  - Several `Map`s / dictionaries / switch statements keyed by the same enum or string set.
  - String literals matching enum names (`"PENDING"`, `"SHIPPED"`) in other modules, SQL or config.
  - Front-end and back-end defining the same list of codes independently.
  - Hotspot data: files that always change together when a new case is added (cross-check CHG-02).
- **False positives:**
  - Generated code from a single source of truth (OpenAPI, protobuf, schema).
  - Contracts across independently deployed services, where duplication is a deliberate decoupling choice (flag as context, not a defect).
- **Default severity:** Medium
- **Source:** Hunt & Thomas, *The Pragmatic Programmer* — DRY; Fowler, *Refactoring* — Shotgun Surgery (the symptom).

---

### DUP-03 — Domain constants repeated as literals

- **Aliases:** magic values with domain meaning
- **Definition:** Values that carry business meaning (thresholds, rates, limits, durations, codes) appear as raw literals in several places instead of being named once.
- **Why it hurts:** Changing the business value requires a search-and-hope; readers can't tell whether two `30`s are the same concept or a coincidence.
- **Detection heuristics:**
  - The same non-trivial literal (`0.2`, `30`, `5000`, `"EUR"`) used in several files in a business context.
  - Literals in conditions that read like policy (`if (days > 30)`, `amount * 0.2`).
  - Only flag when the *meaning* is shared; generic magic-number detection belongs to linters.
- **False positives:**
  - Universal constants (0, 1, 100 for percentages, 60 for seconds).
  - Literals in tests (they should often be explicit).
- **Default severity:** Low
- **Source:** Hunt & Thomas, *The Pragmatic Programmer* — DRY; Fowler, *Refactoring* — Replace Magic Literal.

---

## Parked

| Name | Definition | Reason parked |
|---|---|---|
| Copy-paste code | Textually duplicated blocks | Better caught by tooling (PMD CPD, jscpd, SonarQube duplication) |
| Validation duplicated across layers | Same check client- and server-side | Covered by another entry — DUP-01, which flags it only when the copies of the rule diverge |

## See also (owned elsewhere — do not report here)

- Type switching repeated across the codebase → ABS-04
- Change coupling seen in history → CHG-02
- Shotgun surgery seen in history → CHG-03
