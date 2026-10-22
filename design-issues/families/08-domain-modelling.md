# Family 08 — Domain modelling

Whether business concepts are represented explicitly and kept in the right place.

Read `../conventions.md` first: severity rubric, hotspot adjustment, ownership rule.

**This family is opinionated.** Before reporting, establish intent:
1. Read `design-context.md` (intended style) and the README.
2. If absent, look for signals of an intended rich model: aggregates/entities/value-object packages, domain events, DDD vocabulary, behaviour on at least some domain types.
3. If the codebase is clearly CRUD or transaction-script by design, skip DOM-01 and lower other severities by one level; say so at the top of the findings.

---

### DOM-01 — Anaemic domain model

- **Aliases:** anemic domain model, transaction script dressed as a domain model
- **Definition:** The codebase has domain classes named after business concepts, but they hold only data; all behaviour lives in service classes that manipulate them.
- **Why it hurts:** You pay the cost of a domain model (mapping, layering) without its benefit (rules living with the data, protected invariants); rules scatter across services.
- **Detection heuristics:**
  - Domain/entity packages whose classes are data holders (see ENC-01), alongside `*Service` classes containing the business decisions about them.
  - Service methods following a get-compute-set pattern on entities.
  - Invariants checked in services rather than in entities (cross-check ENC-05).
  - Report at the *model* level (one finding with examples), not one finding per class — per-class cases are ENC-01.
- **False positives:**
  - Intended CRUD / transaction-script designs (see intent check above).
  - Functional-core designs where data and pure functions are separated by choice.
- **Default severity:** Medium
- **Source:** Fowler, bliki *AnemicDomainModel* (2003); Evans, *Domain-Driven Design*.

---

### DOM-02 — Primitive obsession

- **Aliases:** missing value objects, stringly typed code
- **Definition:** Domain concepts with their own rules (money, email, identifiers, quantities, date ranges, codes) are represented as raw primitives — strings, ints, decimals — instead of dedicated types.
- **Why it hurts:** Validation and formatting are repeated wherever the value is used; values of different concepts can be mixed up (a `customerId` passed as an `orderId`); units and currencies get lost.
- **Detection heuristics:**
  - Monetary amounts as `double`/`float`/bare `decimal` without currency.
  - Identifiers of different entities all typed as `string`/`long` and passed side by side.
  - The same validation or formatting function called on a primitive in several places (`isValidEmail(str)`).
  - Strings used as enums (`status == "active"`).
- **False positives:**
  - Boundary code (DTOs, serialisation, persistence) where primitives are converted to/from domain types.
  - Small projects where the cost of a type outweighs the risk (judgement: lower severity rather than skip).
- **Default severity:** Medium
- **Source:** Fowler, *Refactoring* — Primitive Obsession; Evans, *Domain-Driven Design* — Value Objects.

---

### DOM-03 — Business logic in the wrong place

- **Aliases:** fat controllers, smart repositories, logic in handlers
- **Definition:** Business decisions are made in controllers, request handlers, message consumers, repositories, SQL/stored procedures or UI code instead of in the domain/application layer.
- **Why it hurts:** Rules are tied to a delivery mechanism or storage technology, can't be reused from another entry point, and are hard to test without that infrastructure.
- **Detection heuristics:**
  - Controllers/handlers containing conditionals about business state, calculations, or orchestration beyond parsing input and calling a use case.
  - Repositories/queries embedding business rules (filters encoding eligibility, pricing in SQL).
  - The same rule reachable only via one entry point (HTTP) when others (batch, queue) need it too.
- **False positives:**
  - Input validation and mapping in controllers (format checks are not business rules).
  - Query filters that are pure data selection.
  - Deliberate use of the database for rules where documented (constraints, performance).
- **Default severity:** High
- **Source:** Evans, *Domain-Driven Design* — layered architecture, isolating the domain; Fowler, *Patterns of Enterprise Application Architecture* (2002) — Transaction Script vs Domain Model.

---

### DOM-04 — Bounded context leak

- **Aliases:** shared model across contexts, one model to rule them all
- **Definition:** A single model of a concept is shared across parts of the system where the concept means different things (e.g. one `Customer` class used by billing, support and marketing, accumulating fields and rules for each).
- **Why it hurts:** Each context's changes risk breaking the others; the shared model grows into a god object; teams become coupled through it.
- **Detection heuristics:**
  - Domain classes imported by many feature modules, each using a different subset of fields/methods.
  - Classes with fields that only make sense in one context (`marketingOptIn` next to `creditLimit`).
  - Conditional logic on "who is calling" inside a shared model.
  - Shared database tables/entities used directly by several modules or services.
- **False positives:**
  - Genuine shared kernel types (IDs, money, common value objects).
  - Small systems with a single context.
- **Default severity:** High
- **Source:** Evans, *Domain-Driven Design* — Bounded Context, Context Map; Vernon, *Implementing Domain-Driven Design* (2013).

---

### DOM-05 — Vocabulary mismatch

- **Aliases:** ubiquitous language drift, misleading names
- **Definition:** Code names don't match the business terms (developers say `Record` where the business says `Policy`), the same term means different things in different modules, or different terms are used for the same thing.
- **Why it hurts:** Translation between business and code is a constant source of misunderstanding; misleading names cause wrong changes.
- **Detection heuristics:**
  - Compare names in code against terms in the README, docs, API, UI strings and `design-context.md`.
  - Synonyms for the same concept across modules (`Client` / `Customer` / `Account` for one thing).
  - Names that contradict behaviour (a `validate` method that also saves, a `get` that mutates).
  - Generic names (`Data`, `Info`, `Item`, `Entity`) used for important domain concepts.
- **False positives:**
  - Different names that are correct because they belong to different bounded contexts.
  - Technical classes where no business term applies.
- **Default severity:** Low
- **Source:** Evans, *Domain-Driven Design* — Ubiquitous Language.

---

## Parked

None.

## See also (owned elsewhere — do not report here)

- Individual data classes → ENC-01
- Invalid states constructible → ENC-05
- Dumping-ground classes (`Helper`, `Manager`) → COH-04
- Infrastructure concerns in domain code → DEP-06
