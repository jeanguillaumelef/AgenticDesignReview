# Family 04 — Encapsulation and data

Whether objects protect their state and invariants, or expose them for others to manage.

Read `../conventions.md` first: severity rubric, hotspot adjustment, ownership rule.

---

### ENC-01 — Data class

- **Aliases:** dumb data holder (outside of DTO use)
- **Definition:** A class with fields and accessors but no meaningful behaviour, while the logic that operates on its data lives in other classes.
- **Why it hurts:** The rules about the data are spread across its users; nothing guards its consistency; changes to its fields ripple widely.
- **Detection heuristics:**
  - Class with only fields, getters/setters, `equals`/`hashCode`/`toString`.
  - Other classes contain multiple methods whose main work is reading and writing this class's fields (cross-check with COU-02 Feature envy — report the data class here, the envious methods there).
  - Setters for every field, including ones that should only change through a business operation.
- **False positives:**
  - DTOs, API request/response models, persistence records, events and messages — data carriers at boundaries are fine.
  - Immutable value objects / records with no behaviour needed yet.
  - Functional or data-oriented designs (check `design-context.md`).
- **Default severity:** Low
- **Source:** Fowler, *Refactoring* — Data Class.

---

### ENC-02 — Exposed mutable internals

- **Aliases:** leaking internal state, representation exposure
- **Definition:** An object hands out direct references to its mutable internal state (collections, arrays, nested mutable objects) or exposes public mutable fields, so outsiders can change it without going through its methods.
- **Why it hurts:** Invariants can be broken from anywhere; bugs appear far from their cause; the object can never change its internal representation safely.
- **Detection heuristics:**
  - Getters returning internal `List`/`Map`/`Set`/arrays without copying or wrapping as read-only.
  - Public non-final / non-readonly fields on classes with behaviour.
  - Callers doing `obj.getItems().add(...)` / `.clear()` / `.remove(...)`.
  - Constructors storing caller-supplied mutable collections without copying.
- **False positives:**
  - Builders and deliberately mutable data structures.
  - Immutable collection types returned by the language/runtime.
  - Performance-sensitive code where this is documented.
- **Default severity:** High
- **Source:** Bloch, *Effective Java* — defensive copies, minimise mutability; Fowler, *Refactoring* — Encapsulate Collection.

---

### ENC-03 — Temporal coupling

- **Aliases:** hidden call-order dependency, sequential coupling
- **Definition:** An object only works if its methods are called in a particular order that the API doesn't enforce (`init()` before `process()`, `setConnection()` before `query()`, `open()`/`close()` pairs left to the caller).
- **Why it hurts:** Correct usage depends on tribal knowledge; misuse compiles and fails at runtime, often intermittently.
- **Detection heuristics:**
  - Methods named `init`, `setup`, `prepare`, `configure`, `start` that must precede others.
  - Fields that are `null` until a setter is called, with later methods dereferencing them.
  - Guards like `if (!initialized) throw ...`.
  - Resource pairs (open/close, begin/commit) managed manually by callers instead of a scoped construct.
- **False positives:**
  - Lifecycle methods called by a framework that guarantees order.
  - Builders (the order is enforced by `build()`).
  - Explicit state machines where order is the modelled behaviour and illegal transitions are rejected.
- **Default severity:** Medium
- **Source:** Seemann, *Design Smell: Temporal Coupling* (2011); Martin, *Clean Code* — hidden temporal couplings.

---

### ENC-04 — Data clumps

- **Aliases:** missing parameter object
- **Definition:** The same group of fields or parameters travels together through many signatures and classes (`street, city, postcode`; `startDate, endDate`; `amount, currency`) without being a type of its own.
- **Why it hurts:** Validation and behaviour for the group get duplicated; signatures grow; it's easy to mix up order or pass inconsistent combinations.
- **Detection heuristics:**
  - Three or more parameters appearing together, in the same order, across several method signatures.
  - The same set of fields repeated in multiple classes.
  - Repeated validation of the same combination (`start < end`).
- **False positives:**
  - Coincidental pairs with no shared meaning.
  - Framework-generated signatures.
- **Default severity:** Low
- **Source:** Fowler, *Refactoring* — Data Clumps, Introduce Parameter Object.

---

### ENC-05 — Invalid states constructible

- **Aliases:** half-built objects, unprotected invariants, "make illegal states unrepresentable"
- **Definition:** The type allows objects to exist in states the domain forbids: constructors that don't establish invariants, setters that bypass validation, or field combinations that make no sense (`status = SHIPPED` with `shippedAt = null`).
- **Why it hurts:** Every consumer must defensively re-check validity; bugs show up as corrupt data rather than as rejected operations.
- **Detection heuristics:**
  - No-arg constructors followed by setter calls on domain objects.
  - Validation done in services or controllers rather than when the object is created or changed.
  - Nullable fields whose nullness depends on another field's value (should be a separate type or state).
  - Boolean flags that are mutually exclusive (`isDraft`, `isPublished`, `isArchived`).
- **False positives:**
  - ORM or serialisation constraints requiring no-arg constructors — check whether invariants are enforced elsewhere (factory, domain methods).
  - DTOs at boundaries (validated when converted to domain objects).
- **Default severity:** High
- **Source:** Evans, *Domain-Driven Design* — invariants and aggregates; Minsky, *Effective ML* (2011) — make illegal states unrepresentable.

---

## Parked

None.

## See also (owned elsewhere — do not report here)

- Feature envy / tell-don't-ask (the methods using the data class) → COU-02
- Message chains through object graphs → COU-01
- Primitive obsession, missing value objects → DOM-02
- Global mutable state, singletons → DEP-04
