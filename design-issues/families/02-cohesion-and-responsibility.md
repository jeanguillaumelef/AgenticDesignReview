# Family 02 — Cohesion and responsibility

Whether each unit of code does one coherent job, at one level of abstraction.

Read `../conventions.md` first: severity rubric, hotspot adjustment, ownership rule.

---

### COH-01 — Multiple responsibilities

- **Aliases:** god class, blob, SRP violation, large class (by responsibility)
- **Definition:** A class or module serves several unrelated purposes or several actors (e.g. pricing rules, persistence and email formatting in one place), so it has more than one reason to change.
- **Why it hurts:** Unrelated changes collide in the same file: more merge conflicts, wider regression risk, and every change requires understanding everything the class does.
- **Detection heuristics:**
  - Groups of methods that use disjoint subsets of the fields (low cohesion) — the class could be split along those groups with few cross-references.
  - Dependencies on several unrelated infrastructure concerns at once (database, HTTP, file system, messaging) plus business rules.
  - A class name that needs "and" to describe it honestly, or a vague name (`OrderManager`) covering many verbs.
  - Imports spanning several unrelated domain areas.
  - Size is a *supporting* signal only; judge by the number of distinct reasons to change.
- **False positives:**
  - Composition roots / wiring modules (DI configuration, `main`) whose job is to know about everything.
  - Facades that deliberately expose many operations but delegate each one.
  - Large but cohesive classes (a parser, a state machine, a well-structured algorithm).
- **Default severity:** High
- **Source:** Martin, *Agile Software Development: Principles, Patterns, and Practices* (2002) — SRP; Fowler, *Refactoring* — Large Class; Riel, *Object-Oriented Design Heuristics* (1996) — god class.

---

### COH-02 — Mixed levels of abstraction in a method

- **Aliases:** violation of "one level of abstraction per function", missing composed method
- **Definition:** A single method interleaves high-level intent (`applyDiscounts`, `notifyCustomer`) with low-level mechanics (string parsing, index arithmetic, byte buffers, SQL fragments).
- **Why it hurts:** Readers must mentally switch levels to follow the logic; the high-level policy is buried, so it's hard to see what changes when the policy changes.
- **Detection heuristics:**
  - A method that calls well-named domain operations and also contains raw loops, regexes, date arithmetic or format strings inline.
  - Comment headers inside a method marking "steps" (`// step 2: validate`) — each step is a candidate method.
  - Nesting depth that jumps between sections of the same method.
- **False positives:**
  - Small methods where the low-level detail *is* the whole job.
  - Performance-critical code deliberately inlined (should be stated in a comment or in `design-context.md`).
  - Test methods (arrange/act/assert naturally mix levels).
- **Default severity:** Low
- **Source:** Beck, *Smalltalk Best Practice Patterns* (1996) — Composed Method; Martin, *Clean Code* (2008).

---

### COH-03 — Flag arguments

- **Aliases:** boolean parameter, control coupling, selector argument
- **Definition:** A function takes a boolean or enum parameter whose purpose is to choose between different behaviours inside it (`render(order, true)`, `save(user, Mode.DRAFT)`).
- **Why it hurts:** The function does two (or more) jobs; call sites are unreadable without opening the function; every new mode adds another branch to the same body.
- **Detection heuristics:**
  - A boolean/enum parameter used mainly in an `if`/`switch` that selects large, distinct blocks of logic.
  - Call sites passing literal `true`/`false`/`null`.
  - Pairs of call sites that always pass the same literal — a hint that two separate functions are wanted.
- **False positives:**
  - Parameters that carry *data* rather than select behaviour (`setEnabled(bool)`, `withTimeout(ms)`).
  - Options objects for public APIs with many genuinely independent settings.
  - Framework callbacks with mandated signatures.
- **Default severity:** Low
- **Source:** Fowler, bliki *FlagArgument* (2011); Fowler, *Refactoring* 2nd ed. — Remove Flag Argument; Martin, *Clean Code*.

---

### COH-04 — Dumping-ground classes

- **Aliases:** utility class sprawl, `Helper` / `Utils` / `Manager` / `Common` / `Misc`
- **Definition:** Classes or modules named for *being a place to put things* rather than for a concept, accumulating unrelated static functions or operations over time.
- **Why it hurts:** Behaviour that belongs to a concept (a `Money`, an `Order`) lives far from it; the dumping ground becomes a widely imported hub that couples unrelated parts of the system.
- **Detection heuristics:**
  - Names ending in `Utils`, `Helper(s)`, `Manager`, `Common`, `Misc`, `Tools`, `Shared`, or generic verbs (`Processor`, `Handler`) without a domain noun.
  - Static methods whose first parameter is always the same domain type (behaviour that wants to live on that type).
  - High fan-in from many unrelated packages.
- **False positives:**
  - Genuinely generic, domain-free helpers (string/collection extensions) — if they are small and cohesive.
  - Language-imposed patterns (e.g. Kotlin/C# extension holders, Go package-level functions in a well-named package).
- **Default severity:** Medium
- **Source:** Riel, *Object-Oriented Design Heuristics*; Evans, *Domain-Driven Design* (2003) — behaviour belongs with the concept.

---

## Parked

| Name | Definition | Reason parked |
|---|---|---|
| Long method | Method exceeding a length threshold | Better caught by tooling (length / cyclomatic complexity rules) |
| Large class (by size) | Class exceeding a size threshold | Better caught by tooling; responsibility-based version is COH-01 |
| Divergent change | One file changed for unrelated reasons | Covered by another entry — CHG-04, measured from history |

## See also (owned elsewhere — do not report here)

- Infrastructure concerns (logging, config) mixed into core logic → DEP-06
- Business logic in controllers / repositories → DOM-03
- Eager tests (one test, many behaviours) → TST-03
