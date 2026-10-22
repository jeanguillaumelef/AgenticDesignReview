# Family 03 — Abstraction and inheritance

Whether type hierarchies and abstractions are sound, honest and earning their keep.

Read `../conventions.md` first: severity rubric, hotspot adjustment, ownership rule.

---

### ABS-01 — Liskov substitution violation

- **Aliases:** LSP violation, broken subtype contract
- **Definition:** A subtype cannot be used wherever its supertype is expected without surprising the caller: it strengthens preconditions, weakens postconditions, throws where the parent doesn't, or ignores inherited behaviour.
- **Why it hurts:** Polymorphism stops being safe; callers start checking concrete types, and every new subtype risks breaking existing code paths.
- **Detection heuristics:**
  - Overrides that throw `UnsupportedOperationException` / `NotImplementedError` or silently do nothing.
  - Callers using `instanceof` / type checks / casts on an abstraction to special-case one subtype.
  - Overrides that add validation the parent doesn't have, or return values outside the parent's documented range.
  - Classic shape problems (`Square extends Rectangle` with independent setters).
- **False positives:**
  - Abstract template methods intentionally left as no-ops for optional hooks.
  - Read-only views of mutable collections when the API documents optional operations (e.g. Java collections) — note but lower severity.
  - Test doubles.
- **Default severity:** High
- **Source:** Liskov, *Data Abstraction and Hierarchy* (1987); Liskov & Wing (1994); Martin, *Agile Software Development* — LSP.

---

### ABS-02 — Refused bequest

- **Aliases:** unwanted inheritance
- **Definition:** A subclass inherits methods or data it doesn't use or want, overriding them to do nothing or ignoring most of the parent's interface.
- **Why it hurts:** The hierarchy misrepresents the relationship; changes to the parent force reasoning about subclasses that don't share its concept.
- **Detection heuristics:**
  - Subclass uses or overrides only a small fraction of the parent's public interface.
  - Many inherited methods are irrelevant to the subclass's purpose (judged by name and domain).
  - Overrides that exist only to neutralise parent behaviour.
- **False positives:**
  - Framework base classes (`Activity`, `Controller`, `TestCase`) that are large by design.
  - Abstract base classes offering optional hooks.
- **Default severity:** Medium
- **Source:** Fowler, *Refactoring* — Refused Bequest.

---

### ABS-03 — Inheritance used for code reuse

- **Aliases:** implementation inheritance, prefer composition over inheritance
- **Definition:** A class extends another to reuse its code rather than because it *is a* specialisation of it (`OrderService extends BaseRepository`, `ReportGenerator extends HttpClient`).
- **Why it hurts:** The subclass is tied to the parent's implementation details (fragile base class); it exposes an interface it shouldn't; reuse that could be swapped or tested in isolation is frozen into the type hierarchy.
- **Detection heuristics:**
  - Parent and child belong to different concepts or layers (a domain class extending an infrastructure class).
  - Base classes named `Base*`, `Abstract*`, `Common*` that hold grab-bags of helper methods.
  - Deep hierarchies (3+ levels) where intermediate levels add utilities rather than meaning.
  - Subclass uses parent methods but never participates in polymorphism (never passed as the parent type).
- **False positives:**
  - Framework-mandated inheritance.
  - Genuine specialisation with shared behaviour (template method pattern used intentionally).
- **Default severity:** Medium
- **Source:** Gamma et al., *Design Patterns* (1994) — "favor object composition over class inheritance"; Bloch, *Effective Java* — favour composition over inheritance.

---

### ABS-04 — Type switching where polymorphism belongs

- **Aliases:** switch statements, repeated conditionals on type, missing polymorphism
- **Definition:** The same `switch`/`if` chain on a type code, enum or class is repeated across the codebase to pick behaviour, instead of letting each type carry its own behaviour.
- **Why it hurts:** Adding a new case means finding and updating every switch; missing one is a silent bug.
- **Detection heuristics:**
  - The same discriminator (`type`, `kind`, `status`, enum) switched on in two or more places with different behaviour per branch.
  - `instanceof` chains choosing behaviour.
  - Default branches that throw "unknown type".
- **False positives:**
  - A single switch at a boundary (factory, parser, deserialiser) that *creates* polymorphic objects.
  - Exhaustive pattern matching over closed sum types in functional or modern code (sealed types, ADTs) where the compiler enforces completeness — this is a valid design choice.
  - Switches over values that are not behavioural types (HTTP status → message).
- **Default severity:** Medium
- **Source:** Fowler, *Refactoring* — Switch Statements / Repeated Switches, Replace Conditional with Polymorphism.

---

### ABS-05 — Speculative generality

- **Aliases:** YAGNI violation, premature abstraction
- **Definition:** Abstractions, hooks, parameters or extension points built for anticipated needs that never arrived: interfaces with one implementation and no seam purpose, unused parameters, abstract classes with one subclass.
- **Why it hurts:** Extra indirection costs every reader; the wrong abstraction is harder to undo than duplication; it suggests flexibility that isn't actually exercised.
- **Detection heuristics:**
  - Interfaces or abstract classes with exactly one implementation *and* no use as a test seam or module boundary.
  - Parameters always passed the same value; unused generic type parameters.
  - Plugin/strategy machinery with a single registered strategy.
  - Hotspot data showing the abstraction never changes while its single implementation does.
- **False positives:**
  - Interfaces at a module or layer boundary (ports in hexagonal architecture) — one implementation is normal.
  - Interfaces introduced as test seams for external dependencies.
  - Public library APIs designed for third-party extension.
- **Default severity:** Low
- **Source:** Fowler, *Refactoring* — Speculative Generality; Metz, *The Wrong Abstraction* (2016).

---

### ABS-06 — Leaky abstraction

- **Aliases:** abstraction exposing implementation details
- **Definition:** An interface or abstraction reveals details of its implementation in its signatures, types, errors or required usage (`UserRepository.find(...)` returning a `ResultSet`, a `Storage` interface throwing `S3Exception`, callers having to know about pagination tokens of a specific provider).
- **Why it hurts:** Callers become coupled to the implementation the abstraction was meant to hide; swapping or changing it requires touching every caller.
- **Detection heuristics:**
  - Interface signatures using vendor/framework/ORM types (entity managers, SDK types, HTTP types) in a supposedly neutral abstraction.
  - Callers that must call methods in a vendor-specific order or handle vendor-specific errors.
  - Abstractions whose method names describe *how* (`executeSqlForUser`) rather than *what*.
- **False positives:**
  - Thin adapters explicitly meant to be vendor-specific.
  - Performance hints deliberately surfaced (batch sizes) where documented.
- **Default severity:** High
- **Source:** Spolsky, *The Law of Leaky Abstractions* (2002); Parnas, *On the Criteria To Be Used in Decomposing Systems into Modules* (1972) — information hiding.

---

## Parked

| Name | Definition | Reason parked |
|---|---|---|
| Parallel inheritance hierarchies | Adding a subclass in one hierarchy forces one in another | Low value for effort |
| Lazy class | A class that does too little to justify existing | Low value for effort |

## See also (owned elsewhere — do not report here)

- Low-level errors leaking across layers → ERR-04
- Fat interfaces (ISP) → DEP-05
- Parallel structures kept in sync (enum mirrored in mappings) → DUP-02
