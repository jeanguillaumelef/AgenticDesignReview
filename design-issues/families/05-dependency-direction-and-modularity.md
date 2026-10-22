# Family 05 — Dependency direction and modularity

Which way dependencies point between modules and layers, and whether they are explicit.

Read `../conventions.md` first: severity rubric, hotspot adjustment, ownership rule. This family relies most on `design-context.md`: if layers aren't declared, infer them from package/folder structure and the README, and state the inferred layering at the top of the findings.

---

### DEP-01 — Dependency inversion violation

- **Aliases:** DIP violation, policy depends on detail
- **Definition:** High-level policy code (business rules, use cases) depends directly on concrete low-level details (a specific database client, HTTP library, file format, vendor SDK) instead of on an abstraction it owns.
- **Why it hurts:** Business rules can't be changed, reused or tested without dragging infrastructure along; replacing a detail means editing the policy.
- **Detection heuristics:**
  - Domain or use-case code importing database drivers, ORM sessions, HTTP clients, queue clients, cloud SDKs.
  - `new ConcreteInfrastructureThing(...)` inside business logic.
  - Interfaces for infrastructure defined *in the infrastructure package* and imported by the domain (the abstraction is owned by the wrong side).
- **False positives:**
  - Small scripts, CLIs or CRUD apps where layering is not intended (check `design-context.md` / README).
  - Dependencies on stable standard-library abstractions (collections, time types).
- **Default severity:** High
- **Source:** Martin, *Agile Software Development* — DIP; Cockburn, *Hexagonal Architecture* (2005).

---

### DEP-02 — Layer violations

- **Aliases:** wrong-direction dependency, layer skipping, architecture erosion
- **Definition:** A module depends on a layer it shouldn't: lower layers importing higher ones (domain → web), or layers being bypassed (controller → repository, skipping the application layer).
- **Why it hurts:** The intended structure stops protecting anything; changes in one layer ripple in directions the architecture was designed to prevent.
- **Detection heuristics:**
  - Imports from a lower layer to a higher layer per declared or inferred layering.
  - Presentation code calling persistence directly where an application/service layer exists elsewhere.
  - Domain types annotated with or importing web/serialisation framework types.
  - Inconsistency: most features follow the layering, a few don't (the few are the finding).
- **False positives:**
  - Projects without intended layering.
  - Shared kernel / common types modules explicitly allowed everywhere.
  - Read-side shortcuts in CQRS designs where queries legitimately bypass the domain (check `design-context.md`).
- **Default severity:** High
- **Source:** Buschmann et al., *Pattern-Oriented Software Architecture Vol. 1* (1996) — Layers; Martin, *Clean Architecture* (2017) — the Dependency Rule.

---

### DEP-03 — Unstable dependencies

- **Aliases:** stable dependencies principle violation
- **Definition:** A module that many others depend on (and is therefore hard to change) depends on a module that changes frequently or is likely to change.
- **Why it hurts:** Volatility propagates into the most widely used code; changes to the volatile module force changes or re-testing across the system.
- **Detection heuristics:**
  - Core/shared modules importing feature-specific or experimental modules.
  - With hotspot data: a high-fan-in module depending on a module with high churn.
  - Shared libraries depending on application code.
- **False positives:**
  - Dependencies on stable third-party libraries.
  - Early-stage projects where everything is volatile.
- **Default severity:** Medium
- **Source:** Martin, *Agile Software Development* — Stable Dependencies Principle.

---

### DEP-04 — Hidden dependencies

- **Aliases:** singletons, service locator, static coupling, ambient context, global state
- **Definition:** A class obtains collaborators it needs without declaring them: static calls to singletons, service locators, global registries, or direct use of the system clock, randomness, environment variables or I/O.
- **Why it hurts:** The class's real dependencies are invisible from its constructor/signature; tests need global setup; behaviour depends on hidden shared state.
- **Detection heuristics:**
  - `Something.getInstance()`, `ServiceLocator.get(...)`, `Container.resolve(...)` inside business code.
  - Direct calls to `now()`, `Date()`, `random()`, `System.getenv`, file/network APIs in logic that should be deterministic.
  - Mutable static fields / module-level mutable variables.
  - Tests that reset globals or patch modules to run.
- **False positives:**
  - Composition root / bootstrap code.
  - Immutable constants and pure static functions.
  - Loggers obtained statically (widely accepted idiom; covered by DEP-06 only if logging is entangled with logic).
- **Default severity:** Medium
- **Source:** Seemann & van Deursen, *Dependency Injection Principles, Practices, and Patterns* (2019) — Service Locator and Ambient Context anti-patterns; Feathers, *Working Effectively with Legacy Code* (2004).

---

### DEP-05 — Fat interfaces

- **Aliases:** ISP violation, interface pollution
- **Definition:** An interface bundles many operations that different clients use in disjoint subsets, so each client depends on methods it never calls.
- **Why it hurts:** Clients are recompiled/retested for changes they don't care about; implementers must provide (or stub) methods irrelevant to them; test doubles become large.
- **Detection heuristics:**
  - Interfaces with many methods where each implementer or each client uses a clearly separate subset.
  - Implementations throwing "not supported" for some interface methods (cross-check ABS-01).
  - Test doubles stubbing many methods with dummy values.
- **False positives:**
  - Cohesive interfaces that are large because the concept is large (a collection API).
  - Generated interfaces (API clients).
- **Default severity:** Medium
- **Source:** Martin, *Agile Software Development* — Interface Segregation Principle.

---

### DEP-06 — Infrastructure concerns in core logic

- **Aliases:** cross-cutting tangling, framework coupling in the domain
- **Definition:** Business logic is interleaved with logging, configuration lookups, metrics, transaction management, retries or framework annotations, so the rule and the plumbing can't be read or changed separately.
- **Why it hurts:** Business rules become hard to see and test; changing the plumbing (logging library, config source, framework) means editing domain code.
- **Detection heuristics:**
  - Domain methods reading configuration directly (env vars, config singletons) instead of receiving values.
  - Heavy framework annotations (persistence, serialisation, DI, validation) on core domain types.
  - Logging/metrics statements making up a large share of business methods, or log content driving control flow.
  - Manual transaction/retry handling inside business rules.
- **False positives:**
  - Frameworks where annotated domain types are the documented idiom and the team has chosen it (check `design-context.md`).
  - Occasional diagnostic logging at significant domain events.
- **Default severity:** Medium
- **Source:** Kiczales et al., *Aspect-Oriented Programming* (1997) — tangling of cross-cutting concerns; Evans, *Domain-Driven Design* — isolating the domain.

---

## Parked

| Name | Definition | Reason parked |
|---|---|---|
| Cyclic dependencies | Packages/modules depending on each other in a cycle | Better caught by tooling (dependency-cruiser, ArchUnit, jdepend, madge…) |

## See also (owned elsewhere — do not report here)

- Leaky abstractions (vendor types in interfaces) → ABS-06
- Business logic in controllers / repositories → DOM-03
- Low-level errors crossing layers → ERR-04
