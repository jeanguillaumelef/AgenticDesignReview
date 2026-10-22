# Family 09 — Test design

Whether the tests support change or resist it. This family looks at the design of the tests themselves; the testability of production code is covered by family 05 (DEP-04 Hidden dependencies).

Read `../conventions.md` first: severity rubric, hotspot adjustment, ownership rule.

**Classify test level first.** For each test file, decide whether it is a unit, integration or end-to-end test, using: location and naming conventions (`unit/`, `it/`, `*.e2e.*`, `*IT`), the framework (Playwright, Cypress, Selenium, Testcontainers…), what it touches (real database, network, browser, multiple services) and `design-context.md`. Each entry below states which levels it applies to. State the classification rules used at the top of the findings.

---

### TST-01 — Tests coupled to implementation

- **Aliases:** overspecified tests, fragile tests, testing internals
- **Definition:** Tests assert *how* the code works (which internal methods are called, in which order, private state, exact SQL or call counts) instead of *what* it produces.
- **Why it hurts:** Refactoring — changing structure without changing behaviour — breaks tests, so the tests punish exactly the work that improves the design.
- **Applies to:** all levels (at e2e level: tests relying on DOM structure, CSS selectors or internal IDs rather than user-visible behaviour).
- **Detection heuristics:**
  - Heavy use of `verify(...)`/`toHaveBeenCalledWith` on collaborators that are not observable boundaries.
  - Tests accessing private members via reflection or test-only accessors.
  - Assertions on intermediate state, call order or call counts.
  - E2E selectors based on structure (`div > div:nth-child(3)`) rather than roles, labels or test IDs.
- **False positives:**
  - Verifying calls to genuine outbound boundaries (an email sent, a message published, a payment requested) — that *is* the behaviour.
  - Contract tests that deliberately pin an interface.
- **Default severity:** High
- **Source:** Meszaros, *xUnit Test Patterns* (2007) — Fragile Test, Overspecified Software; Khorikov, *Unit Testing Principles, Practices, and Patterns* (2020).

---

### TST-02 — Over-mocking

- **Aliases:** mocking what you don't need to, mock-heavy tests
- **Definition:** Tests replace collaborators that don't need replacing — value objects, simple domain objects, in-process pure logic, or types owned by third parties mocked directly instead of behind an adapter.
- **Why it hurts:** Tests stop exercising real behaviour; mocks drift from reality; setup dominates the test; refactoring breaks mocks (overlaps with TST-01's symptoms).
- **Applies to:** unit and integration. **Not applicable to e2e.**
- **Detection heuristics:**
  - Mocks of data/value types, collections or domain entities.
  - Mocks of third-party library types (HTTP clients, SDKs, ORM sessions) directly in tests of business code.
  - Mock setup longer than the act/assert sections; mocks returning mocks.
  - More than a few mocks for one unit under test (also a design signal for COH-01 on the production class).
- **False positives:**
  - Mocking slow or non-deterministic boundaries (network, clock, random) — appropriate.
  - Integration tests that mock one external system deliberately (e.g. a payment provider).
- **Default severity:** Medium
- **Source:** Freeman & Pryce, *Growing Object-Oriented Software, Guided by Tests* (2009) — "only mock types you own"; Khorikov (2020).

---

### TST-03 — Eager tests

- **Aliases:** testing many behaviours at once, assertion roulette
- **Definition:** A single unit test exercises and asserts several distinct behaviours, so a failure doesn't say which behaviour broke.
- **Why it hurts:** Failures are hard to diagnose; the test is a catch-all that changes whenever any of its behaviours change.
- **Applies to:** **unit tests only.** E2E and many integration tests legitimately walk a journey and assert along the way.
- **Detection heuristics:**
  - Multiple act steps (calls to the unit under test) interleaved with assertions in one test.
  - Test names joined by "and" or generic names (`testOrder`, `itWorks`).
  - Many unrelated assertions without messages.
- **False positives:**
  - Several assertions describing *one* outcome (all fields of a returned object).
  - Parameterised tests.
- **Default severity:** Low
- **Source:** Meszaros, *xUnit Test Patterns* — Eager Test, Assertion Roulette.

---

### TST-04 — Logic in tests

- **Aliases:** conditional test logic
- **Definition:** Tests contain conditionals, loops, try/catch or computations that decide what to assert, or that recompute the expected value the same way the production code does.
- **Why it hurts:** The test can be wrong in the same way as the code; some paths never execute; readers can't see the expected outcome at a glance.
- **Applies to:** **unit tests only.** E2E/integration tests may need waits, retries and loops over pages.
- **Detection heuristics:**
  - `if`/`switch`/`for`/`while` inside test bodies (not in helpers building fixtures).
  - Expected values computed with the same formula as production code.
  - `try { … } catch { fail() }` patterns instead of the framework's exception assertions.
- **False positives:**
  - Loops in clearly named fixture builders.
  - Property-based tests (generators and properties are expected).
- **Default severity:** Medium
- **Source:** Meszaros, *xUnit Test Patterns* — Conditional Test Logic.

---

### TST-05 — Shared or opaque fixtures

- **Aliases:** general fixture, mystery guest, obscure test
- **Definition:** Tests depend on large shared setup or on data defined elsewhere (shared fixture files, seeded databases, external files) so the reader can't see what matters for a given test, and tests interfere with each other.
- **Why it hurts:** Changing the fixture for one test breaks others; tests become order-dependent; understanding a failure requires hunting for hidden data.
- **Applies to:** all levels (at e2e level: tests depending on seeded data created by other tests or on environment state).
- **Detection heuristics:**
  - Large `setUp`/`beforeEach`/`beforeAll` blocks used by many tests, each using a small part.
  - Tests referencing external files or IDs (`loadFixture("big.json")`, `userId = 42`) without showing the relevant values.
  - Tests that pass alone but fail in suite (or vice versa), shared mutable state between tests.
  - E2E tests depending on data created by earlier tests.
- **False positives:**
  - Expensive shared infrastructure (a database container) shared across tests while each test owns its data.
  - Clearly named builders / object mothers that make the relevant values explicit.
- **Default severity:** Medium
- **Source:** Meszaros, *xUnit Test Patterns* — General Fixture, Mystery Guest, Obscure Test.

---

## Parked

| Name | Definition | Reason parked |
|---|---|---|
| Slow tests | Tests that take too long to run | Needs runtime information |
| Coverage gaps | Code not exercised by tests | Better caught by tooling (coverage reports) |

## See also (owned elsewhere — do not report here)

- Production code hard to test because of singletons, statics, ambient clock/IO → DEP-04
- Production classes needing many mocks (multiple responsibilities) → COH-01
