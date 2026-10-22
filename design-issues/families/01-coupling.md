# Family 01 — Coupling

How much a piece of code knows about, and depends on, the internals of other code.

Read `../conventions.md` first: severity rubric, hotspot adjustment, ownership rule.

---

### COU-01 — Law of Demeter violation

- **Aliases:** message chains, train wreck, "don't talk to strangers"
- **Definition:** A method reaches through one object to call methods on objects it obtained from it (`order.getCustomer().getAddress().getCity()`), instead of talking only to its own fields, parameters, objects it creates, and itself.
- **Why it hurts:** The caller is coupled to the whole navigation path. Restructuring any intermediate object (moving `address`, splitting `customer`) breaks every caller that walks the chain.
- **Detection heuristics:**
  - Chains of two or more calls where each call returns a *different domain object* whose methods are then used.
  - The same chain (or the same prefix) repeated in several places — a sign the target object is missing a method.
  - Temporary variables that only exist to step through a chain (`c = o.getCustomer(); a = c.getAddress(); …`).
- **False positives:**
  - Fluent interfaces and builders where each call returns the same object or a builder (`query.where(…).orderBy(…)`).
  - Stream/collection pipelines (`list.stream().filter(…).map(…)`), LINQ, promise chains.
  - Navigating plain data structures (DTOs, JSON, config trees) at a system boundary — Demeter targets objects with behaviour.
  - Framework idioms the team can't change (e.g. `request.getSession().getAttribute(…)`).
- **Default severity:** Medium
- **Source:** Lieberherr & Holland, *Assuring Good Style for Object-Oriented Programs* (1989); Fowler, *Refactoring* — Message Chains.

---

### COU-02 — Feature envy

- **Aliases:** tell-don't-ask violation
- **Definition:** A method uses data from another object more than from its own, pulling values out with getters to make a decision or computation that belongs with that data.
- **Why it hurts:** The behaviour and the data it depends on change for the same reasons but live in different places, so every change to the data's shape ripples into the envious method. Rules about one concept end up scattered.
- **Detection heuristics:**
  - A method calls several getters on one other object and few or no members of its own class.
  - `if (x.getStatus() == … && x.getAmount() > …)` style decisions taken *about* an object by an outsider.
  - Calculations combining multiple fields of another object (`a.getPrice() * a.getQuantity() - a.getDiscount()`).
- **False positives:**
  - Strategy, visitor and similar patterns deliberately separate behaviour from data.
  - Mappers, serialisers, presenters and view models whose job is to read another object.
  - Functional-style code where data and functions are separated by design (check `design-context.md`).
- **Default severity:** Medium
- **Source:** Fowler, *Refactoring* — Feature Envy; Hunt & Thomas / Fowler bliki — TellDontAsk.

---

### COU-03 — Inappropriate intimacy

- **Aliases:** insider trading
- **Definition:** Two classes or modules know too much about each other's internals: accessing non-public members, relying on implementation details, or calling back and forth bidirectionally.
- **Why it hurts:** Neither side can change its internals independently; the pair behaves as one unit split across two files, with none of the benefits of either merging or separating them.
- **Detection heuristics:**
  - Use of package-private / internal / `friend` access, reflection, or "test-only" accessors from production code to reach another class's state.
  - Bidirectional references where each class calls into the other's methods.
  - A class that manipulates another's collections or fields directly rather than through operations.
  - Subclasses depending on protected fields of their parent in ways that would break if the parent changed its implementation.
- **False positives:**
  - Inner/nested classes and closely designed pairs inside one module (an aggregate and its entities, an iterator and its collection).
  - Parent/child structures (tree nodes) where bidirectional links are inherent.
- **Default severity:** High
- **Source:** Fowler, *Refactoring* — Inappropriate Intimacy (Insider Trading in 2nd ed.).

---

### COU-04 — Middle man

- **Aliases:** excessive delegation, pass-through class
- **Definition:** A class whose methods mostly just forward calls to another object, adding no behaviour, policy or abstraction of its own.
- **Why it hurts:** Every change to the delegate's interface has to be mirrored in the middle man; readers have an extra hop to understand the flow for no gain.
- **Detection heuristics:**
  - More than half of a class's public methods are single-line delegations to the same field.
  - Service classes that wrap a repository one-to-one (`findById` → `repo.findById`).
  - Layers of wrappers with identical method signatures.
- **False positives:**
  - Deliberate facades, adapters, proxies, decorators — check whether the wrapper hides a third-party API, adds a seam, or is the documented entry point of a module.
  - Anti-corruption layers at a context boundary (they translate, even if thinly today).
  - Application-service layers mandated by the architecture (see `design-context.md`).
- **Default severity:** Low
- **Source:** Fowler, *Refactoring* — Middle Man.

---

## Parked

| Name | Definition | Reason parked |
|---|---|---|
| Global data | Mutable state reachable from anywhere | Covered by another entry — DEP-04 Hidden dependencies |

## See also (owned elsewhere — do not report here)

- Data class (an object with no behaviour that others manipulate) → ENC-01
- Hidden dependencies, singletons → DEP-04
- Change coupling visible in history → CHG-02
